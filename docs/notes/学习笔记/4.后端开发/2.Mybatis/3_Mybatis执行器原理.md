---
title: 3、Mybatis执行器原理
icon: material-icon-theme:folder-tools-open
order: 3
author: bugcode
date: 2024-11-19T00:00:00.000Z
copyright: bugcode
createTime: 2025/09/04 14:46:24
permalink: /learning-notes/backend/mybatis/Mybatis执行器原理/
---

MyBatis 执行器（Executor）是 MyBatis 的核心组件之一，负责 SQL 语句的执行和缓存管理。以下是执行器的详细原理：

## 1. **执行器层次结构**

```java
SqlSession
    ↓
Executor（接口）
    ├── BaseExecutor（抽象类，实现一级缓存和基本操作）
    │   ├── SimpleExecutor（默认，每次执行创建新Statement）
    │   ├── ReuseExecutor（重用Statement）
    │   └── BatchExecutor（批处理）
    └── CachingExecutor（装饰器，实现二级缓存）
```

## 2. **执行器类型**

### **SimpleExecutor（默认）**

```java
// 每次执行都创建新的PreparedStatement
public class SimpleExecutor extends BaseExecutor {
    @Override
    public <E> List<E> doQuery(...) {
        Statement stmt = null;
        try {
            Configuration configuration = ms.getConfiguration();
            stmt = prepareStatement(handler, ms.getStatementLog());
            return handler.query(stmt, resultHandler);
        } finally {
            closeStatement(stmt); // 每次关闭
        }
    }
}
```

- **特点**：每次执行 SQL 都创建新的 `PreparedStatement`
- **缺点**：性能较差（频繁创建/销毁 Statement）
- **优点**：简单可靠

### **ReuseExecutor（重用执行器）**

```java
// 重用PreparedStatement
public class ReuseExecutor extends BaseExecutor {
    private final Map<String, Statement> statementMap = new HashMap<>();
    
    @Override
    public <E> List<E> doQuery(...) {
        Statement stmt = getStatement(sql); // 从缓存获取
        if (stmt == null) {
            stmt = prepareStatement(...);
            statementMap.put(sql, stmt);
        }
        return handler.query(stmt, resultHandler);
    }
}
```

- **特点**：缓存 `PreparedStatement`，相同 SQL 复用
- **优点**：减少 Statement 创建开销
- **适用**：短时间内重复执行相同 SQL

### **BatchExecutor（批处理执行器）**

```java
public class BatchExecutor extends BaseExecutor {
    private final List<Statement> statementList = new ArrayList<>();
    private final List<BatchResult> batchResultList = new ArrayList<>();
    
    @Override
    public int doUpdate(...) {
        Statement stmt = getBatchStatement(sql);
        handler.parameterize(stmt);
        handler.batch(stmt); // 添加到批处理
        return BATCH_UPDATE_RETURN_VALUE;
    }
    
    @Override
    public List<BatchResult> doFlushStatements() {
        // 一次性执行所有批处理
        for (Statement stmt : statementList) {
            stmt.executeBatch();
        }
    }
}
```

- **特点**：批量执行多个更新操作
- **优点**：大幅提升批量插入/更新性能
- **缺点**：需要手动刷新 `flushStatements()`

### **CachingExecutor（缓存执行器）**

```java
// 装饰器模式，为其他执行器添加二级缓存
public class CachingExecutor implements Executor {
    private final Executor delegate;
    private final TransactionalCacheManager tcm = new TransactionalCacheManager();
    
    @Override
    public <E> List<E> query(...) {
        // 1. 获取缓存key
        CacheKey key = createCacheKey(ms, parameter, rowBounds, boundSql);
        
        // 2. 检查二级缓存
        if (ms.isUseCache() && ms.getCache() != null) {
            cache = ms.getCache();
            if (cache != null) {
                List<E> list = (List<E>) tcm.getObject(cache, key);
                if (list != null) {
                    return list;
                }
            }
        }
        
        // 3. 委托给实际执行器查询数据库
        list = delegate.query(ms, parameter, rowBounds, resultHandler, key, boundSql);
        
        // 4. 存入二级缓存
        if (cache != null && ms.isUseCache()) {
            tcm.putObject(cache, key, list);
        }
        return list;
    }
}
```

## 3. **执行器的工作流程**

### **查询流程**

```java
// 整体执行流程
public <E> List<E> query(...) {
    // 1. 获取MappedStatement
    MappedStatement ms = configuration.getMappedStatement(statement);
    
    // 2. 创建缓存Key
    CacheKey key = createCacheKey(ms, parameter, rowBounds, boundSql);
    
    // 3. 查询（先查缓存，再查数据库）
    return query(ms, parameter, rowBounds, resultHandler, key, boundSql);
}

protected <E> List<E> queryFromDatabase(...) {
    List<E> list;
    localCache.putObject(key, EXECUTION_PLACEHOLDER);
    try {
        // 4. 调用具体执行器执行SQL
        list = doQuery(ms, parameter, rowBounds, resultHandler, boundSql);
    } finally {
        localCache.removeObject(key);
    }
    // 5. 结果放入一级缓存
    localCache.putObject(key, list);
    return list;
}
```

## 4. **一级缓存（本地缓存）原理**

### **BaseExecutor中的实现**

```java
public abstract class BaseExecutor implements Executor {
    // 一级缓存（PerpetualCache）
    protected PerpetualCache localCache;
    
    @Override
    public <E> List<E> query(...) {
        // 1. 检查一级缓存
        list = (E) localCache.getObject(key);
        if (list != null) {
            return list;
        }
        
        // 2. 缓存未命中，查询数据库
        list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);
        
        return list;
    }
    
    // 缓存清除条件
    @Override
    public int update(...) {
        // 执行更新操作后，清除一级缓存
        clearLocalCache();
        return doUpdate(ms, parameter);
    }
}
```

**一级缓存特点**：

- **作用域**：SqlSession 级别
- **存储位置**：`PerpetualCache`（简单的 HashMap）
- **清除时机**：
    1. 执行更新操作（insert/update/delete）
    2. 手动调用 `clearLocalCache()`
    3. 提交/回滚事务
    4. 关闭 SqlSession

## 5. **二级缓存原理**

### **工作流程**

```shell
查询请求
    ↓
CachingExecutor.query()
    ↓
检查二级缓存 → 命中 → 返回结果
    ↓
未命中
    ↓
委托实际执行器查询数据库
    ↓
结果存入二级缓存
    ↓
返回结果
```

### **配置示例**

```xml
<!-- 1. 全局开启二级缓存 -->
<settings>
    <setting name="cacheEnabled" value="true"/>
</settings>

<!-- 2. Mapper中配置缓存 -->
<cache
  eviction="LRU"
  flushInterval="60000"
  size="512"
  readOnly="true"/>
```

## 6. **执行器选择与配置**

### **配置方式**

```java
// 通过SqlSessionFactory配置
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder()
    .build(configuration);

// 创建SqlSession时指定执行器类型
SqlSession session = sqlSessionFactory.openSession(ExecutorType.BATCH);
SqlSession session = sqlSessionFactory.openSession(ExecutorType.REUSE);
```

### **执行器类型对比**

| 执行器类型          | 一级缓存 | Statement处理 | 适用场景          |
| :------------------ | :------- | :------------ | :---------------- |
| **SimpleExecutor**  | 支持     | 每次新建      | 常规操作          |
| **ReuseExecutor**   | 支持     | 缓存重用      | SQL重复率高的场景 |
| **BatchExecutor**   | 不支持   | 批处理        | 批量插入/更新     |
| **CachingExecutor** | 装饰器   | 依赖被装饰者  | 需要二级缓存      |

## 7. **执行器与插件（Interceptor）**

MyBatis 插件通过拦截执行器方法实现功能扩展：

```java
@Intercepts({
    @Signature(type = Executor.class,
               method = "query",
               args = {MappedStatement.class, Object.class, 
                       RowBounds.class, ResultHandler.class})
})
public class MyPlugin implements Interceptor {
    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        // 执行前处理
        System.out.println("Before query...");
        
        // 执行原方法
        Object result = invocation.proceed();
        
        // 执行后处理
        System.out.println("After query...");
        return result;
    }
}
```

## **总结**

1. **执行器是 MyBatis 的 SQL 执行引擎**，负责具体的数据操作
2. **三级结构**：接口 → 抽象基类 → 具体实现
3. **核心功能**：
    - SQL 语句执行
    - 一级缓存管理
    - 事务管理（BaseExecutor）
4. **设计模式**：
    - 模板方法模式（BaseExecutor）
    - 装饰器模式（CachingExecutor）
5. **性能优化**：通过不同执行器类型适应不同场景
6. **扩展性**：通过插件机制可以拦截执行器方法

理解执行器原理对于优化 MyBatis 性能、实现自定义扩展非常重要。