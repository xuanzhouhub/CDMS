# 图数据库的索引

接下来，本节以Neo4j为例，介绍图数据库索引的构建与使用。

## 图数据库索引

Neo4j的索引可以分为两类：搜索性能索引和语义索引。
- 搜索性能索引（Search-performance indexes）：包括范围索引（range）、文本索引（text）、点索引（point）和令牌查找索引（lookup），用于加速基于确切匹配的数据检索。
- 语义索引（Semantic indexes）：包括全文索引（fulltext）和矢量索引（vector），用于近似匹配和计算查询字符串与匹配数据之间的相似度分数。

其中，范围索引、文本索引、点索引和全文索引（下文中，我们将简称为属性索引）实现了从一个属性值到一个实体ID（节点或关系）的映射。当我们根据某个索引标记的属性查找实体时，数据库会在索引中找到对应的ID，然后通过ID再去相应的存储文件中找到该实体所在数据页的地址。这种间接映射让数据存储位置的变化不会影响索引结构。例如，如果数据页调整或数据被重新分配到新的位置，索引仍然有效，因为实体ID保持不变。但是，相比关系数据库和文档数据库，这种索引查找过程会多一步，需要先通过ID找到具体存储位置。

令牌查找索引则不同，它建立的映射关系不是基于属性值，而是基于标签到节点的映射或关系类型到关系的映射。令牌查找索引是默认存在的（一个节点标签查找索引和一个关系类型查找索引），无需手动创建，是Neo4j中最重要的索引，因为它们显著加快了其他索引的填充速度和Cypher查询的性能。

相比于关系数据库和文档数据库，Neo4j提供了多种类型的索引，但大多为非聚簇索引，即索引项与数据项分离，索引只记录属性/标签/关系类型到节点/关系 ID 的映射，而不会改变数据的物理存储顺序。节点和关系是通过图的连接性直接存储，数据的物理存储不依赖于特定属性的排序，数据的访问效率主要依赖于其图结构和索引类型的优化。

## 索引的定义与删除

图数据库系统对外提供了命令允许用户创建索引。图数据库系统Neo4j的索引创建命令为`CREATE ... INDEX ...`，第一个参数指定了索引创建类型，缺省情况下是范围索引，第二个参数是用户指定的索引名称。用户可以根据应用需求创建不同的索引。

（1）建立索引

我们可以使用`CREATE INDEX`来创建范围索引，例如，为标记为`ENROLLS`的关系的`Grade`属性创建一个范围索引：
```cypher
CREATE INDEX rel_range_index_Grade FOR ()-[r:ENROLLS]-() ON (r.Grade)
```

同样，也可以在多个属性上创建复合索引。例如，为标记为`Student`的节点的`Sno`和`Dept`属性创建复合范围索引：
```cypher
CREATE INDEX composite_range_node_index FOR (n:Student) ON (n.Sno, n.Dept)
```

（2）显示索引

如果要列出所有索引，可以执行以下命令：
```cypher
SHOW INDEXES
```

如果要返回可用索引的特定列，可以使用`YIELD`子句。例如，以下语句将返回索引的名称、索引的类型和索引适用于哪些标签和类型：
```cypher
SHOW INDEXES YIELD name, type, labelsOrTypes;

```

同样，也可以通过多种方式对显示的索引进行过滤，例如，`SHOW RANGE INDEXES`将只显示范围索引。另一种更灵活的过滤方式则是使用`WHERE`子句，例如，以下命令将只过滤出与`Student`标签相关的范围索引
```cypher
SHOW RANGE INDEXES WHERE labelsOrTypes CONTAINS 'Student';
```

（3）删除索引

在Neo4j的Cypher中，索引一旦创建后就不能修改，只能先删除现有索引，然后再创建一个新的索引。Cypher使用`DROP INDEX`命令删除索引，其基本格式如下：
```cypher
DROP INDEX index_name [IF EXISTS]
```

例如，删除上面创建的复合索引`composite_range_node_index`：
```cypher
DROP INDEX composite_range_node_index
```

[**上一页<<**](chapter2.6-G.md) | [**>>下一页**](chapter3.1.md)