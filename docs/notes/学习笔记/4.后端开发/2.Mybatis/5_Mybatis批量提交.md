---
title: 5、Mybatis批量提交
icon: material-icon-theme:folder-tools-open
order: 5
author: bugcode
date: 2024-11-19T00:00:00.000Z
copyright: bugcode
createTime: 2025/09/04 14:46:24
permalink: /learning-notes/backend/mybatis/Mybatis批量提交/
---

# MyBatis BatchExecutor 批量提交原理深度解析

## 一、**BatchExecutor 整体设计思想**

### **核心设计理念**

1. **延迟执行**：不是立即执行每个SQL，而是累积后批量执行
2. **Statement重用**：相同SQL重用同一个PreparedStatement
3. **批量优化**：利用JDBC的 `addBatch()` 和 `executeBatch()`
4. **资源管理**：需要手动控制提交时机

## 二、**BatchExecutor 核心实现原理**

### **1. 类结构设计**

```java
public class BatchExecutor extends BaseExecutor {
    // 当前正在使用的Statement
    private Statement currentStatement;
    
    // 当前SQL（用于判断是否相同SQL）
    private String currentSql;
    
    // 存储所有Statement的列表
    private final List<Statement> statementList = new ArrayList<>();
    
    // 存储批处理结果的列表
    private final List<BatchResult> batchResultList = new ArrayList<>();
    
    // 批处理中每个Statement对应的参数列表
    private final Map<String, List<Object>> parameterMap = new HashMap<>();
}
```



### **2. 关键属性说明**

| 属性               | 类型                        | 作用                    |
| :----------------- |:--------------------------| :---------------------- |
| `currentStatement` | Statement                 | 当前正在累积的Statement |
| `currentSql`       | String                    | 当前Statement对应的SQL  |
| `statementList`    | `List<Statement>`           | 所有批处理中的Statement |
| `batchResultList`  | `List<BatchResult>`         | 批量执行结果            |
| `parameterMap`     | `Map<String, List<Object>>` | 存储每个SQL的参数列表   |

## 三、**批量执行流程详解**

### **1. 批量添加操作（doUpdate/doQuery）**

java

```
@Override
public int doUpdate(MappedStatement ms, Object parameterObject) throws SQLException {
    // 1. 获取配置
    final Configuration configuration = ms.getConfiguration();
    
    // 2. 创建StatementHandler
    final StatementHandler handler = configuration.newStatementHandler(
        this, ms, parameterObject, RowBounds.DEFAULT, null, null);
    
    // 3. 获取BoundSql（包含最终SQL）
    final BoundSql boundSql = handler.getBoundSql();
    final String sql = boundSql.getSql();
    
    // 4. 判断是否需要创建新的Statement
    if (currentStatement == null || !sql.equals(currentSql)) {
        // 关闭当前的（如果有）
        if (currentStatement != null) {
            currentStatement.close();
        }
        
        // 创建新的Statement
        currentStatement = prepareStatement(handler, ms.getStatementLog());
        currentSql = sql;
        
        // 添加到statementList
        statementList.add(currentStatement);
    }
    
    // 5. 设置参数到当前Statement
    handler.parameterize(currentStatement);
    
    // 6. 添加到批处理（关键！）
    handler.batch(currentStatement);
    
    // 7. 记录参数（用于错误时重试）
    List<Object> parameters = parameterMap.get(sql);
    if (parameters == null) {
        parameters = new ArrayList<>();
        parameterMap.put(sql, parameters);
    }
    parameters.add(parameterObject);
    
    // 8. 返回特殊标记（不是实际影响行数）
    return BATCH_UPDATE_RETURN_VALUE;
}
```



### **2. 批量执行（doFlushStatements）**

java

```
@Override
public List<BatchResult> doFlushStatements(boolean isRollback) throws SQLException {
    try {
        // 结果列表
        List<BatchResult> results = new ArrayList<>();
        
        // 如果回滚，直接清理不执行
        if (isRollback) {
            return Collections.emptyList();
        }
        
        // 遍历所有Statement并执行批处理
        for (int i = 0; i < statementList.size(); i++) {
            Statement stmt = statementList.get(i);
            
            try {
                // 获取该Statement对应的SQL
                String sql = getSqlForStatement(i);
                
                // 执行批处理（关键！）
                int[] updateCounts = stmt.executeBatch();
                
                // 创建BatchResult
                BatchResult batchResult = new BatchResult(ms, sql);
                batchResult.setParameterObjects(parameterMap.get(sql));
                batchResult.setUpdateCounts(updateCounts);
                
                results.add(batchResult);
                
            } finally {
                // 关闭Statement
                stmt.close();
            }
        }
        
        return results;
        
    } finally {
        // 清理所有资源
        for (Statement stmt : statementList) {
            closeStatement(stmt);
        }
        currentStatement = null;
        statementList.clear();
        parameterMap.clear();
        batchResultList.clear();
    }
}
```



## 四、**StatementHandler.batch() 方法实现**

### **1. BaseStatementHandler中的实现**

java

```
@Override
public void batch(Statement statement) throws SQLException {
    // 直接调用PreparedStatement的addBatch()
    PreparedStatement ps = (PreparedStatement) statement;
    ps.addBatch();
}
```



### **2. 与SimpleExecutor的对比**

java

```
// SimpleExecutor：立即执行
public int doUpdate(...) {
    Statement stmt = prepareStatement(handler, ms.getStatementLog());
    stmt.executeUpdate();  // 立即执行
    return stmt.getUpdateCount();  // 返回实际影响行数
}

// BatchExecutor：延迟执行
public int doUpdate(...) {
    // ... 准备Statement ...
    handler.batch(currentStatement);  // 只是添加到批处理
    return BATCH_UPDATE_RETURN_VALUE; // 返回特殊标记
}
```



## 五、**BatchExecutor 的特别返回值**

java

```
// 特殊返回值常量
public static final int BATCH_UPDATE_RETURN_VALUE = Integer.MIN_VALUE + 1002;

// 为什么需要特殊返回值？
// 因为批量操作不能立即知道影响行数，需要等flush后才知道

// 使用时需要注意：
@Test
public void testBatchUpdate() {
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    
    // 批量插入
    int result1 = mapper.insertUser(user1);  // 返回BATCH_UPDATE_RETURN_VALUE
    int result2 = mapper.insertUser(user2);  // 返回BATCH_UPDATE_RETURN_VALUE
    
    // 此时数据库还没有数据！
    System.out.println("result1: " + result1); // -2147482646
    System.out.println("result2: " + result2); // -2147482646
    
    // 必须手动flush
    List<BatchResult> results = sqlSession.flushStatements();
    
    // 现在才能获取实际影响行数
    for (BatchResult br : results) {
        int[] updateCounts = br.getUpdateCounts();
        System.out.println("实际影响行数: " + Arrays.toString(updateCounts));
    }
}
```



## 六、**批量执行性能优化原理**

### **1. JDBC 批量优化机制**

java

```
// BatchExecutor 利用JDBC的批处理特性
PreparedStatement ps = connection.prepareStatement(sql);

// 传统方式：每条记录执行一次
for (User user : userList) {
    ps.setString(1, user.getName());
    ps.setInt(2, user.getAge());
    ps.executeUpdate();  // 每次都有网络往返
}

// 批处理方式：累积后一次执行
for (User user : userList) {
    ps.setString(1, user.getName());
    ps.setInt(2, user.getAge());
    ps.addBatch();  // 只累积，不执行
}
int[] results = ps.executeBatch();  // 一次执行所有

// 性能提升原因：
// 1. 减少网络往返次数（N次 → 1次）
// 2. 数据库可以优化执行计划
// 3. 减少事务开销
```



### **2. 数据库批量优化支持**

java

```
// MySQL需要添加rewriteBatchedStatements参数
String url = "jdbc:mysql://localhost:3306/test?rewriteBatchedStatements=true";

// 不同数据库的批处理支持：
// MySQL:    需要rewriteBatchedStatements=true才能真正批量
// Oracle:   原生支持批处理
// PostgreSQL: 原生支持
// SQL Server: 原生支持
```



## 七、**完整批量操作示例**

### **1. 正确使用方式**

java

```
@Test
public void testBatchInsert() {
    SqlSession sqlSession = sqlSessionFactory.openSession(ExecutorType.BATCH);
    
    try {
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        
        // 批量插入1000条数据
        for (int i = 0; i < 1000; i++) {
            User user = new User();
            user.setName("user" + i);
            user.setAge(20 + i % 30);
            
            // 这里不会立即执行，只是添加到批处理
            mapper.insert(user);
            
            // 每100条提交一次，避免内存溢出
            if (i % 100 == 0) {
                sqlSession.flushStatements();
            }
        }
        
        // 提交剩余的数据
        sqlSession.flushStatements();
        
        // 提交事务
        sqlSession.commit();
        
    } catch (Exception e) {
        sqlSession.rollback();
        throw e;
    } finally {
        sqlSession.close();
    }
}
```



### **2. Mapper XML 配置**

xml

```
<!-- 批量插入Mapper -->
<insert id="batchInsert" parameterType="list">
    INSERT INTO users (name, age, email)
    VALUES
    <foreach collection="list" item="user" separator=",">
        (#{user.name}, #{user.age}, #{user.email})
    </foreach>
</insert>

<!-- 注意：批量插入有SQL长度限制，需要分段 -->
```



## 八、**BatchExecutor 的局限性**

### **1. 查询操作不支持批量**

java

```
// BatchExecutor 只优化更新操作，查询操作不受影响
@Override
public <E> List<E> doQuery(...) {
    // 查询操作走普通流程，不进行批量优化
    return simpleExecutor.doQuery(ms, parameterObject, rowBounds, resultHandler, boundSql);
}
```



### **2. 需要手动控制刷新**

java

```
// 常见错误：忘记flush
public void errorDemo() {
    SqlSession session = factory.openSession(ExecutorType.BATCH);
    
    // 插入数据
    mapper.insert(user1);  // 没执行
    mapper.insert(user2);  // 没执行
    
    // 查询数据（这里会flush之前的操作）
    User user = mapper.selectById(1);  // 自动flush！
    
    session.commit();  // 事务提交也会flush
    session.close();
}
```



### **3. 事务边界需要注意**

java

```
// 批量操作的事务控制
public void batchWithTransaction() {
    SqlSession session = null;
    try {
        session = factory.openSession(ExecutorType.BATCH);
        
        // 批量操作
        for (int i = 0; i < 100; i++) {
            mapper.insert(data[i]);
        }
        
        // 手动flush
        session.flushStatements();
        
        // 其他业务操作
        mapper.updateSomething();
        
        // 提交事务（包含所有操作）
        session.commit();
        
    } catch (Exception e) {
        if (session != null) {
            // 回滚会清空所有批处理
            session.rollback();
        }
        throw e;
    } finally {
        if (session != null) {
            session.close();
        }
    }
}
```



## 九、**批量操作最佳实践**

### **1. 性能优化建议**

java

```
public class BatchOptimizer {
    
    // 1. 合适的批量大小
    private static final int BATCH_SIZE = 1000;
    
    // 2. 分批次提交
    public void batchInsert(List<User> users) {
        SqlSession session = sqlSessionFactory.openSession(ExecutorType.BATCH);
        
        try {
            UserMapper mapper = session.getMapper(UserMapper.class);
            
            for (int i = 0; i < users.size(); i++) {
                mapper.insert(users.get(i));
                
                // 分批次提交
                if (i > 0 && i % BATCH_SIZE == 0) {
                    session.flushStatements();
                }
            }
            
            // 提交剩余
            session.flushStatements();
            session.commit();
            
        } finally {
            session.close();
        }
    }
    
    // 3. 使用rewriteBatchedStatements（MySQL）
    public DataSource createDataSource() {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:mysql://localhost:3306/test" +
                         "?rewriteBatchedStatements=true" +
                         "&useServerPrepStmts=true" +
                         "&cachePrepStmts=true");
        return new HikariDataSource(config);
    }
}
```



### **2. 错误处理机制**

java

```
public class BatchErrorHandler {
    
    public void safeBatchInsert(List<User> users) {
        SqlSession session = null;
        try {
            session = sqlSessionFactory.openSession(ExecutorType.BATCH);
            UserMapper mapper = session.getMapper(UserMapper.class);
            
            for (int i = 0; i < users.size(); i++) {
                try {
                    mapper.insert(users.get(i));
                    
                    // 每100条提交一次，失败时可以重试
                    if (i % 100 == 0) {
                        session.flushStatements();
                    }
                    
                } catch (Exception e) {
                    // 记录失败位置，可以重试
                    log.error("Batch insert failed at index: " + i, e);
                    session.rollback();
                    // 可以重新尝试从失败位置开始
                    retryFromIndex(i);
                    return;
                }
            }
            
            session.flushStatements();
            session.commit();
            
        } finally {
            if (session != null) {
                session.close();
            }
        }
    }
}
```



## 十、**与其他批量方式的对比**

### **1. 三种批量插入方式对比**

| 方式              | 原理                                      | 优点                 | 缺点         | 适用场景     |
| :---------------- | :---------------------------------------- | :------------------- | :----------- | :----------- |
| **foreach标签**   | 生成 `INSERT INTO ... VALUES (...),(...)` | 一次SQL执行          | SQL长度限制  | 数据量不大   |
| **BatchExecutor** | JDBC `addBatch()` + `executeBatch()`      | 性能好，支持大数据量 | 需要手动控制 | 大数据量插入 |
| **多值插入**      | `INSERT ALL INTO ... INTO ... SELECT`     | 数据库级优化         | 语法复杂     | Oracle数据库 |

### **2. 性能测试对比**

java

```
// 测试10000条数据插入
@Test
public void performanceTest() {
    // 方式1：foreach标签
    long time1 = testForeachInsert(10000);  // ~2秒
    
    // 方式2：BatchExecutor
    long time2 = testBatchExecutor(10000);  // ~0.5秒
    
    // 方式3：逐条插入
    long time3 = testSingleInsert(10000);   // ~10秒
    
    System.out.println("foreach标签: " + time1 + "ms");
    System.out.println("BatchExecutor: " + time2 + "ms");
    System.out.println("逐条插入: " + time3 + "ms");
}
```



## **总结**

### **BatchExecutor 核心原理总结**

1. **延迟执行机制**：通过 `addBatch()` 累积SQL，`executeBatch()` 批量执行
2. **Statement重用**：相同SQL重用同一个PreparedStatement
3. **特殊返回值**：使用 `BATCH_UPDATE_RETURN_VALUE` 标记批量操作
4. **手动控制**：需要显式调用 `flushStatements()` 来执行批量操作
5. **性能优化**：大幅减少网络往返和数据库事务开销

### **使用建议**

1. **大数据量操作**时使用BatchExecutor
2. **注意事务边界**，及时flush和commit
3. **合理设置批量大小**，避免内存溢出
4. **配合数据库参数**优化（如MySQL的rewriteBatchedStatements）
5. **做好错误处理**，支持重试机制

BatchExecutor 是MyBatis针对批量操作的重要优化手段，理解其原理能够帮助我们在实际开发中更好地处理大数据量场景。