## 4.5 关系数据库的事务处理

关系数据管理系统提供了事务处理功能用于确保应用层业务流程的原子性。事务是关系数据库事务处理的最小单元，也是应用程序的基本逻辑单元，它由一系列的数据库增删改查操作组成，用于描述应用层的业务逻辑。本节主要介绍关系数据库的事务基本特性。


### 4.5.1 事务的定义

在关系数据库中，事务的定义包含以下三个关键字：

```SQL
BEGIN TRANSACTION /*事务开始*/
COMMIT /*事务提交*/
ABORT /*事务回滚*/
```
事务由BEGIN TRANSACTION和COMMIT/ABORT之间的所有SQL语句组成。其中，BEGIN TRANSACTION表示事务的开始，COMMIT和ABORT表示事务的结束。COMMIT表示事务提交，即事务中所有UPDATE、DELETE、INSERT操作产生的结果已写回磁盘，事务正常结束；ABORT表示事务回滚，即在执行过程中事务因某种故障不能继续执行，那么撤销已经完成的操作，使数据库回滚到事务开始之前的状态。

下面给出转账交易在关系数据管理系统中的事务定义。
```SQL
[例4.1] 账户U1向账户U2转账50元（事务T0）。
BEGIN TRANSACTION
SELECT Balance A FROM Deposit WHERE ID='U1' FOR UPDATE;
IF(A < 50) 
THEN ROLLBACK; 
ELSE
{
UPDATE Deposit SET  Balance = A - 50 WHERE ID='U1';
UPDATE Deposit SET  Balance = Balance + 50 WHERE ID='U2';
COMMIT;
}
```

### 4.5.2 事务的ACID性质

为了实现数据访问的正确性，数据库管理系统需要保证事务的四个特性：原子性（Atomicity）、一致性（Consistency）、隔离性（Isolation）和持久性（Durability）。这四个特性简称事务的ACID特性。

* 原子性：事务中的所有操作要么都执行，要么都不执行，不会出现中间状态。
* 一致性：事务执行的结果必须保证数据库状态的一致性。比如，A账户向B账户转账100元。该事务执行前和事务执行后两个账户的总额度是不发生变化的，并且账户的余额不能为负数。事务的一致性与原子性是密切相关的，甚至有时需要和应用程序配合才能保证。
* 隔离性：多个并发执行的事务相互之间不受干扰，所有的事务像是按某个顺序执行的，彼此之间是隔离的。事务的隔离级别包括读未提交（Read Uncommitted）、读已提交（Read Committed）、可重复读（Repetable Read）、可串行化（Serializable）、快照读（Snapshot）等。
* 持久性：一个事务一旦提交之后，它对数据库产生的数据修改是永久性的并且不能撤销。之后的其他操作或者故障都不会对执行结果产生任何影响。

### 4.5.3 事务处理机制

在关系型数据库中，事务可以看做一个复杂的数据访问操作，可以使用前文中的日志恢复机制和并发控制机制来保证事务的ACID属性。其中日志恢复机制保证事务的原子性和持久性，并发控制机制保证事务的隔离性。

[**上一页<<**](chapter4.4.md) | [**>>下一页**](chapter4.6-D.md)













