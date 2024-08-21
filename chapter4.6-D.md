# 文档数据库事务处理
文档数据库系统使用前文介绍的日志机制和并发控制机制能够保证单个文档访问操作的原子性。由于文档模型的灵活性，我们可以使用包含嵌套文档和数组的单个文档模型来描述业务应用数据。在合理的建模和消息队列的使用下，单文档访问操作可以满足大多数应用的业务逻辑。然而，对于一些特殊的应用场景（比如金融、会计等）需要对多个文档（在单个或多个集合中）进行读写操作。为了实现在多个文档、集合、数据库的读写操作的原子性，文档数据库系统支持了多文档事务。本小节以mongoDB为例，介绍文档数据库的多文档事务处理。

## 多文档事务

多文档事务是由对多个文档的增删改查操作组成，这些文档可以来自不同的集合和不同的数据库。在mongoDB中，多文档事务的定义包含以下三个接口：

```SQL
startTransaction() /*事务开始*/
commitTransaction() /*事务提交*/
abortTransaction() /*事务回滚*/
```

事务由startTransaction()和commitTransaction()/abortTransaction()之间的所有操作组成，其中startTransaction()表示事务开始，commitTransaction()表示事务中的所有操作都执行成功，事务正常结束，abortTransaction()表示事务因故障无法执行成功，则撤销已经完成的操作，使数据库回滚到事务开始之前的状态。

下面给出转账交易的多文档事务定义：

```SQL
/*创建存款文档集(deposit)，并插入两个文档*/
db.getSiblingDB("mydb1").deposit.insertMany([  
    {"id": "U1","balance":100},
    {"id": "U2","balance":10}
])

/*转账事务定义，U1用户向U2用户转账50*/
session = db.getMongo().startSession();  /*开启会话*/
coll1 = session.getDatabase("mydb1").deposit;   /*获取会话访问的文档集deposit*/
session.startTransaction(); /*开启事务*/
B1=coll1.find( {"id":"U1"},{"balance":1} ); /*查询U1用户的账户余额*/
if(B1.balance < 50)
{
	session.abortTransaction();  /*如果U1账户余额小于50，则事务回滚*/
}
else{
   	coll1.updateOne( {"id":"U1"}, {$inc:{"balance":-50}} ); /*U1账户余额减50*/
   	coll1.updateOne( {"id":"U2"}, {$inc:{"balance":50}} ); /*U2账户余额加50*/
   	session.commitTransaction();  /*提交事务*/
}
session.endSession(); /*结束会话*/
```

## 多文档事务处理

多文档事务包含对不同数据库、集合的多个文档的更新、删除、新增和查询操作，单文档的原子性机制无法保证多文档事务的原子性。因此，文档数据库系统将多文档事务的所有操作看作一个整体，使用前文中介绍的日志机制和并发控制机制来保证多文档事务的原子性，避免因故障或并发冲突操作影响数据访问的正确性。每个文档数据库系统使用的日志机制和并发控制机制略有不同，详细内容，读者可阅读对应系统的相关文档。

[**上一页<<**](chapterD1.2.md) | [**>>下一页**](chapterD5.1.md)



