---
title: 6、Mybatis SQL执行全流程解析
icon: material-icon-theme:folder-tools-open
order: 6
author: bugcode
date: 2024-11-19T00:00:00.000Z
copyright: bugcode
createTime: 2025/09/04 14:46:24
permalink: /learning-notes/backend/mybatis/Mybatis_SQL执行全流程解析/
---

# MyBatis 架构深度分析：SQL执行全流程解析

## 一、**MyBatis 整体架构图**

```shell
用户代码层
    ↓
SqlSession API层
    ↓
执行器层 (Executor)
    ↓
Statement处理器层 (StatementHandler)
    ↓
参数处理器层 (ParameterHandler)
    ↓
结果处理器层 (ResultSetHandler)
    ↓
JDBC驱动层
    ↓
数据库
```

## 二、**核心组件职责分析**

| 组件                 | 职责                          | 关键接口/类                       |
| :------------------- | :---------------------------- | :-------------------------------- |
| **SqlSession**       | 用户操作入口，提供CRUD API    | `DefaultSqlSession`               |
| **Executor**         | SQL执行调度器，管理缓存和事务 | `BaseExecutor`, `CachingExecutor` |
| **StatementHandler** | Statement创建和执行           | `PreparedStatementHandler`        |
| **ParameterHandler** | SQL参数设置                   | `DefaultParameterHandler`         |
| **ResultSetHandler** | 结果集映射处理                | `DefaultResultSetHandler`         |
| **TypeHandler**      | 类型转换                      | 各种 `TypeHandler` 实现           |
| **MappedStatement**  | SQL映射信息封装               | `MappedStatement`                 |
| **Configuration**    | 全局配置信息                  | `Configuration`                   |

## 三、**SQL执行全流程详细分析**

### **阶段1：用户发起请求**

```java
// 用户代码示例
SqlSession sqlSession = sqlSessionFactory.openSession();
try {
    User user = sqlSession.selectOne("com.example.UserMapper.selectUser", 1);
} finally {
    sqlSession.close();
}
```

### **阶段2：SqlSession处理**

```java
// DefaultSqlSession.selectOne()
public <T> T selectOne(String statement, Object parameter) {
    // 1. 委托给selectList
    List<T> list = selectList(statement, parameter);
    
    // 2. 验证结果数量
    if (list.size() == 1) {
        return list.get(0);
    } else if (list.size() > 1) {
        throw new TooManyResultsException();
    } else {
        return null;
    }
}

// selectList方法
public <E> List<E> selectList(String statement, Object parameter) {
    // 3. 获取MappedStatement
    MappedStatement ms = configuration.getMappedStatement(statement);
    
    // 4. 委托给Executor执行
    return executor.query(ms, wrapCollection(parameter), 
                         RowBounds.DEFAULT, Executor.NO_RESULT_HANDLER);
}
```

### **阶段3：Executor执行（核心流程）**

```java
// CachingExecutor.query() - 装饰器模式
public <E> List<E> query(MappedStatement ms, Object parameter, 
                         RowBounds rowBounds, ResultHandler resultHandler) {
    // 1. 获取BoundSql（包含最终SQL和参数映射）
    BoundSql boundSql = ms.getBoundSql(parameter);
    
    // 2. 创建缓存Key
    CacheKey key = createCacheKey(ms, parameter, rowBounds, boundSql);
    
    // 3. 执行查询
    return query(ms, parameter, rowBounds, resultHandler, key, boundSql);
}

// BaseExecutor.query() - 模板方法模式
public <E> List<E> query(MappedStatement ms, Object parameter, 
                         RowBounds rowBounds, ResultHandler resultHandler,
                         CacheKey key, BoundSql boundSql) {
    // 4. 检查一级缓存
    List<E> list = (E) localCache.getObject(key);
    if (list != null) {
        return list;
    }
    
    // 5. 缓存未命中，查询数据库
    list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);
    
    return list;
}

// 查询数据库
private <E> List<E> queryFromDatabase(...) {
    List<E> list;
    try {
        // 6. 占位，防止并发重复查询
        localCache.putObject(key, EXECUTION_PLACEHOLDER);
        
        // 7. 执行doQuery（由子类实现）
        list = doQuery(ms, parameter, rowBounds, resultHandler, boundSql);
    } finally {
        localCache.removeObject(key);
    }
    
    // 8. 放入一级缓存
    localCache.putObject(key, list);
    return list;
}

// SimpleExecutor.doQuery()
public <E> List<E> doQuery(...) {
    Statement stmt = null;
    try {
        Configuration configuration = ms.getConfiguration();
        
        // 9. 创建StatementHandler
        StatementHandler handler = configuration.newStatementHandler(
            wrapper, ms, parameter, rowBounds, resultHandler, boundSql);
        
        // 10. 准备Statement
        stmt = prepareStatement(handler, ms.getStatementLog());
        
        // 11. 执行查询
        return handler.query(stmt, resultHandler);
    } finally {
        closeStatement(stmt);
    }
}
```

### **阶段4：StatementHandler处理**

```java
// BaseStatementHandler.prepare()
public Statement prepare(Connection connection, Integer transactionTimeout) {
    // 1. 实例化Statement
    Statement statement = instantiateStatement(connection);
    
    // 2. 设置参数
    setStatementTimeout(statement, transactionTimeout);
    setFetchSize(statement);
    
    return statement;
}

// PreparedStatementHandler.instantiateStatement()
protected Statement instantiateStatement(Connection connection) throws SQLException {
    // 3. 获取SQL（已解析好的）
    String sql = boundSql.getSql();
    
    // 4. 创建PreparedStatement
    return connection.prepareStatement(sql);
}

// parameterize() - 参数化处理
public void parameterize(Statement statement) throws SQLException {
    // 5. 委托给ParameterHandler设置参数
    parameterHandler.setParameters((PreparedStatement) statement);
}
```

### **阶段5：ParameterHandler参数设置**

```java
// DefaultParameterHandler.setParameters()
public void setParameters(PreparedStatement ps) throws SQLException {
    // 1. 获取参数映射列表
    List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();
    
    if (parameterMappings != null) {
        for (int i = 0; i < parameterMappings.size(); i++) {
            ParameterMapping parameterMapping = parameterMappings.get(i);
            
            // 2. 获取参数值
            Object value = getParameterValue(parameterMapping, parameterObject);
            
            // 3. 获取TypeHandler
            TypeHandler typeHandler = parameterMapping.getTypeHandler();
            JdbcType jdbcType = parameterMapping.getJdbcType();
            
            // 4. 设置参数（核心！）
            typeHandler.setParameter(ps, i + 1, value, jdbcType);
        }
    }
}
```

### **阶段6：执行SQL并处理结果**

```java
// PreparedStatementHandler.query()
public <E> List<E> query(Statement statement, ResultHandler resultHandler) throws SQLException {
    // 1. 执行SQL
    PreparedStatement ps = (PreparedStatement) statement;
    ps.execute();
    
    // 2. 处理结果集
    return resultSetHandler.handleResultSets(ps);
}

// DefaultResultSetHandler.handleResultSets()
public List<Object> handleResultSets(Statement stmt) throws SQLException {
    final List<Object> multipleResults = new ArrayList<>();
    
    ResultSetWrapper rsw = getFirstResultSet(stmt);
    
    // 3. 获取ResultMap
    List<ResultMap> resultMaps = mappedStatement.getResultMaps();
    
    while (rsw != null && resultMaps.size() > 0) {
        ResultMap resultMap = resultMaps.get(0);
        
        // 4. 处理单行结果
        handleResultSet(rsw, resultMap, multipleResults, null);
        
        rsw = getNextResultSet(stmt);
    }
    
    return collapseSingleResultList(multipleResults);
}

// 处理结果集
private void handleResultSet(...) throws SQLException {
    if (resultHandler == null) {
        // 5. 创建默认结果处理器
        DefaultResultHandler defaultResultHandler = new DefaultResultHandler();
        handleRowValues(rsw, resultMap, defaultResultHandler, rowBounds, null);
        multipleResults.add(defaultResultHandler.getResultList());
    }
}

// 处理行数据
public void handleRowValues(...) throws SQLException {
    while (shouldProcessMoreRows(resultContext, rowBounds) && rsw.getResultSet().next()) {
        // 6. 解析单行数据
        Object rowValue = getRowValue(rsw, discriminatedResultMap);
        
        // 7. 存储结果
        storeObject(resultHandler, resultContext, rowValue, parentMapping);
    }
}

// 获取行值
private Object getRowValue(ResultSetWrapper rsw, ResultMap resultMap) throws SQLException {
    // 8. 创建结果对象
    Object rowValue = createResultObject(rsw, resultMap, null);
    
    if (rowValue != null && !hasTypeHandlerForResultObject(rsw, resultMap.getType())) {
        // 9. 应用自动映射
        applyAutomaticMappings(rsw, resultMap, rowValue, null);
        
        // 10. 应用属性映射
        applyPropertyMappings(rsw, resultMap, rowValue, null);
    }
    
    return rowValue;
}
```

## 四、**组件协调时序图**

```shell
用户代码
    │
    ↓ 1.selectOne()
SqlSession
    │
    ↓ 2.getMappedStatement()
Configuration
    │
    ↓ 3.query()
Executor
    │   ├── 4.检查一级缓存
    │   ├── 5.检查二级缓存(CachingExecutor)
    │   └── 6.缓存未命中
    │
    ↓ 7.doQuery()
StatementHandler
    │   ├── 8.prepare()创建Statement
    │   ├── 9.parameterize()设置参数
    │   └── 10.query()执行查询
    │
    ↓ 11.setParameters()
ParameterHandler
    │   ├── 12.获取参数值
    │   └── 13.使用TypeHandler设置参数
    │
    ↓ 14.execute()
JDBC PreparedStatement
    │
    ↓ 15.handleResultSets()
ResultSetHandler
    │   ├── 16.遍历ResultSet
    │   ├── 17.创建结果对象
    │   ├── 18.自动映射字段
    │   └── 19.属性映射
    │
    ↓ 20.返回结果
用户代码
```

## 五、**数据流转分析**

### **1. SQL语句流转**

```shell
Mapper XML中的SQL
    ↓ 解析时
MappedStatement中保存
    ↓ 执行时
BoundSql（已替换#{}为?）
    ↓ 
StatementHandler.prepare()
    ↓
JDBC PreparedStatement
```

### **2. 参数流转**

```shell
用户传入参数Object
    ↓
SqlSession.wrapCollection()
    ↓
ParameterObject
    ↓
ParameterHandler.getParameterValue()
    ↓
TypeHandler.setParameter()
    ↓
PreparedStatement.setXxx()
```

### **3. 结果流转**

```shell
JDBC ResultSet
    ↓
ResultSetWrapper封装
    ↓
ResultSetHandler.handleResultSets()
    ↓
创建Java对象
    ↓
自动映射 + 手动映射
    ↓
返回List<Object>
```

## 六、**缓存协调机制**

### **一级缓存（Local Cache）**

```java
// 工作流程
1. 查询前：创建CacheKey(SQL + 参数 + 分页等)
2. 查询时：localCache.getObject(key)
3. 缓存命中：直接返回
4. 缓存未命中：查询数据库 → localCache.putObject(key, result)
5. 更新操作：clearLocalCache() // 清空缓存

// 作用域：SqlSession级别
// 存储：PerpetualCache（简单的HashMap）
```

### **二级缓存（全局缓存）**

```java
// CachingExecutor中的处理
1. 检查是否启用缓存：ms.isUseCache()
2. 获取缓存对象：ms.getCache()
3. 从缓存获取：tcm.getObject(cache, key)
4. 未命中：委托实际Executor查询
5. 查询后：tcm.putObject(cache, key, result)

// 事务性缓存管理
TransactionalCacheManager tcm:
  - 事务提交：将数据刷入真实缓存
  - 事务回滚：清除临时缓存
```

## 七、**事务管理协调**

### **事务与执行器关系**

```java
// 执行器与事务绑定
public Executor newExecutor(Transaction transaction, ExecutorType executorType) {
    Executor executor;
    
    // 创建基础执行器
    if (ExecutorType.BATCH == executorType) {
        executor = new BatchExecutor(this, transaction);
    } else if (ExecutorType.REUSE == executorType) {
        executor = new ReuseExecutor(this, transaction);
    } else {
        executor = new SimpleExecutor(this, transaction);
    }
    
    // 包装缓存执行器
    if (cacheEnabled) {
        executor = new CachingExecutor(executor);
    }
    
    // 插件拦截（责任链模式）
    executor = (Executor) interceptorChain.pluginAll(executor);
    
    return executor;
}
```

## 八、**插件机制协调**

### **插件拦截链**

```java
// InterceptorChain.pluginAll()
public Object pluginAll(Object target) {
    for (Interceptor interceptor : interceptors) {
        target = interceptor.plugin(target); // 层层包装
    }
    return target;
}

// 典型插件：PageHelper
// 拦截Executor.query()方法，在查询前后添加分页逻辑

// 插件执行顺序（责任链模式）：
// 调用 → 插件1.before → 插件2.before → 实际方法 → 插件2.after → 插件1.after
```

## 九、**异常处理流程**

```java
// 异常传播路径
JDBC SQLException
    ↓
StatementHandler/ResultSetHandler捕获
    ↓
包装为PersistenceException
    ↓
SqlSession向上抛出
    ↓
用户代码捕获处理

// 关键点：事务回滚
try {
    // 执行SQL
    executor.update(...);
    // 提交事务
    sqlSession.commit();
} catch (Exception e) {
    // 回滚事务
    sqlSession.rollback();
    throw e;
}
```

## 十、**性能优化点分析**

### **1. Statement重用**

```java
// ReuseExecutor优化
private Map<String, Statement> statementMap = new HashMap<>();

// 相同SQL重用Statement，减少创建开销
```

### **2. 批量处理**

```java
// BatchExecutor优化
List<Statement> statementList;
// 累积多个操作，一次性执行
```

### **3. 延迟加载**

```java
// 关联查询优化
<association property="dept" select="selectDept" column="dept_id" fetchType="lazy"/>

// 使用时才触发查询，避免N+1问题
```

### **4. 缓存策略**

```java
// 合理配置缓存
<cache eviction="LRU" size="1024" readOnly="true"/>

// 根据业务特点选择：
// - 读多写少：适合缓存
// - 实时性要求高：不适合缓存
```

## 总结：MyBatis 架构特点

1. **分层清晰**：每层职责单一，便于理解和扩展
2. **模板方法模式**：BaseExecutor 定义流程，子类实现细节
3. **装饰器模式**：CachingExecutor 增强缓存功能
4. **插件机制**：通过拦截器实现功能扩展
5. **配置化**：XML配置与代码分离，灵活可维护
6. **类型安全**：TypeHandler 保障类型转换安全

整个流程体现了 **"配置优于编码"** 的设计哲学，通过灵活的配置和清晰的架构，实现了SQL与Java代码的优雅分离。