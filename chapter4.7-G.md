# 图数据库事务处理
图数据库管理系统同样提供了事务处理功能。图数据库会记录所有的写入操作日志，并对数据访问进行并发控制，以保证图数据库事务的ACID属性。本节以Cypher为例，介绍图数据库的事务处理机制。

## 事务

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

## 事务处理
图数据库通常也使用日志机制来保证事务的原子性和持久性。每当一个事务开始时，图数据库会记录事务的日志，包括事务开始、事务中的所有操作以及事务结束。当事务提交时，图数据库会将日志写入磁盘，确保事务的持久性。如果事务失败或系统崩溃，图数据库可以通过日志回滚未完成的事务，确保事务的原子性。图数据库使用并发控制机制来保证事务的隔离性。不同于关系数据库和文档数据库，图数据库根据图数据结构和特性对事务处理进行了特定优化，以减少日志记录和并发锁带来的开销。关于这些优化的详细内容，读者可以参考相应系统的官方文档及相关学术论文。
<!-- 在图数据库中，事务是基于锁定机制来实现隔离性的。当一个事务访问某个节点或关系时，图数据库会自动为该节点或关系加锁，防止其他事务对其进行修改，直到当前事务提交或回滚。这种锁定机制确保了事务的隔离性，避免了并发访问导致的数据不一致问题。 -->

[**上一页<<**](chapter4.6-D.md) | [**>>下一页**](chapter5.1.md)



