## 4.7 图数据库的事务处理
当前的大部分图数据库系统都能保证单个增、删、改、查操作的原子性。为了更广泛地满足应用的需求，部分图数据库系统（例如Neo4j）也提供了事务处理功能。本节以Neo4j的Cypher语言为例，介绍图数据库的事务处理功能接口。

### 4.7.1 事务的定义

在图数据库中，事务的定义包含以下三个关键字：

```SQL
BEGIN  /*事务开始*/
COMMIT  /*事务提交*/
ROLLBACK  /*事务回滚*/
```

事务由BEGIN和COMMIT/ROLLBACK之间的所有Cypher语句组成。其中，BEGIN表示事务的开始，COMMIT表示事务提交，即事务中所有CREATE、DELETE、SET操作产生的结果已写回磁盘，事务正常结束；ROLLBACK表示事务回滚，即在执行过程中事务因某种故障不能继续执行，那么撤销已经完成的操作，使数据库回滚到事务开始之前的状态。

下面同样给出一个转账交易在图数据库中的事务定义。

```SQL

/*转账事务定义，U1用户向U2用户转账50*/
BEGIN;
MATCH (u:User {id: 'U1'})
RETURN u;
IF (u1.balance < 50) {
    ROLLBACK;
} ELSE {
    MATCH (u1: User {id: 'U1'}), (u2: User {id: 'U2'})
    SET u1.balance = u1.balance - 50, u2.balance = u2.balance + 50;
}
COMMIT;
```

### 4.7.2  事务处理机制

图数据库通常也使用日志机制来保证数据访问操作和事务的原子性、持久性和隔离性。不同于关系数据库和文档数据库，图数据库根据图数据结构和特性对事务处理进行了特定优化，以减少日志记录和并发锁带来的开销。关于这些优化的详细内容，读者可以参考相应系统（例如Neo4j）的官方文档及相关学术论文。
<!-- 在图数据库中，事务是基于锁定机制来实现隔离性的。当一个事务访问某个节点或关系时，图数据库会自动为该节点或关系加锁，防止其他事务对其进行修改，直到当前事务提交或回滚。这种锁定机制确保了事务的隔离性，避免了并发访问导致的数据不一致问题。 -->

### 练习题

**1.** 在一个在线教育平台中，课程购买功能涉及两个节点：`User`（用户）和 `Course`（课程），以及一个关系 `PURCHASED`（购买）。`User` 节点包含属性 `id` (用户ID) 和 `credits` (学分余额)，`Course` 节点包含属性 `id` (课程ID) 和 `price` (课程价格)。

用户购买课程的功能涉及以下步骤：

- 检查用户的学分余额是否足够支付课程价格。
- 如果学分足够，则从用户的学分余额中扣除课程价格。
- 创建一个 `PURCHASED` 关系，连接 `User` 节点和 `Course` 节点，记录购买信息。

请参照本章节介绍的转账事务定义，给出用户购买课程的事务处理代码。

[**上一页<<**](chapter4.6-D.md) | [**>>下一页**](chapter5.1.md)



