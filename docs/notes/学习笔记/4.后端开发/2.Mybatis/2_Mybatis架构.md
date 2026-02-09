---
title: 2、Mybatis架构
icon: material-icon-theme:folder-tools-open
order: 2
author: bugcode
date: 2024-11-19T00:00:00.000Z
copyright: bugcode
createTime: 2025/09/04 14:46:24
permalink: /learning-notes/backend/mybatis/Mybatis架构/
---

## 一、**MyBatis 整体架构设计**

### **核心设计理念**

1. **SQL与Java代码分离**：SQL在XML中，业务逻辑在Java中
2. **轻量级封装**：只做必要的封装，保持JDBC灵活性
3. **可配置性**：高度可配置，适应不同场景需求
4. **分层架构**：清晰的分层设计，职责分离

### **架构层次图**

```text
┌─────────────────────────────────────────┐
│           用户接口层 (API Layer)         │
│   SqlSessionFactory │ SqlSession         │
└─────────────────────────────────────────┘
                    │
┌─────────────────────────────────────────┐
│         核心处理层 (Core Layer)          │
│  Executor │ Configuration │ MappedStatement│
└─────────────────────────────────────────┘
                    │
┌─────────────────────────────────────────┐
│       基础操作层 (Base Layer)           │
│ StatementHandler │ ParameterHandler     │
│ ResultSetHandler │ TypeHandler          │
└─────────────────────────────────────────┘
                    │
┌─────────────────────────────────────────┐
│       数据源层 (DataSource Layer)       │
│     DataSource │ Transaction │ Pooled    │
└─────────────────────────────────────────┘
```

## 二、**核心模块详细设计分析**

### **1. 配置模块 (Configuration)**

#### **设计模式：单例模式 + 建造者模式**

```java
public class Configuration {
    // 核心配置容器（单例）
    protected final MapperRegistry mapperRegistry = new MapperRegistry(this);
    protected final Map<String, MappedStatement> mappedStatements = new StrictMap<>();
    protected final Map<String, Cache> caches = new StrictMap<>();
    protected final TypeHandlerRegistry typeHandlerRegistry = new TypeHandlerRegistry();
    protected final TypeAliasRegistry typeAliasRegistry = new TypeAliasRegistry();
    
    // 建造者模式：SqlSessionFactoryBuilder
    public SqlSessionFactory build(Reader reader, String environment, Properties properties) {
        XMLConfigBuilder parser = new XMLConfigBuilder(reader, environment, properties);
        return build(parser.parse());
    }
}
```

#### **配置加载流程**

```java
// 配置解析器链式设计
XMLConfigBuilder.parse()
    ├── parseConfiguration() // 解析mybatis-config.xml
    │   ├── environmentsElement() // 环境配置
    │   ├── mappersElement()      // Mapper配置
    │   ├── typeHandlersElement() // 类型处理器
    │   └── pluginsElement()      // 插件配置
    └── 构建Configuration对象
```

### **2. SQL映射模块 (MappedStatement)**

#### **设计模式：享元模式**

```java
public final class MappedStatement {
    // 核心属性（享元模式，减少重复对象）
    private String resource;
    private Configuration configuration;
    private String id;
    private SqlSource sqlSource;
    private SqlCommandType sqlCommandType;
    private ResultMap resultMap;
    private Cache cache;
    
    // 获取BoundSql（动态SQL处理后）
    public BoundSql getBoundSql(Object parameterObject) {
        return sqlSource.getBoundSql(parameterObject);
    }
}
```

#### **SQL源设计 (SqlSource)**

```java
public interface SqlSource {
    BoundSql getBoundSql(Object parameterObject);
}

// 实现类：
// - DynamicSqlSource: 动态SQL（包含if/where等标签）
// - RawSqlSource:    静态SQL（不包含${}和动态标签）
// - ProviderSqlSource: 注解方式提供SQL
// - StaticSqlSource:  最终可执行的SQL（已解析完成）
```

### **3. 执行器模块 (Executor)**

#### **设计模式：模板方法 + 装饰器模式**

```java
// 抽象基类（模板方法模式）
public abstract class BaseExecutor implements Executor {
    // 模板方法
    public <E> List<E> query(...) {
        // 1. 获取BoundSql
        BoundSql boundSql = ms.getBoundSql(parameter);
        
        // 2. 创建缓存Key
        CacheKey key = createCacheKey(ms, parameter, rowBounds, boundSql);
        
        // 3. 查询（模板方法，具体由子类实现）
        return query(ms, parameter, rowBounds, resultHandler, key, boundSql);
    }
    
    // 抽象方法，由子类实现
    protected abstract <E> List<E> doQuery(...);
}

// 装饰器（装饰器模式）
public class CachingExecutor implements Executor {
    private final Executor delegate; // 被装饰的执行器
    
    @Override
    public <E> List<E> query(...) {
        // 1. 检查二级缓存
        if (cache != null) {
            // 缓存逻辑...
        }
        
        // 2. 委托给实际执行器
        return delegate.query(ms, parameter, rowBounds, resultHandler);
    }
}
```

#### **执行器类型设计对比**

| 执行器类型          | 设计目的 | 适用场景      | 关键技术                  |
| :------------------ | :------- | :------------ | :------------------------ |
| **SimpleExecutor**  | 简单执行 | 通用场景      | 每次创建新Statement       |
| **ReuseExecutor**   | 性能优化 | SQL重复率高   | Statement缓存池           |
| **BatchExecutor**   | 批量操作 | 批量插入/更新 | addBatch()/executeBatch() |
| **CachingExecutor** | 缓存增强 | 读多写少      | 二级缓存装饰器            |

### **4. 语句处理器模块 (StatementHandler)**

#### **设计模式：策略模式**

```java
// 策略接口
public interface StatementHandler {
    Statement prepare(Connection connection, Integer transactionTimeout);
    void parameterize(Statement statement);
    <E> List<E> query(Statement statement, ResultHandler resultHandler);
}

// 具体策略实现
public class PreparedStatementHandler implements StatementHandler {
    // 处理预编译SQL
}

public class SimpleStatementHandler implements StatementHandler {
    // 处理静态SQL
}

public class CallableStatementHandler implements StatementHandler {
    // 处理存储过程
}
```

#### **参数处理器设计 (ParameterHandler)**

```java
public class DefaultParameterHandler implements ParameterHandler {
    private final TypeHandlerRegistry typeHandlerRegistry;
    
    @Override
    public void setParameters(PreparedStatement ps) {
        List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();
        
        // 参数映射链式处理
        for (int i = 0; i < parameterMappings.size(); i++) {
            ParameterMapping mapping = parameterMappings.get(i);
            
            // 获取值
            Object value = getParameterValue(mapping);
            
            // 获取TypeHandler
            TypeHandler typeHandler = mapping.getTypeHandler();
            
            // 设置参数（适配器模式）
            typeHandler.setParameter(ps, i + 1, value, mapping.getJdbcType());
        }
    }
}
```

### **5. 结果集处理器模块 (ResultSetHandler)**

#### **设计模式：组合模式 + 访问者模式**

```java
public class DefaultResultSetHandler implements ResultSetHandler {
    
    // 处理结果集（组合模式处理多层嵌套）
    public List<Object> handleResultSets(Statement stmt) throws SQLException {
        final List<Object> multipleResults = new ArrayList<>();
        
        do {
            // 处理单个ResultSet
            ResultSetWrapper rsw = getResultSetWrapper(stmt);
            
            // 获取ResultMap（可能包含嵌套映射）
            ResultMap resultMap = mappedStatement.getResultMaps().get(0);
            
            // 处理行数据
            handleRowValues(rsw, resultMap, multipleResults, null);
        } while (hasMoreResultSets(stmt));
        
        return collapseSingleResultList(multipleResults);
    }
    
    // 处理行数据（访问者模式遍历属性）
    private void handleRowValues(...) {
        while (rs.next()) {
            // 为每行创建对象
            Object rowValue = createResultObject(rsw, resultMap, null);
            
            // 应用自动映射
            applyAutomaticMappings(rsw, resultMap, rowValue, null);
            
            // 应用手动映射（访问者模式遍历ResultMapping）
            applyPropertyMappings(rsw, resultMap, rowValue, null);
        }
    }
}
```

## 三、**关键设计模式应用分析**

### **1. 工厂模式 (Factory Pattern)**

```java
// SqlSessionFactory 创建 SqlSession
public interface SqlSessionFactory {
    SqlSession openSession();
    SqlSession openSession(boolean autoCommit);
    SqlSession openSession(ExecutorType execType);
}

// 环境工厂
public class Environment {
    private final DataSource dataSource;
    private final TransactionFactory transactionFactory;
}
```

### **2. 建造者模式 (Builder Pattern)**

```java
// XMLConfigBuilder 构建 Configuration
public class XMLConfigBuilder extends BaseBuilder {
    public Configuration parse() {
        // 解析XML并构建配置对象
        parseConfiguration(parser.evalNode("/configuration"));
        return configuration;
    }
}

// SqlSessionFactoryBuilder 构建 SqlSessionFactory
public class SqlSessionFactoryBuilder {
    public SqlSessionFactory build(Configuration config) {
        return new DefaultSqlSessionFactory(config);
    }
}
```

### **3. 责任链模式 (Chain of Responsibility)**

```java
// 插件拦截器链
public class InterceptorChain {
    private final List<Interceptor> interceptors = new ArrayList<>();
    
    public Object pluginAll(Object target) {
        // 层层包装，形成责任链
        for (Interceptor interceptor : interceptors) {
            target = interceptor.plugin(target);
        }
        return target;
    }
}

// 每个插件实现拦截
@Intercepts({@Signature(...)})
public class PageInterceptor implements Interceptor {
    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        // 前置处理
        // ...
        
        // 传递到链中的下一个
        Object result = invocation.proceed();
        
        // 后置处理
        // ...
        return result;
    }
}
```

### **4. 代理模式 (Proxy Pattern)**

```java
// Mapper接口代理
public class MapperProxy<T> implements InvocationHandler {
    private final SqlSession sqlSession;
    private final Class<T> mapperInterface;
    
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // 将接口方法调用转换为SQL执行
        if (method.getDeclaringClass() == Object.class) {
            return method.invoke(this, args);
        }
        
        // 获取MappedStatement
        String statementId = mapperInterface.getName() + "." + method.getName();
        MappedStatement ms = configuration.getMappedStatement(statementId);
        
        // 执行SQL
        return sqlSession.selectOne(statementId, args[0]);
    }
}
```

## 四、**缓存架构设计**

### **缓存体系设计**

```shell
┌──────────────────────────────────┐
│        缓存接口 Cache            │
├──────────────────────────────────┤
│  +getId()                       │
│  +putObject(key, value)         │
│  +getObject(key)                │
│  +removeObject(key)             │
│  +clear()                       │
└──────────────────────────────────┘
            △
            │
┌──────────────────────────────────┐
│     装饰器缓存 DecoratorCache    │
├──────────────────────────────────┤
│  -delegate: Cache               │
│  +FifoCache                     │
│  +LruCache                      │
│  +SoftCache                     │
│  +WeakCache                     │
│  +ScheduledCache                │
│  +SerializedCache               │
│  +TransactionalCache            │
└──────────────────────────────────┘
```

### **一级缓存设计**

```java
// PerpetualCache - 永久缓存（简单HashMap实现）
public class PerpetualCache implements Cache {
    private final String id;
    private final Map<Object, Object> cache = new HashMap<>();
    
    // 线程安全访问
    private ReadWriteLock readWriteLock = new ReentrantReadWriteLock();
}
```

### **二级缓存设计**

```java
// 事务性缓存管理
public class TransactionalCacheManager {
    private final Map<Cache, TransactionalCache> transactionalCaches = new HashMap<>();
    
    public void commit() {
        for (TransactionalCache txCache : transactionalCaches.values()) {
            txCache.commit();
        }
    }
    
    public void rollback() {
        for (TransactionalCache txCache : transactionalCaches.values()) {
            txCache.rollback();
        }
    }
}

// 事务性缓存
public class TransactionalCache implements Cache {
    private final Cache delegate;
    private boolean clearOnCommit;
    private final Map<Object, Object> entriesToAddOnCommit = new HashMap<>();
    
    public void commit() {
        if (clearOnCommit) {
            delegate.clear();
        }
        flushPendingEntries();
    }
}
```

## 五、**事务管理设计**

### **事务工厂模式**

```java
public interface TransactionFactory {
    Transaction newTransaction(Connection conn);
    Transaction newTransaction(DataSource dataSource, 
                              TransactionIsolationLevel level, boolean autoCommit);
}

// 实现：
// - JdbcTransactionFactory: JDBC原生事务
// - ManagedTransactionFactory: 容器管理事务
```

### **事务接口设计**

```java
public interface Transaction {
    Connection getConnection() throws SQLException;
    void commit() throws SQLException;
    void rollback() throws SQLException;
    void close() throws SQLException;
    Integer getTimeout() throws SQLException;
}
```

## 六、**数据源与连接池设计**

### **连接池设计 (PooledDataSource)**

```java
public class PooledDataSource implements DataSource {
    // 连接池状态
    private final PoolState state = new PoolState();
    
    // 活跃连接
    private final List<PooledConnection> activeConnections = new ArrayList<>();
    
    // 空闲连接
    private final List<PooledConnection> idleConnections = new ArrayList<>();
    
    // 获取连接
    public Connection getConnection() throws SQLException {
        return popConnection().getProxyConnection();
    }
    
    // 连接池管理
    private PooledConnection popConnection() throws SQLException {
        while (true) {
            if (!idleConnections.isEmpty()) {
                // 从空闲池获取
                PooledConnection conn = idleConnections.remove(0);
                if (conn.isValid()) {
                    return conn;
                }
            }
            
            // 创建新连接
            if (activeConnections.size() < poolMaximumActiveConnections) {
                conn = new PooledConnection(dataSource.getConnection(), this);
                activeConnections.add(conn);
                return conn;
            }
            
            // 等待连接释放
            wait(poolTimeToWait);
        }
    }
}
```

## 七、**动态SQL设计**

### **SQL节点树设计**

```java
// SQL节点接口
public interface SqlNode {
    boolean apply(DynamicContext context);
}

// 节点实现类：
// - IfSqlNode: if条件节点
// - WhereSqlNode: where子句节点
// - ForEachSqlNode: foreach循环节点
// - TextSqlNode: 文本节点
// - MixedSqlNode: 混合节点（组合模式）

// 示例：IfSqlNode实现
public class IfSqlNode implements SqlNode {
    private final ExpressionEvaluator evaluator;
    private final String test;
    private final SqlNode contents;
    
    @Override
    public boolean apply(DynamicContext context) {
        // 评估OGNL表达式
        if (evaluator.evaluateBoolean(test, context.getBindings())) {
            contents.apply(context);
            return true;
        }
        return false;
    }
}
```

### **动态SQL解析流程**

```xml
<!-- XML配置 -->
<select id="findUser">
    SELECT * FROM users
    <where>
        <if test="name != null">AND name = #{name}</if>
        <if test="age != null">AND age = #{age}</if>
    </where>
</select>
```

解析过程:

```java
// 解析过程：
1. XML解析为SqlNode树
2. 根据参数动态评估节点
3. 生成最终SQL字符串
4. 创建StaticSqlSource
```

## 八、**插件扩展架构**

### **插件拦截机制**

```java
// 拦截器接口
public interface Interceptor {
    Object intercept(Invocation invocation) throws Throwable;
    Object plugin(Object target);
    void setProperties(Properties properties);
}

// 拦截器链执行
public class Plugin implements InvocationHandler {
    private final Object target;
    private final Interceptor interceptor;
    
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // 检查是否拦截该方法
        if (interceptor.getClass().isAnnotationPresent(Intercepts.class)) {
            Signature[] sigs = interceptor.getClass()
                .getAnnotation(Intercepts.class).value();
            
            for (Signature sig : sigs) {
                if (sig.type().isAssignableFrom(target.getClass()) &&
                    sig.method().equals(method.getName())) {
                    // 执行拦截
                    return interceptor.intercept(new Invocation(target, method, args));
                }
            }
        }
        return method.invoke(target, args);
    }
}
```

## 九、**类型处理架构**

### **类型处理器注册表**

```java
public class TypeHandlerRegistry {
    // 类型映射注册表
    private final Map<JdbcType, TypeHandler<?>> jdbcTypeHandlerMap = new EnumMap<>();
    private final Map<Type, Map<JdbcType, TypeHandler<?>>> typeHandlerMap = new HashMap<>();
    
    // 注册类型处理器
    public void register(Type javaType, JdbcType jdbcType, TypeHandler<?> handler) {
        // 建立双向映射
    }
    
    // 获取类型处理器
    public TypeHandler<?> getTypeHandler(Type type, JdbcType jdbcType) {
        // 精确匹配 → 父类匹配 → 默认处理器
    }
}
```

### **自定义类型处理器**

```java
@MappedTypes(String.class)
@MappedJdbcTypes(JdbcType.VARCHAR)
public class CustomStringHandler extends BaseTypeHandler<String> {
    @Override
    public void setNonNullParameter(PreparedStatement ps, int i, 
                                   String parameter, JdbcType jdbcType) {
        // 自定义设置逻辑
    }
    
    @Override
    public String getNullableResult(ResultSet rs, String columnName) {
        // 自定义获取逻辑
    }
}
```

## 十、**架构优势与设计权衡**

### **架构优势**

1. **灵活性高**：SQL与代码分离，便于优化SQL
2. **扩展性强**：插件机制支持功能扩展
3. **性能可控**：缓存、批处理等优化手段丰富
4. **学习曲线平缓**：基于JDBC，易于理解

### **设计权衡**

1. **灵活性 vs 类型安全**：XML配置灵活但缺乏编译时检查
2. **轻量级 vs 功能完备**：保持核心功能，复杂功能通过插件扩展
3. **性能 vs 内存**：缓存提升性能但增加内存消耗

### **架构演进**

- **MyBatis 3.x**：引入注解、动态SQL重构
- **MyBatis-Spring**：与Spring深度集成
- **MyBatis-Plus**：增强功能，提供更多便捷操作

## 总结

MyBatis 架构设计体现了 **"简单而强大"** 的设计哲学：

1. **分层清晰**：各层职责明确，便于维护和扩展
2. **模式运用恰当**：模板方法、装饰器、策略等模式运用合理
3. **扩展点丰富**：插件、类型处理器、缓存等均可扩展
4. **配置驱动**：XML配置提供了高度的灵活性
5. **性能优化**：多级缓存、批处理、Statement重用等优化机制

这种设计使得 MyBatis 在保持轻量级的同时，能够满足企业级应用的需求，成为 Java 持久层框架中的重要选择。