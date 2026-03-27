---
title: 2、CppTrader交易系统技术架构设计
createTime: 2026/03/23 11:27:45
permalink: /project-docs/Opensource/cpptrader/CppTrader交易系统技术架构设计/
icon: streamline-ultimate-color:responsive-design-1
order: 2
---


## 股票符号定义

股票代表交易所中的一个交易标的，如股票、ETF等。

设计特点：
- 使用固定大小数组存储名称（8字节），避免动态内存分配
- 支持高性能的复制和比较操作
- 适用于高频交易场景，减少内存碎片和分配开销

NASDAQ ITCH 协议中，股票代码通常为1-5个字符，如 "AAPL"、"MSFT"，8字节足以容纳最常见的股票代码（包括空字符终止符）；

```c++
uint32_t Id; //股票唯一标识符（Symbol Id）
char Name[8] //固定8字节字符数组，存储股票代码如 "AAPL", "MSFT"
```

### 设计要点解析

1、为什么使用固定大小数组而不是 std::string？


| 特性       | std::string  | char[8]       |
| :--------- | :----------- | :------------ |
| 内存分配   | 堆分配       | 栈分配        |
| 缓存友好性 | 差           | 好            |
| 拷贝开销   | 需要分配内存 | 直接复制8字节 |
| 性能       | 较慢         | 极快          |
| 灵活性     | 可变长度     | 固定长度      |
| 适用场景   | 通用场景     | 高频交易      |


在高频交易系统中，性能是首要目标，固定大小数组避免了：

1. 动态内存分配（减少延迟）
2. 内存碎片（提高缓存命中率）
3. 间接访问（减少指针跳转）

2、内存布局优化

```c++
// Symbol 对象内存布局（64位系统）
struct Symbol {
    uint32_t Id;    // 偏移 0-3 字节
    char Name[8];   // 偏移 4-11 字节
};                  // 总大小 12 字节
                    // 可能对齐到 16 字节（取决于编译器）
```

这种紧凑布局使得：

1. 单个Symbol对象可以放入一个CPU缓存行（通常64字节）
2. 多个Symbol可以连续存储，提高遍历效率
3. 复制操作只需复制12-16字节

3、noexcept 的重要性

在所有函数上使用 noexcept 确保：

1. 编译器可以生成更优化的代码
2. 在性能关键路径上避免异常处理开销
3. 明确表明这些操作是安全的

## 价格层级设计




### 数据结构层次关系

**成员设计**:

```c++
struct Level
{
    //! Level type 行情类型
    //! 层级类型（买单或卖单）
    LevelType Type;
    //! Level price 行情价格
    //! 层级价格（该价格水平的所有订单价格相同）
    uint64_t Price;
    //! Level volume 行情总量
     //! 总成交量（该价格水平所有订单的总数量）
    uint64_t TotalVolume;
    //! 隐藏成交量（冰山订单中不可见的部分）
    //! Level hidden volume 隐藏量
    uint64_t HiddenVolume;
    //! 可见成交量（冰山订单中可见的部分）
    //! Level visible volume
    uint64_t VisibleVolume; //可见量
    //! 订单数量（该价格水平上的订单个数）
    //! Level orders 行情订单
    size_t Orders;
};
```

**数据结构设计**:

```c++
Level (基础层级信息)
   ↑
LevelNode (层级节点)
   ├── 继承 Level (价格、成交量等)
   ├── 继承 AVL Node (树节点功能)
   └── 包含 OrderList (该层级的订单链表)
```

### AVL树与链表结合

```c++
// 订单簿的数据结构
Levels _bids;  // AVL树，按价格排序
  └── LevelNode (价格 $100.50)
       └── OrderList: [Order1, Order2, Order3]  // 时间优先
  └── LevelNode (价格 $100.49)
       └── OrderList: [Order4, Order5]
```

### 价格优先 + 时间优先

```c++
// 价格优先：AVL树按价格排序
// 买单：高价格优先（通过反向迭代器）
// 卖单：低价格优先（通过正向迭代器）

// 时间优先：同一价格内使用链表（FIFO）
OrderList: [最早订单, 中间订单, 最新订单]
```

### 冰山订单支持

```c++
struct Level {
    uint64_t TotalVolume;   // 总数量 = 可见 + 隐藏
    uint64_t VisibleVolume; // 可见数量（显示在订单簿）
    uint64_t HiddenVolume;  // 隐藏数量（冰山订单未显示部分）
    
    // 冰山订单行为：
    // 1. 初始显示 VisibleVolume
    // 2. VisibleVolume 被消耗完后
    // 3. 从 HiddenVolume 补充到 VisibleVolume
    // 4. 重复直到订单完全成交
};
```

### 内存布局优化

```c++
// Level 结构体大小（64位系统）
struct Level {
    LevelType Type;      // 1字节（实际可能对齐到4）
    // 3字节填充
    uint64_t Price;      // 8字节
    uint64_t TotalVolume;// 8字节
    uint64_t HiddenVolume;// 8字节
    uint64_t VisibleVolume;// 8字节
    size_t Orders;       // 8字节（64位）
}; // 总共约 48字节

// LevelNode 继承 Level 并添加：
// - AVL树节点指针（左右父指针，平衡因子）
// - 订单链表头指针
// 总大小约 72-96字节，适合缓存行
```

### 更新事件机制

```c++
// 当价格层级发生变化时，创建 LevelUpdate 对象
LevelUpdate update(UpdateType::ADD, level, is_top);
// 通过 MarketHandler 通知订阅者
market_handler.onUpdateLevel(order_book, update.Update, update.Top);
```

### noexcept 的重要性

所有操作都标记为 noexcept 确保：

1. 性能：编译器可以生成更优化的代码
2. 安全：在高频交易中避免异常传播
3. 可预测：延迟确定性更高

这个设计实现了高性能订单簿的核心数据结构，通过组合 AVL 树和链表，在保证排序的同时实现了 O(log n) 的查找和 O(1) 的订单插入/删除。

## 订单簿设计

### 订单簿整体架构

在交易系统中，每只股票都有自己独立的订单簿，这是标准的市场架构设计。因此通常配置一个市场管理器,管理不同的股票和订单簿；

```c++
// 从 market_manager.h 可以看到关键容器
class MarketManager {
private:
    // 股票容器（按索引访问）
    Symbols _symbols;           // vector<Symbol*>
    
    // 订单簿容器（与股票一一对应，相同索引）
    OrderBooks _order_books;    // vector<OrderBook*>
    
    // 全局订单索引（快速查找任意订单）
    Orders _orders;             // hashmap<order_id, OrderNode*>
    
    // 通过股票ID快速获取订单簿
    const OrderBook* GetOrderBook(uint32_t id) const noexcept;
};
```

多订单簿的设计核心要点：

1. 一对一映射：每只股票一个独立的订单簿
2. 高效查找：通过股票ID直接索引（O(1)）
3. 内存优化：使用vector连续存储，缓存友好
4. 独立匹配：各订单簿独立进行价格发现
5. 全局索引：统一的订单索引用于快速访问
6. 可扩展性：支持分片和并发优化

这种设计既保证了性能（O(1)查找），又保证了隔离性（不同股票的订单不会互相干扰），是交易系统的标准实践。

### 数据成员设计

```c++

class OrderBook
{
    // 声明 MarketManager 为友元类，允许其访问私有成员
    friend class MarketManager;
  
private:
    // Market manager // 市场管理器引用，用于触发市场事件回调
    MarketManager& _manager;

    // Order book symbol 该订单簿对应的股票符号
    Symbol _symbol;

     // ==================== 普通订单簿（限价单） ====================
    // Bid/Ask Price Levels（买卖盘口价格）指的是在订单簿中，不同价格上等待成交的买入和卖出委托单。
    //它们是市场深度的直接体现，反映了各个价位上的供需关系
    LevelNode* _best_bid;//最佳买价层级指针（买一价格）
    LevelNode* _best_ask;// 最佳卖价层级指针（卖一价格）
    // AVl平衡二叉树结构
    Levels _bids;// 买单层级容器（AVL树，按价格降序）
    Levels _asks;// 卖单层级容器（AVL树，按价格升序）

     // ==================== 止损订单簿 ====================
    // Buy/Sell stop orders levels
    LevelNode* _best_buy_stop;// 最佳买入止损价格层级
    LevelNode* _best_sell_stop;// 最佳卖出止损价格层级
    Levels _buy_stop;// 买入止损单容器（按触发价格排序）
    Levels _sell_stop;// 卖出止损单容器（按触发价格排序）

        // ==================== 跟踪止损订单簿 ====================

    // Buy/Sell trailing stop orders levels
    LevelNode* _best_trailing_buy_stop; // 最佳跟踪买入止损价格层级
    LevelNode* _best_trailing_sell_stop;// 最佳跟踪卖出止损价格层级
    Levels _trailing_buy_stop;// 跟踪买入止损单容器
    Levels _trailing_sell_stop;// 跟踪卖出止损单容器

     // ==================== 市场价格跟踪 ====================
    
    // 最后成交的买价（用于跟踪止损计算）
    // Market last and trailing prices
    uint64_t _last_bid_price;
    uint64_t _last_ask_price;// 最后成交的卖价
    uint64_t _matching_bid_price;// 匹配时的买价
    uint64_t _matching_ask_price; // 匹配时的卖价
    uint64_t _trailing_bid_price;// 跟踪止损的买价参考
    uint64_t _trailing_ask_price;// 跟踪止损的卖价参考

};

} // namespace Matching
} // namespace CppTrader
```

### 订单簿多层次设计

```yaml
OrderBook (订单簿)
├── 限价单簿 (Limit Orders)
│   ├── Bids (买单) - 价格降序
│   └── Asks (卖单) - 价格升序
├── 止损单簿 (Stop Orders)
│   ├── Buy Stop (买入止损)
│   └── Sell Stop (卖出止损)
└── 跟踪止损单簿 (Trailing Stop Orders)
    ├── Trailing Buy Stop
    └── Trailing Sell Stop
```

### 价格层级管理

使用 AVL平衡二叉树 存储价格层级：

- 插入/删除/查找：O(log n) 时间复杂度
- 自动排序：支持价格优先匹配
- 最佳价格快速访问：通过 _best_bid 和 _best_ask 指针

### 订单类型详解

| 订单类型       | 触发条件 | 执行方式   | 用途       |
| :------------- | :------- | :--------- | :--------- |
| **限价单**     | 立即     | 指定价格   | 提供流动性 |
| **市价单**     | 立即     | 最优价格   | 快速成交   |
| **止损单**     | 价格触及 | 转为市价单 | 止损保护   |
| **止损限价单** | 价格触及 | 转为限价单 | 精确止损   |
| **跟踪止损单** | 价格回撤 | 动态触发   | 趋势跟踪   |


### 跟踪止损计算逻辑

```c++
// 买入跟踪止损：价格回撤到一定幅度时触发买入
trigger_price = highest_price_since_order * (1 - trailing_percent)

// 卖出跟踪止损：价格回撤到一定幅度时触发卖出
trigger_price = lowest_price_since_order * (1 + trailing_percent)
```

### 冰山订单支持

通过 hidden 和 visible 参数实现：

- 只显示部分数量（visible）

- 隐藏剩余数量（hidden）

- 当可见数量消耗完，自动补充

### 性能优化设计

1. 内存池分配：减少内存分配开销

2. AVL树索引：快速价格查找

3. 最佳价格指针：O(1) 获取买卖一价格

4. 友元类设计：允许 MarketManager 高效访问

这个类实现了一个完整的订单簿数据结构，是高频交易引擎的核心组件，支持多种订单类型和复杂的止损逻辑。

## 市场管理器设计

### 数据结构设计

```c++
namespace CppTrader {
namespace Matching {
class MarketManager
{
    friend class OrderBook;// 允许 OrderBook 访问私有成员

public:
    //! Symbols container 符号容器类型（存储股票符号指针的向量）
    typedef std::vector<Symbol*> Symbols;
    //! Order books container 订单簿容器类型（存储订单簿指针的向量）
    typedef std::vector<OrderBook*> OrderBooks;
    //! Orders container 订单容器类型（哈希映射：订单ID -> 订单节点）
    typedef CppCommon::HashMap<uint64_t, OrderNode*, FastHash> Orders;

     // ==================== 查询方法 ====================
    const Symbols& symbols() const noexcept { return _symbols; }
    const OrderBooks& order_books() const noexcept { return _order_books; }
    const Orders& orders() const noexcept { return _orders; }
    
    const Symbol* GetSymbol(uint32_t id) const noexcept;
    const OrderBook* GetOrderBook(uint32_t id) const noexcept;
    const Order* GetOrder(uint64_t id) const noexcept;

    // ==================== 符号管理方法 ====================
    ErrorCode AddSymbol(const Symbol& symbol);
    ErrorCode DeleteSymbol(uint32_t id);

    // ==================== 订单簿管理方法 ====================
    ErrorCode AddOrderBook(const Symbol& symbol);
    ErrorCode DeleteOrderBook(uint32_t id);

    // ==================== 订单管理方法 ====================
    ErrorCode AddOrder(const Order& order);
    ErrorCode ReduceOrder(uint64_t id, uint64_t quantity);
    ErrorCode ModifyOrder(uint64_t id, uint64_t new_price, uint64_t new_quantity);
    ErrorCode MitigateOrder(uint64_t id, uint64_t new_price, uint64_t new_quantity);
    ErrorCode ReplaceOrder(uint64_t id, uint64_t new_id, uint64_t new_price, uint64_t new_quantity);
    ErrorCode ReplaceOrder(uint64_t id, const Order& new_order);
    ErrorCode DeleteOrder(uint64_t id);

    // ==================== 订单执行方法 ====================
    ErrorCode ExecuteOrder(uint64_t id, uint64_t quantity);
    ErrorCode ExecuteOrder(uint64_t id, uint64_t price, uint64_t quantity);

     // ==================== 订单匹配控制 ====================
    bool IsMatchingEnabled() const noexcept { return _matching; }
    void EnableMatching() { _matching = true; Match(); }
    void DisableMatching() { _matching = false; }
    void Match();

private:
    // Market handler/ 默认市场事件处理器（单例模式）
    static MarketHandler _default;
    //// 市场事件处理器引用（用户自定义或默认）
    MarketHandler& _market_handler;
    // Auxiliary memory manager 辅助内存管理器（用于其他内存分配）
    CppCommon::DefaultMemoryManager _auxiliary_memory_manager;

    // ==================== 价格层级内存管理 ====================
    // 价格层级池（用于订单簿中的买/卖盘层级）
    // Bid/Ask price levels
    CppCommon::PoolMemoryManager<CppCommon::DefaultMemoryManager> _level_memory_manager;
    CppCommon::PoolAllocator<LevelNode, CppCommon::DefaultMemoryManager> _level_pool;

     // ==================== 符号内存管理 ====================
    // 符号对象池（减少内存分配开销）
    // Symbols
    CppCommon::PoolMemoryManager<CppCommon::DefaultMemoryManager> _symbol_memory_manager;
    CppCommon::PoolAllocator<Symbol, CppCommon::DefaultMemoryManager> _symbol_pool;
    Symbols _symbols;

     // ==================== 订单簿内存管理 ====================
    // 订单簿对象池
    // Order books
    CppCommon::PoolMemoryManager<CppCommon::DefaultMemoryManager> _order_book_memory_manager;
    CppCommon::PoolAllocator<OrderBook, CppCommon::DefaultMemoryManager> _order_book_pool;
    OrderBooks _order_books;

     // ==================== 订单内存管理 ====================
    // 订单节点池
    // Orders
    CppCommon::PoolMemoryManager<CppCommon::DefaultMemoryManager> _order_memory_manager;
    CppCommon::PoolAllocator<OrderNode, CppCommon::DefaultMemoryManager> _order_pool;
    Orders _orders;

    ErrorCode AddMarketOrder(const Order& order, bool recursive);
    ErrorCode AddLimitOrder(const Order& order, bool recursive);
    ErrorCode AddStopOrder(const Order& order, bool recursive);
    ErrorCode AddStopLimitOrder(const Order& order, bool recursive);
    ErrorCode ReduceOrder(uint64_t id, uint64_t quantity, bool recursive);
    ErrorCode ModifyOrder(uint64_t id, uint64_t new_price, uint64_t new_quantity, bool mitigate, bool recursive);
    ErrorCode ReplaceOrder(uint64_t id, uint64_t new_id, uint64_t new_price, uint64_t new_quantity, bool recursive);
    ErrorCode DeleteOrder(uint64_t id, bool recursive);

      // ==================== 订单匹配相关方法 ====================
    
    // 是否启用自动匹配的标志
    // Matching
    bool _matching;

    void Match(OrderBook* order_book_ptr);
    void MatchMarket(OrderBook* order_book_ptr, Order* order_ptr);
    void MatchLimit(OrderBook* order_book_ptr, Order* order_ptr);
    void MatchOrder(OrderBook* order_book_ptr, Order* order_ptr);
    bool ActivateStopOrders(OrderBook* order_book_ptr);
    bool ActivateStopOrders(OrderBook* order_book_ptr, LevelNode* level_ptr, uint64_t stop_price);
    bool ActivateStopOrder(OrderBook* order_book_ptr, OrderNode* order_ptr);
    bool ActivateStopLimitOrder(OrderBook* order_book_ptr, OrderNode* order_ptr);
    uint64_t CalculateMatchingChain(OrderBook* order_book_ptr, LevelNode* level_ptr, uint64_t price, uint64_t volume);
    uint64_t CalculateMatchingChain(OrderBook* order_book_ptr, LevelNode* bid_level_ptr, LevelNode* ask_level_ptr);
    void ExecuteMatchingChain(OrderBook* order_book_ptr, LevelNode* level_ptr, uint64_t price, uint64_t volume);
    void RecalculateTrailingStopPrice(OrderBook* order_book_ptr, LevelNode* level_ptr);
    void UpdateLevel(const OrderBook& order_book, const LevelUpdate& update) const;
};

} // namespace Matching
} // namespace CppTrader

#include "market_manager.inl"

#endif // CPPTRADER_MATCHING_MARKET_MANAGER_H
```

### 内存管理

- **对象池技术**：使用 `PoolAllocator` 管理符号、订单簿、订单的分配
- **减少内存碎片**：相同类型的对象从同一内存池分配
- **提高缓存命中率**：相关对象在内存中连续存储

### 订单支持类型

- **市价单**：立即以当前最佳价格成交
- **限价单**：指定价格成交，贡献订单簿深度
- **止损单**：触发价格达到后激活
- **止损限价单**：止损单和限价单的组合

### 核心匹配算法

```c++
// 匹配逻辑：价格优先 + 时间优先
void Match() {
    // 1. 遍历所有订单簿
    // 2. 检查买卖盘是否交叉（bid >= ask）
    // 3. 从最优价格开始逐级匹配
    // 4. 生成成交记录
}
```

### **关键特性**

- **飞行中缓和**：防止修改订单时被错误成交
- **递归匹配**：订单成交后可能触发新的匹配机会
- **事件驱动**：所有状态变更都通过回调通知

### **性能设计**

- 非线程安全，避免锁开销
- 内联函数（`.inl` 文件）减少函数调用
- 哈希表快速订单查找 O(1)
- 向量容器按索引访问 O(1)

这个类实现了一个**完整的订单簿匹配引擎**，是高频交易系统的核心组件。


## 市场处理器设计

MarketHandler 是一个抽象基类，用于接收和处理来自 MarketManager 的所有市场事件。用户通过继承此类并重写相应的事件处理函数，可以实现自定义的市场监控逻辑。

可以监控的市场变化包括：

- 股票的增删改
- 订单的增删改
- 订单成交
- 订单簿的更新

### 属性和方法设计

```c++
namespace CppTrader {
namespace Matching {

/*!
 * @brief 市场事件处理器基类
 * 
 * MarketHandler 是一个抽象基类，用于接收和处理来自 MarketManager 的所有市场事件。
 * 用户通过继承此类并重写相应的事件处理函数，可以实现自定义的市场监控逻辑。
 * 
 * 可以监控的市场变化包括：
 * - 股票的增删改
 * - 订单的增删改
 * - 订单成交
 * - 订单簿的更新
 * 
 * 设计模式：观察者模式（Observer Pattern）
 * - MarketManager 是被观察者（Subject）
 * - MarketHandler 是观察者（Observer）
 * 
 * 使用场景：
 * 1. 订单簿深度监控：统计买卖盘口变化
 * 2. 交易策略实现：根据市场事件触发交易决策
 * 3. 性能监控：统计市场事件处理延迟
 * 4. 数据记录：将市场事件写入日志或数据库
 * 5. 风险控制：检测异常交易行为
 * 
 * Not thread-safe. 非线程安全
 */
class MarketHandler
{
    // 声明 MarketManager 为友元类，允许其调用受保护的虚函数
    friend class MarketManager;

public:
    /*!
     * @brief 默认构造函数
     * 使用默认实现，初始化一个空的处理器
     */
    MarketHandler() = default;
    
    // 禁用拷贝构造函数（避免意外的复制）
    MarketHandler(const MarketHandler&) = delete;
    
    // 禁用移动构造函数
    MarketHandler(MarketHandler&&) = delete;
    
    /*!
     * @brief 虚析构函数
     * 使用默认实现，确保派生类对象正确析构
     * virtual 保证通过基类指针删除派生类对象时，调用正确的析构函数
     */
    virtual ~MarketHandler() = default;
    
    // 禁用拷贝赋值运算符
    MarketHandler& operator=(const MarketHandler&) = delete;
    
    // 禁用移动赋值运算符
    MarketHandler& operator=(MarketHandler&&) = delete;

protected:
    // ==================== 股票事件处理器 ====================
    
    /*!
     * @brief 股票添加事件
     * 当新的股票添加到市场时触发
     * 
     * @param symbol 被添加的股票对象
     * 
     * 触发时机：MarketManager::AddSymbol()
     * 应用场景：
     * - 初始化该股票的订单簿监控
     * - 记录股票信息到数据库
     * - 加载该股票的历史数据
     * 
     * 示例：
     * @code
     * virtual void onAddSymbol(const Symbol& symbol) override {
     *     std::cout << "New symbol added: " << symbol.Name << std::endl;
     *     _symbol_count++;
     * }
     * @endcode
     */
    virtual void onAddSymbol(const Symbol& symbol) {}
    
    /*!
     * @brief 股票删除事件
     * 当股票从市场移除时触发
     * 
     * @param symbol 被删除的股票对象
     * 
     * 触发时机：MarketManager::DeleteSymbol()
     * 应用场景：
     * - 清理该股票相关的监控资源
     * - 记录股票下市信息
     * - 归档该股票的订单簿数据
     */
    virtual void onDeleteSymbol(const Symbol& symbol) {}

    // ==================== 订单簿事件处理器 ====================
    
    /*!
     * @brief 订单簿添加事件
     * 当为股票创建新的订单簿时触发
     * 
     * @param order_book 被添加的订单簿对象
     * 
     * 触发时机：MarketManager::AddOrderBook()
     * 注意：通常与 onAddSymbol 成对出现
     * 应用场景：
     * - 初始化订单簿数据结构
     * - 开始监控该订单簿的深度变化
     */
    virtual void onAddOrderBook(const OrderBook& order_book) {}
    
    /*!
     * @brief 订单簿更新事件
     * 当订单簿的状态发生变化时触发（买卖盘变化）
     * 
     * @param order_book 发生变化的订单簿对象
     * @param top 是否为最优价格变化（买一/卖一变化）
     * 
     * 触发时机：
     * - 订单添加/删除/修改
     * - 订单成交
     * - 价格层级变化
     * 
     * top 参数的作用：
     * - true: 最优价格发生变化（买一/卖一变化）
     * - false: 非最优价格变化（深度的变化）
     * 
     * 性能优化：通过 top 参数可以只关注最重要的价格变化
     * 
     * 应用场景：
     * - 实时行情推送（Level 1 数据）
     * - 订单簿深度监控（Level 2 数据）
     * - 策略信号触发
     */
    virtual void onUpdateOrderBook(const OrderBook& order_book, bool top) {}
    
    /*!
     * @brief 订单簿删除事件
     * 当订单簿被移除时触发
     * 
     * @param order_book 被删除的订单簿对象
     * 
     * 触发时机：MarketManager::DeleteOrderBook()
     * 应用场景：
     * - 停止监控该订单簿
     * - 清理相关资源
     */
    virtual void onDeleteOrderBook(const OrderBook& order_book) {}

    // ==================== 价格层级事件处理器 ====================
    
    /*!
     * @brief 价格层级添加事件
     * 当新的价格层级创建时触发（该价格第一次出现订单）
     * 
     * @param order_book 所属的订单簿
     * @param level 添加的价格层级信息
     * @param top 是否为新添加的最优价格
     * 
     * 触发时机：
     * - 订单添加到新的价格水平
     * - 之前为空的价格水平出现订单
     * 
     * 示例：
     * 订单簿原来没有 $100.50 的卖单
     * 添加一个 $100.50 的卖单 → 触发 onAddLevel
     */
    virtual void onAddLevel(const OrderBook& order_book, const Level& level, bool top) {}
    
    /*!
     * @brief 价格层级更新事件
     * 当价格层级的数量发生变化时触发
     * 
     * @param order_book 所属的订单簿
     * @param level 更新后的价格层级信息
     * @param top 是否为最优价格变化
     * 
     * 触发时机：
     * - 同一价格上添加新订单（数量增加）
     * - 同一价格上订单部分成交或取消（数量减少）
     * - 冰山订单的可见数量更新
     * 
     * 应用场景：
     * - 监控订单簿深度的动态变化
     * - 计算买卖盘失衡指标
     * - 检测大单进出
     * 
     * 示例：
     * @code
     * virtual void onUpdateLevel(const OrderBook& order_book, 
     *                            const Level& level, bool top) override {
     *     if (level.VisibleVolume > 10000) {
     *         // 检测到大单，发送告警
     *         alertLargeOrder(level.Price, level.VisibleVolume);
     *     }
     * }
     * @endcode
     */
    virtual void onUpdateLevel(const OrderBook& order_book, const Level& level, bool top) {}
    
    /*!
     * @brief 价格层级删除事件
     * 当价格层级完全清空时触发
     * 
     * @param order_book 所属的订单簿
     * @param level 被删除的价格层级信息
     * @param top 是否为最优价格被删除
     * 
     * 触发时机：
     * - 该价格的所有订单都被成交或取消
     * - 价格层级从订单簿中移除
     * 
     * 示例：
     * 订单簿中原有 $100.50 的卖单
     * 所有 $100.50 的订单都被成交 → 触发 onDeleteLevel
     */
    virtual void onDeleteLevel(const OrderBook& order_book, const Level& level, bool top) {}

    // ==================== 订单事件处理器 ====================
    
    /*!
     * @brief 订单添加事件
     * 当新订单添加到市场时触发
     * 
     * @param order 被添加的订单对象
     * 
     * 触发时机：MarketManager::AddOrder()
     * 应用场景：
     * - 订单生命周期追踪
     * - 订单量统计
     * - 订单流分析
     * 
     * 示例：
     * @code
     * virtual void onAddOrder(const Order& order) override {
     *     _total_orders++;
     *     if (order.Type == OrderType::LIMIT) {
     *         _limit_orders++;
     *     }
     * }
     * @endcode
     */
    virtual void onAddOrder(const Order& order) {}
    
    /*!
     * @brief 订单更新事件
     * 当订单被修改时触发
     * 
     * @param order 更新后的订单对象
     * 
     * 触发时机：
     * - 订单数量减少（ReduceOrder）
     * - 订单价格或数量修改（ModifyOrder）
     * - 订单缓和修改（MitigateOrder）
     * 
     * 应用场景：
     * - 监控订单修改行为
     * - 检测订单闪烁（Quote Stuffing）
     * - 统计订单修改频率
     */
    virtual void onUpdateOrder(const Order& order) {}
    
    /*!
     * @brief 订单删除事件
     * 当订单从市场移除时触发
     * 
     * @param order 被删除的订单对象
     * 
     * 触发时机：
     * - 订单完全取消（Cancel Order）
     * - 订单被替换（Replace Order）
     * - 订单完全成交后移除
     * 
     * 应用场景：
     * - 订单生命周期追踪
     * - 统计订单存活时间
     * - 检测订单取消率
     */
    virtual void onDeleteOrder(const Order& order) {}

    // ==================== 订单执行事件处理器 ====================
    
    /*!
     * @brief 订单执行事件
     * 当订单部分或全部成交时触发
     * 
     * @param order 被执行（成交）的订单对象
     * @param price 成交价格
     * @param quantity 成交数量
     * 
     * 触发时机：
     * - 市价单立即成交
     * - 限价单与对手方订单匹配成交
     * - 止损单被激活后成交
     * 
     * 注意事项：
     * - 一个订单可能多次触发此事件（分批成交）
     * - 完全成交的订单后续还会触发 onDeleteOrder
     * - 部分成交的订单会触发 onUpdateOrder（数量减少）
     * 
     * 应用场景：
     * - 实时成交记录
     * - 交易成本计算
     * - 滑点监控
     * - 生成交易报告
     * 
     * 示例：
     * @code
     * virtual void onExecuteOrder(const Order& order, 
     *                             uint64_t price, 
     *                             uint64_t quantity) override {
     *     // 记录成交
     *     Trade trade{
     *         .order_id = order.Id,
     *         .symbol = order.StockId,
     *         .price = price,
     *         .quantity = quantity,
     *         .side = order.Side,
     *         .timestamp = getCurrentTime()
     *     };
     *     _trades.push_back(trade);
     *     
     *     // 计算成交均价
     *     _total_volume += quantity;
     *     _total_value += price * quantity;
     * }
     * @endcode
     */
    virtual void onExecuteOrder(const Order& order, uint64_t price, uint64_t quantity) {}
};

} // namespace Matching
} // namespace CppTrader

#endif // CPPTRADER_MATCHING_MARKET_HANDLER_H
```

### 引入的设计模式

观察者模式:

```c++
// 观察者（Observer）：MarketHandler
class MarketHandler {
    virtual void onAddSymbol(...) {}   // 观察事件
    virtual void onAddOrder(...) {}    // 观察事件
};

// 被观察者（Subject）：MarketManager
class MarketManager {
    MarketHandler& _handler;  // 持有观察者引用
    
    void AddSymbol(const Symbol& symbol) {
        // 状态改变
        _handler.onAddSymbol(symbol);  // 通知观察者
    }
};
```

### 事件驱动架构

```c++
MarketHandler (状态变化)
    │
    ├── AddOrder
    │   ├── onAddOrder (订单添加)
    │   ├── onAddLevel (可能的新层级)
    │   ├── onUpdateOrderBook (订单簿更新)
    │   └── onExecuteOrder (可能的立即成交)
    │
    ├── Match (订单匹配)
    │   ├── onExecuteOrder (成交)
    │   ├── onUpdateLevel (层级更新)
    │   ├── onDeleteLevel (层级删除)
    │   └── onUpdateOrderBook (订单簿更新)
    │
    └── DeleteOrder
        ├── onDeleteOrder (订单删除)
        ├── onDeleteLevel (可能的层级删除)
        └── onUpdateOrderBook (订单簿更新)
```

### 核心设计点

1. 虚函数接口：提供可扩展的事件处理机制
2. 空实现默认：派生类只需重写感兴趣的事件
3. 事件分类清晰：股票、订单簿、价格层级、订单、成交
4. 性能友好：通过 top 参数减少不必要的事件处理
5. 友元访问：仅允许 MarketManager 调用受保护方法
6. 禁用拷贝：处理器通常作为单例或引用使用

这个设计实现了灵活的事件处理机制，既保持了高性能，又提供了良好的扩展性，是交易系统架构中的关键组件。