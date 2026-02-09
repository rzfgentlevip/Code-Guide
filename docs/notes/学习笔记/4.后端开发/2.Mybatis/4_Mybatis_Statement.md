---
title: 4、Mybatis Statement 对象
icon: material-icon-theme:folder-tools-open
order: 4
author: bugcode
date: 2024-11-19T00:00:00.000Z
copyright: bugcode
createTime: 2025/09/04 14:46:24
permalink: /learning-notes/backend/mybatis/Mybatis_Statement/
---

在 MyBatis 中，**Statement 对象是 JDBC Statement 的封装**，是 MyBatis 执行 SQL 语句的核心载体。下面详细讲解 Statement 对象在 MyBatis 中的角色和原理：

## 1. **Statement 对象层次结构**

```java
java.sql.Statement（JDBC接口）
    ├── java.sql.PreparedStatement（预编译）
    ├── java.sql.CallableStatement（存储过程）
    └── org.apache.ibatis.executor.statement.StatementHandler（MyBatis封装）
        ├── SimpleStatementHandler（对应Statement）
        ├── PreparedStatementHandler（对应PreparedStatement）
        └── CallableStatementHandler（对应CallableStatement）
```

## 2. **MyBatis 中的 StataementHandler**

MyBatis 通过 `StatementHandler` 封装了对 JDBC Statement 的操作：

### **接口定义**

```java
public interface StatementHandler {
    // 准备Statement
    Statement prepare(Connection connection, Integer transactionTimeout);
    
    // 参数化处理（设置参数）
    void parameterize(Statement statement);
    
    // 批量执行
    void batch(Statement statement);
    
    // 执行查询
    <E> List<E> query(Statement statement, ResultHandler resultHandler);
    
    // 执行更新
    int update(Statement statement);
    
    // 获取BoundSql（包含SQL和参数）
    BoundSql getBoundSql();
}
```

## 3. **三种 StatementHandler 实现**

### **SimpleStatementHandler**

```java
// 对应 JDBC 的 Statement（用于静态SQL）
public class SimpleStatementHandler extends BaseStatementHandler {
    @Override
    protected Statement instantiateStatement(Connection connection) throws SQLException {
        // 直接创建普通Statement
        return connection.createStatement();
    }
    
    @Override
    public void parameterize(Statement statement) {
        // 普通Statement不支持参数化，无需处理
    }
}
```

- **适用**：简单 SQL，不包含参数占位符
- **SQL注入风险**：高（直接拼接）
- **性能**：每次都要编译 SQL

### **PreparedStatementHandler（最常用）**

```java
// 对应 JDBC 的 PreparedStatement（用于预编译SQL）
public class PreparedStatementHandler extends BaseStatementHandler {
    @Override
    protected Statement instantiateStatement(Connection connection) throws SQLException {
        String sql = boundSql.getSql();
        // 创建预编译Statement
        return connection.prepareStatement(sql);
    }
    
    @Override
    public void parameterize(Statement statement) throws SQLException {
        // 使用ParameterHandler设置参数
        parameterHandler.setParameters((PreparedStatement) statement);
    }
}
```

- **适用**：带 `#{}` 占位符的 SQL
- **SQL注入风险**：低（预编译）
- **性能**：可预编译，支持批量重用

### **CallableStatementHandler**

```java
// 对应 JDBC 的 CallableStatement（用于存储过程）
public class CallableStatementHandler extends BaseStatementHandler {
    @Override
    protected Statement instantiateStatement(Connection connection) throws SQLException {
        String sql = boundSql.getSql();
        // 创建CallableStatement
        return connection.prepareCall(sql);
    }
    
    @Override
    public void parameterize(Statement statement) throws SQLException {
        // 设置存储过程参数（包括输入/输出参数）
        parameterHandler.setParameters((CallableStatement) statement);
    }
}
```

- **适用**：存储过程调用
- **特点**：支持输入/输出参数

## 4. **Statement 对象的创建流程**

```java
// 在SimpleExecutor中的创建过程
public class SimpleExecutor extends BaseExecutor {
    private Statement prepareStatement(StatementHandler handler, Log statementLog) throws SQLException {
        Statement stmt;
        Connection connection = getConnection(statementLog);
        
        // 1. 创建Statement
        stmt = handler.prepare(connection, transaction.getTimeout());
        
        // 2. 参数化（为?占位符设置值）
        handler.parameterize(stmt);
        
        return stmt;
    }
}

// StatementHandler的prepare方法
public Statement prepare(Connection connection, Integer transactionTimeout) throws SQLException {
    // 实例化具体Statement
    Statement statement = instantiateStatement(connection);
    
    // 设置超时、获取大小等参数
    setStatementTimeout(statement, transactionTimeout);
    setFetchSize(statement);
    
    return statement;
}
```

## 5. **Statement 与 SQL 执行**

### **查询执行流程**

```java
// Executor执行查询
public <E> List<E> doQuery(...) {
    Statement stmt = null;
    try {
        // 1. 准备Statement
        stmt = prepareStatement(handler, ms.getStatementLog());
        
        // 2. 执行查询，返回结果
        return handler.query(stmt, resultHandler);
    } finally {
        // 3. 关闭Statement（SimpleExecutor）
        closeStatement(stmt);
    }
}

// StatementHandler的query方法
public <E> List<E> query(Statement statement, ResultHandler resultHandler) throws SQLException {
    // 执行SQL
    PreparedStatement ps = (PreparedStatement) statement;
    ps.execute();
    
    // 处理结果集
    return resultSetHandler.handleResultSets(ps);
}
```

### **更新执行流程**

```java
public int doUpdate(...) {
    Statement stmt = null;
    try {
        stmt = prepareStatement(handler, ms.getStatementLog());
        
        // 执行更新
        return handler.update(stmt);
    } finally {
        closeStatement(stmt);
    }
}

// 在PreparedStatementHandler中
public int update(Statement statement) throws SQLException {
    PreparedStatement ps = (PreparedStatement) statement;
    ps.execute();
    
    // 返回影响行数
    int rows = ps.getUpdateCount();
    return rows;
}
```

## 6. **Statement 与参数处理**

### **ParameterHandler 的作用**

```java
public class DefaultParameterHandler implements ParameterHandler {
    @Override
    public void setParameters(PreparedStatement ps) throws SQLException {
        List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();
        
        for (int i = 0; i < parameterMappings.size(); i++) {
            ParameterMapping parameterMapping = parameterMappings.get(i);
            String propertyName = parameterMapping.getProperty();
            Object value;
            
            // 获取参数值
            if (boundSql.hasAdditionalParameter(propertyName)) {
                value = boundSql.getAdditionalParameter(propertyName);
            } else {
                value = metaObject.getValue(propertyName);
            }
            
            // 获取TypeHandler
            TypeHandler typeHandler = parameterMapping.getTypeHandler();
            JdbcType jdbcType = parameterMapping.getJdbcType();
            
            // 设置参数（关键！）
            typeHandler.setParameter(ps, i + 1, value, jdbcType);
        }
    }
}
```

## 7. **Statement 与不同执行器的关系**

### **在 SimpleExecutor 中**

```java
// 每次执行都创建新的Statement
public <E> List<E> doQuery(...) {
    Statement stmt = prepareStatement(handler, ms.getStatementLog());
    try {
        return handler.query(stmt, resultHandler);
    } finally {
        closeStatement(stmt); // 立即关闭
    }
}
```

**在 SimpleExecutor 中**：每次执行 SQL 都创建新的 Statement 对象,执行完立即关闭;

### **在 ReuseExecutor 中**

```java
// 重用Statement
public class ReuseExecutor extends BaseExecutor {
    private final Map<String, Statement> statementMap = new HashMap<>();
    
    private Statement prepareStatement(...) {
        String sql = boundSql.getSql();
        Statement stmt = statementMap.get(sql);
        
        if (stmt == null) {
            stmt = super.prepareStatement(handler, statementLog);
            statementMap.put(sql, stmt);
        }
        
        // 重用前需要重新设置参数
        handler.parameterize(stmt);
        return stmt;
    }
}
```

**在 ReuseExecutor 中**：**不是**每次执行都创建新的 Statement，相同 SQL 会重用

### **在 BatchExecutor 中**

```java
// 批处理Statement
public class BatchExecutor extends BaseExecutor {
    private Statement currentStatement;
    
    @Override
    public int doUpdate(...) {
        if (currentStatement == null || !sql.equals(currentSql)) {
            // 创建新的批处理Statement
            currentStatement = prepareStatement(handler, ms.getStatementLog());
            currentSql = sql;
            statementList.add(currentStatement);
        }
        
        // 添加到批处理
        handler.parameterize(currentStatement);
        handler.batch(currentStatement);
        return BATCH_UPDATE_RETURN_VALUE;
    }
}
```

**在 BatchExecutor 中**：相同 SQL 在批处理中重用 Statement

## 8. **Statement 生命周期管理**

```java
// 以 SimpleExecutor 为例
1. 创建：connection.prepareStatement(sql)
2. 参数化：parameterHandler.setParameters(statement)
3. 执行：statement.executeQuery() / executeUpdate()
4. 结果处理：resultSetHandler.handleResultSets()
5. 关闭：statement.close()

// 关键时间点
Statement stmt = null;
try {
    // 创建（此时才真正生成Statement对象）
    stmt = connection.prepareStatement(sql);
    
    // 参数化（设置占位符的值）
    parameterHandler.setParameters(stmt);
    
    // 执行SQL
    ResultSet rs = stmt.executeQuery();
    
    // 处理结果
    return resultSetHandler.handleResultSets(stmt);
    
} finally {
    // 关闭（释放数据库资源）
    if (stmt != null) {
        stmt.close();
    }
}
```

## 9. **Statement 的配置参数**

### **通过 MyBatis 配置**

```xml
<settings>
    <!-- Statement超时时间（秒） -->
    <setting name="defaultStatementTimeout" value="25"/>
    
    <!-- 获取大小 -->
    <setting name="defaultFetchSize" value="100"/>
    
    <!-- 结果集类型 -->
    <setting name="defaultResultSetType" value="FORWARD_ONLY"/>
</settings>
```

### **通过 Mapper 配置**

```xml
<select id="selectUser" 
        statementType="PREPARED"  <!-- 可选项：STATEMENT, PREPARED, CALLABLE -->
        timeout="10"
        fetchSize="100"
        resultSetType="FORWARD_ONLY">
    SELECT * FROM users WHERE id = #{id}
</select>
```

## 10. **Statement 与插件扩展**

MyBatis 插件可以拦截 StatementHandler 的方法：

```java
@Intercepts({
    @Signature(type = StatementHandler.class,
               method = "prepare",
               args = {Connection.class, Integer.class})
})
public class StatementInterceptor implements Interceptor {
    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        // 可以修改Statement创建过程
        StatementHandler handler = (StatementHandler) invocation.getTarget();
        
        // 添加自定义逻辑
        System.out.println("创建Statement前...");
        
        // 执行原方法
        Statement stmt = (Statement) invocation.proceed();
        
        // 可以修改Statement
        System.out.println("创建Statement后...");
        return stmt;
    }
}
```

## **总结**

1. **Statement 是 JDBC 的核心对象**，MyBatis 通过 StatementHandler 对其进行封装
2. **三种类型**：
    - `Statement`：静态 SQL（有注入风险）
    - `PreparedStatement`：预编译 SQL（推荐使用）
    - `CallableStatement`：存储过程调用
3. **主要职责**：
    - SQL 语句执行
    - 参数设置（通过 ParameterHandler）
    - 结果集处理（通过 ResultSetHandler）
4. **生命周期管理**：
    - 创建：由 Executor 负责
    - 参数化：由 StatementHandler 处理
    - 关闭：不同 Executor 策略不同
5. **性能影响**：
    - Statement 创建和销毁有开销
    - PreparedStatement 可预编译和重用
    - 批处理可大幅提升性能
6. **Statement 中的关键信息**
    - 已解析的SQL（#{xxx}已替换为?）
    - Connection信息
    - 事务上下文
    - 执行参数（超时、获取大小等）

理解 Statement 对象对于优化 MyBatis 性能、调试 SQL 执行过程非常重要。

| 执行器类型         | Statement 创建时机             | 是否每次SQL都创建    |
| :----------------- | :----------------------------- | :------------------- |
| **SimpleExecutor** | 每次执行 doQuery/doUpdate 时   | ✅ 是                 |
| **ReuseExecutor**  | 首次执行某SQL时创建，后续重用  | ❌ 否（相同SQL重用）  |
| **BatchExecutor**  | 批处理开始时创建，批处理中重用 | ❌ 否（批处理内重用） |