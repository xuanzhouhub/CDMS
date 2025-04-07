## 1.13 文档数据库查询语言

了解数据模型之后，我们来看看文档数据库提供的基本功能和查询语言。市面上有不少文档数据库系统，如MongoDB、微软的CosmosDB、亚马逊的DocumentDB等。这些系统的基本功能大同小异，但具体的使用接口却不尽相同。由于MongoDB是目前使用最为广泛的文档数据库系统，本节将它作为范例来讲解文档数据库的功能和使用。读者在使用其他文档数据库的时候需查阅相应文档。

数据管理系统最基本的功能是存取数据。文档数据库将数据存取的功能抽象为创建文档（Create）、查询文档（Read）、更新文档（Update）和删除文档（Delete）四类操作，简称为CRUD操作。除了简单的CRUD操作之外，文档数据库还支持复杂的聚合操作。在介绍基本操作和聚合操作之前，我们先来看看文档数据库的数据组织体系。

### 1.13.1  文档的组织体系

上一节讲到文档数据库将数据组织在一个个的文档中。通常，每个文档描述了现实世界中的一个实体或对象。为了方便文档的管理，文档数据库将相同类型的文档归为一类，放置在同一个**文档集**（Collection）中。此外，属于同一个应用的所有文档集又被放置在同一个**数据库**（Database）中。

换言之，一个文档数据库系统通常会维护多个数据库，每个数据库对应一个应用，为这个应用提供数据管理功能。一个数据库包含若干文档集，每个文档集用于存放某一类对象的数据。比如，关于人的数据存放在一个文档集中，关于书的数据存放在另一个文档集中。一个文档集又包含了若干文档，每个文档描述了一个对象，比如某个人或某本书。文档，文档集和数据库三者之间的关系如下：

$$
文档(Document) \in 文档集(Collection) \in 数据库(Database)
$$

具备关系型数据库基础知识的读者不难发现，文档数据库中的**文档集**相当于关系型数据库中的**表**，**文档**则相当于表中的**元组**（表中的行）。

在使用文档数据库系统时，我们首先需要指定一个数据库作为访问对象。在MongoDB中，我们使用下面的指令：

```sql
use myDB
```

其中，myDB是数据库的名称。选择myDB数据库作为访问对象之后，接下来的操作指令都实施在myDB上。对MongoDB而言，如果myDB数据库不存在，那么该指令会在系统中直接创建一个名为myDB的数据库。

当新建一个数据库时，数据库里面没有任何的文档集和文档。因此，需要在该数据库中创建文档集并插入文档。MongoDB一般不用显示地创建文档集，而是在插入文档的同时隐式地创建文档集，具体指令如下：

```sql
db.myNewCollection.insertOne( { x: 1 } )
```

上述指令表示的是向文档集myNewCollection中插入一个文档 {x:1}。如果myNewCollection文档集不存在，系统则会自动创建该文档集。指令中的db指代当前正在使用的数据库，即之前指定的myDB数据库。也就是说，新文档集myNewCollection将被放置在myDB中。

此外，MongoDB也允许用户使用createCollection指令来显示地创建文档集，指令的使用如下:

```sql
db.createCollection("myBook", {max : 5000} )
```

该指令要求系统在当前使用的数据库中创建一个名为myBook的文档集，并且设置这个文档集最多只能容纳5000个文档。

### 1.13.2 文档的创建

本小节以学生-课程数据库为例来介绍文档数据库的CRUD操作。学生-课程文档数据库中包含一个学生文档集Student，学生文档集中记录学生学号、姓名、性别、年龄、系以及学生选课信息，学生选课信息具体包含课程号、课程名、课程学分以及课程成绩，学生的选课信息以文档数组的形式嵌套在学生文档中。学生文档集中的文档结构如下：

```SQL
student文档集中的文档结构
{
  "sno": " ",    	/*学号*/
  "sname": " ", 	/*姓名*/
  "gender": " ",	/*性别*/
  "age": " ",		/*年龄*/
  "department": " ",/*系*/
  "courses": [		/*选课信息*/
      {
      	"cno":" ",	/*课程号*/
      	"cname":" ",/*课程名*/
      	"credit":" ",/*课程学分*/
      	"grade":" "	/*课程成绩*/
      },
     ...
  ]
}
```

文档数据库系统允许用户创建任意形式的文档，并将它放置在任意一个文档集中。在MongoDB中，用户可以使用insertOne指令来创建一个新文档，并将该文档插入某个文档集中。比如：

```sql
[例1.53] 在文档集student中创建一个文档
db.student.insertOne( {
  "sno": "2022001",
  "sname": "沐辰",
  "gender": "male",
  "age": 19,
  "department": "计算机",
  "courses": [
      {"cno":"1","cname":"高数","credit":4,"grade":92},
      {"cno":"3","cname":"数据库","credit":4,"grade": 85}
  ]
} )
```

上例表示创建一个关于"沐辰"的文档，并将它插入到student文档集中，其中沐辰的选课信息courses是文档数组的形式，。上一节讲到，每一个文档有一个"\_id"属性，作为文档的唯一标识。如果在插入文档时用户没有显示地设置"\_id"属性值，那么MongoDB会自动生成一个全局唯一的值并赋给该文档的"\_id"属性。

此外，MongoDB也支持使用insertMany指令向一个文档集中一次性插入多个文档，比如：

```sql
[例1.54] 在文档集中一次性创建多个文档
db.student.insertMany( [
  {"sno": "2022001","sname": "沐辰","gender": "male","age": 19,"department": "计算机", 
    "courses":[
        {"cno":"1","cname":"高数","credit":4,"grade":92},
        {"cno":"3","cname":"数据库","credit":4,"grade": 85}
    ]
  },
  {"sno": "2022123","sname": "浩宇","gender": "male","age": 18, "department": "计算机"},
  {"sno": "2022191","sname": "若汐","gender": "female","age": 18,"department": "数学", "courses":[]}
] )
```

上例表示向文档集student插入三个学生的文档。与传统的关系型数据库不同，文档数据库允许同时插入不同结构的多个文档。

### 1.13.3 文档的查询

文档数据库支持文档读取操作，MongoDB提供find指令来实现文档查询，比如：

```sql
[例1.55] 文档查询
db.student.find( {
  "gender": "female",
  "department": "数学"
} )
```

上例表示在文档集student中查找gender属性为"female"并且department属性为"数学"的文档，实质上就是在文档集student中查询和{"gender":"female", "department": "数学"}相匹配的文档。基于文档匹配运算方式，我们可以这样理解：指令x.find(y)的目的是在x中找到y的所有匹配。上述指令的查询结果为关于"若汐"的文档。

文档匹配是一种基本运算。MongoDB在其上增加了很多灵活性，便于用户表达更广泛的需求。除了上面提到的文档属性的单个值匹配之外，MongoDB还支持属性的多个值匹配、范围匹配等。

MongoDB支持通过关键字**in**来实现属性的多个值匹配。以下指令表示在student文档集中查询department属性为“计算机”或“数学”的文档：

```sql
[例1.56] 多值匹配的文档查询
db.student.find( 
  { "department": { $in: [ "计算机", "数学" ] } } 
)
```

同样地，多值匹配也可以使用逻辑符号**or**来实现：

```sql
[例1.57] 多值匹配的文档查询
db.student.find( 
 { $or: [ { "department": "计算机" }, { "department": "数学" } ] } 
)
```

此外，MongoDB支持使用关键字**lt,lte,gt,gte**来实现范围匹配，其中lt表示小于，lte表示小于等于，gt表示大于，gte表示大于等于。以下指令表示查询在属性age上取值大于18并且小于20的文档：

```sql
[例1.58] 范围匹配的文档查询
db.student.find( 
  { "age": { $gt: 18, $lt: 20} } 
)
```

通常，在不做特殊要求的前提下，find指令将找到所有满足条件的文档，并返回这些文档的所有属性，因此，下面例子的查询结果展示了“若汐”文档的所有属性和属性值（包括"\_id"属性）。

```sql
[例1.59] 查询文档的所有属性
db.student.find( 
  { "gender": "female", "department": "数学"} 
)
结果为：
{
  "_id": ObjectId("4b2b9f67a1f631733d917a7a"),
  "sno": "2022191",
  "sname": "若汐",
  "gender": "female",
  "age": 18,
  "department": "数学",
  "courses":[]
}
```

但是，很多时候，我们并不需要一个文档的所有属性。find指令允许指定需要返回的属性。例如：

```sql
[例1.60] 查询文档的指定属性
db.person.find( 
  { "gender": "female", "department": "数学" }, 
  { "sname": 1, "department": 1 } 
)
结果为：
{
  "_id": ObjectId("4b2b9f67a1f631733d917a7a"),
  "sname": "若汐",
  "department": "数学"
}
```

这里，find指令包含两个参数，第一个参数指定查询条件，第二个参数指定需返回的属性。上例只要求返回查询结果文档中的name和department两个属性。因此，所得到的查询结果就被大大简化了。值得注意的是，"\_id"属性是文档的标识属性，即使不被指定，它也会返回。

MongoDB的查询指令还有很多使用细节，这里将不再赘述。在实际使用时，读者可以查阅MongoDB的[使用手册](https://docs.mongodb.com/manual/)。

### 1.13.4  文档的更新

MongoDB使用update指令实现对文档的更新。其中，updateOne用于更新单个文档，updateMany用于更新多个文档。

update指令包含三个参数，第一个参数指定查询条件，即表示将对什么文档进行更新；第二个参数指定具体的更新操作，即更新文档的哪些属性；第三个参数为可选参数。

```sql
[例1.61]更新单个文档
db.student.updateOne(
   { "sname": "沐辰" },
   {  $set: {"department": "电气" } }
)
```

updateOne指令首先找到属性sname取值为“沐辰”的文档，然后将该文档的department属性改为“电气”。updateOne指令只更新找到的第一个文档，也就是说，如果文档集student中存在两个名叫“沐辰”的人，那么只有第一个被找到的“沐辰”文档会被修改。

```sql
[例1.62]更新多个文档
db.student.updateMany(
   { "age": {$lt: 19} },
   {  $set: { "courses": [] } }
)
```

updateMany指令首先找到在属性age上取值小于19的文档（lt代表“小于”（less than）），即年龄小于19岁的学生，然后将这些学生的课程属性courses改为空数组。updateMany指令允许同时更新多个文档，因此凡是年龄小于19岁的学生，无论多少，都会被更新。有时，满足查询条件的文档可能没有courses属性。遇到这种情况，MongoDB会自动地为这些文档添加courses属性，然后设置courses为空数组。

### 1.13.5  文档的删除

MongoDB使用delete指令实现从一个文档集中删除文档。具体指令分为deleteOne和deleteMany。

```sql
[例1.63] 删除所有文档
db.student.deleteMany({})
```

deleteMany指令允许同时删除满足查询条件的所有文档。上例中，deleteMany指令中没有指定查询条件，即没有指定删除对象的条件，它表示将文档集student中的所有文档全部删除。

```SQL
[例1.64] 删除满足指定条件的所有文档
db.student.deleteMany( { "name": "沐辰" } )
```

上例中，deleteMany指令首先找到属性name取值为"沐辰"的所有文档，然后将查询到的所有文档从文档集student中删除。


```sql
[例1.65] 删除一个文档
db.student.deleteOne( { "_id": ObjectId("4b2b9f67a1f631733d917a7a") } )
```

deleteOne指令只允许删除满足查询条件的第一个文档。上例中的指令表示删除某一特定"\_id"的文档。

以上简单地介绍了文档数据库MongoDB的CRUD操作。在使用不同的文档数据库系统时，读者需要查阅对应系统的相关文档，从而才能准确掌握CRUD指令的具体使用方法。

### 1.13.6  聚合操作

MongoDB通常使用aggregate指令实现对文档集进行分组、过滤、排序、计算统计等多种复杂操作，并将结果以新的形式返回。常用的聚合操作有\$match、\$project、\$group、\$sort、\$limit、\$unwind等。

| MongoDB聚合操作 |                 描述                 |
| --------------- | :----------------------------------: |
| $match          |   筛选满足条件的文档，类似find指令   |
| $project        |    指定返回的字段和对字段进行转换    |
| $group          |     将文档按照指定的字段进行分组     |
| $sort           |            对文档进行排序            |
| $limit          |      限制聚合操作返回的文档数量      |
| \$unwind        | 将数组类型的字段展开为多个单独的文档 |

接下来以学生-课程数据库为例介绍文档数据库的聚合操作。学生-课程数据库中包含学生文档集（student），该文档集中的数据实例如下：

```SQL
学生文档集(student)中包含如下4个文档，每个文档包含学号、姓名、性别、年龄、系以及学生选课信息五个属性，其中学生选课信息以文档数组的形式
 {"sno": "2022001","sname": "沐辰","gender": "male","age": 19,"department": "计算机", 
  "courses":[
     {"cno":"1","cname":"高数","credit":4,"grade":92},
     {"cno":"3","cname":"数据库","credit":4,"grade": 85}
  ]
 }/*文档1*/
{"sno": "2022123","sname": "浩宇","gender": "male","age": 18 ,"department": "计算机", "courses":[]}/*文档2*/
{"sno": "2022191","sname": "若汐","gender": "female","age": 18 ,"department": "数学",
 "courses":[
     {"cno": "2","cname":"C语言","credit":3,"grade": 88},
     {"cno": "3","cname":"数据库","credit":4,"grade": 90}
 ]
}/*文档3*/
{"sno": "2022267","sname": "依诺","gender": "female","age": 19 ,"department": "金融", "courses":[]}/*文档4*/
```

```SQL
[例1.66] 查询每个系中女生的人数并按降序排序
db.student.aggregate( [ 
						{
						    $match:{"gender":"female"}
						},  /*$match操作*/
						{
						    $group:{
						        "_id":"$department",   /*按department属性进行分组*/
						        "totalnum":{"$sum":1 /*统计每个分组中的文档数量*/
						     }
						},  /*$group操作*/
    					{
    						$sort:{"totalnum":-1}  /*按totalnum的降序排序*/
    					}   /*sort操作*/
					] )
```

例1.66是在学生文档集student 上依次进行\$match（筛选）、\$group（分组）和\$sort（排序）三个聚合操作。首先\$match操作从student文档集中筛选出gender属性为"female"的学生文档，然后\$group操作对筛选结果按department属性进行分组，并使用\$sum操作符计算每个分组中女生的人数，最后\$sort操作按照每个分组统计的个数降序排序。聚合查询的结果如下：

```SQL
{"_id": "数学","totalnum":1},
{"_id": "金融","totalnum":1},
{"_id": "计算机","totalnum":0},
```

注意：\$group操作中的“\_id”字段是必填，用于指定分组属性，如果“\_id”值为null则表示对输入文档不进行分组，整个输入文档为一个组。\$group操作中的其他字段则是对分组之后的结果进行计算，比如求每个分组的平均值“\$avg”，返回每个分组的数值总和“$sum”，返回每个分组的最大值\$max等。\$sort操作是对指定的字段进行排序，-1表示升序排序，1表示降序排序。

```SQL
[例1.67] 查询学生的选课课程的情况，输出前3条学生选课情况（包含学生学号、学生姓名、所选课程名以及成绩）
db.student.aggregate( [ 
    					{
    						$unwind:"$courses"  /*将course属性展开成多个单独的文档*/
    					}, /*$unwind操作*/
						{
						    $project:{
    							"sno":1,
						        "sname":1,  
						        "cname":"$courses.cname",
    							"grade":"$courses.grade"
						     }
						},  /*$project操作*/
    					{
    						$limit:3   /*返回查询结果的前3个文档*/
    					}   /*$limit操作*/
					] )
```

例1.67在学生文档集student 上依次进行\$unwind（数组展开）、\$project（投影）和\$limit（限制）三个聚合操作。 首先\$unwind操作将courses数组展开，使得每一条学生选课记录成为一个独立的文档，例如，沐辰选修了两门课程，展开后会变成两个独立文档，每个文档分别对应沐辰选修某一门课程的信息。然后\$project操作指定返回数组展开结果中的学生学号、学生姓名、所选课程名及成绩字段，其中所选课程名及成绩分别来自于开展后的courses的cname和grade，最后\$limit操作限制最终返回的结果是前3条学生选课文档。聚合查询的结果如下：

```SQL
{
	"sno": "2022001",
	"sname": "沐辰",
	"cname": "高数",
	"grade": 92
}，
{
	"sno": "2022001",
	"sname": "沐辰",
	"cname": "数据库",
	"grade": 85
}，
{
	"sno": "2022191",
	"sname": "若汐",
	"cname": "C语言",
	"grade": 88
}

```

注意，查询结果没有显示“浩宇”同学的选课情况，这是因为“浩宇”同学的选课信息courses为空，\$unwind操作符会直接跳过没有选课信息的文档，因此数组展开后的结果中不包含“浩宇”同学的信息。同理，\$unwind操作符执行之后的结果中也不包含“依诺”同学的信息。

```SQL
[例1.68] 查询每个学生的平均成绩，输出学生学号和平均成绩
db.student.aggregate( [ 
    					{
    						$unwind:"$courses"  /*将course属性展开成多个单独的文档*/
    					}, /*$unwind操作*/
    					{
						    $group:{
						        "_id":"$sno",   /*按sno属性进行分组*/
						        "averageGrade":{"$avg":"$courses.grade" /*计算每个分组中所有课程成绩的平均值*/
						     }
						},  /*$group操作*/
						{
						    $project:{
    							"sno":"$_id",
    							"averageGrade":1
						     }
						}  /*$project操作*/
					] )
```

例1.68中，首先\$unwind操作将courses数组展开，使得每一条学生选课记录成为一个独立的文档，然后\$group操作对开展后的结果按sno属性进行分组，并使用$avg操作符计算每个组内所有课程成绩的平均值，将结果放在averageGrade字段中，最后\$project操作指定返回的字段包括学号和平均成绩，学号sno来自于分组的\_id，平均成绩来自于averageGrade字段。聚合查询结果如下：
```SQL
{
	"sno": "2022001",
	"averageGrade": 88.5
}，
{
	"sno": "2022191",
	"averageGrade": 89
}
```

以上简单地介绍了MongoDB文档数据库的增删改查操作以及聚合操作。更加详细的内容，读者需要阅读对应系统的相关文档。

### 练习题

**1**. 文档数据库允许一个属性有多个取值，比如{...colors: ["red", "blank"] ...} 或者 {...hobbies: ["red", "blank"] ...}。那么，哪些文档满足以下查询db.inventory.find( { tags: ["computer", "music"] } )？

 <ol type="A">
 <li>仅{... tags: ["computer", "music"]... }。</li>
 <li>{... tags: ["computer"]... }和{... tags: ["music"]... }</li>
 <li>{... tags: ["computer", "music"]... }和{... tags: ["computer", "music", "movie"]... }</li>
 <li>{... tags: ["computer"]... }和{... tags: ["music"]... }和{... tags: ["computer", "music"]... }和{... tags: ["computer", "music", "movie"]... }</li>
 </ol>
**2.** 假设有一用户文档集users存储用户的评论数据，其文档结构如下：

```sql
{
  "user_id": "U1001",
  "comments": [
    { "text": "Great product!", "rating": 5 },
    { "text": "Average service", "rating": 3 }
  ]
}
```

   请写出以下信息需求的查询语句：统计所有用户的平均评分，输出用户名和平均评分。

**3.** 假设有一日志文档集logs存储应用访问日志，其文档结构如下：

```sql
{
  "timestamp": ISODate("2023-10-05T08:30:00Z"),
  "user": "U1001",
  "action": "login",
  "duration": 2 // 单位：秒
}
```

   请写出以下信息需求的查询语句：按小时统计每天上午（8:00-12:00）的 action 为 "login" 的总次数和平均耗时（四舍五入到整数）。

**4**. 关系数据库要求用户在使用一张表之前用DDL对表进行事先定义，并且给出表需满足的各种约束（比如，主键，外键等）。文档数据库则不同，它通常不需求用户对文档集的结构做事先定义，甚至允许用户在文档集中插入任意结构的文档。请思考：关系数据库和文档数据库为什么使用两种不同的功能设计？背后的原因是什么？

[**上一页<<**](chapter1.12-D.md) | [**>>下一页**](chapter1.14-G.md)
