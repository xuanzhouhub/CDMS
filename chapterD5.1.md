# 文档数据库设计

前面的章节介绍了数据库设计的基本步骤以及如何根据需求分析进行概念模型设计。概念设计解决了一个特定应用的数据库应该存什么数据的问题，而数据如何存的问题由结构设计解决。结构设计与具体数据库管理系统的数据模型紧密相关。本章将主要介绍文档数据库的结构设计。

## 设计规则

文档数据库的逻辑结构设计其实是将概念设计阶段设计好的基于ERD的概念模型转换成文档模型，它的要点是如何将ERD的实体、实体的属性和实体之间的联系转换成文档模式。所谓的文档模式是对文档集的描述。

<center>
	<img src="fig/ch5.1-E-Rforemployee.jpg" width="85%" alt="E-R" />
	<br>
	<div display: inline-block; padding : 2px>
		图 D5.1 员工与项目ERD概念模型
	</div>
</center>

文档数据库支持嵌套文档、数组以及文档数组等多种结构，因此基于ERD概念模型生成的文档模型也是多种多样的。目前，没有一套文档数据库结构设计的统一范式。我们这里只介绍ERD转换成文档模型的常用规则。

* 实体转换规则：
  * 若文档模型中不存在嵌套文档，则一个实体转换为一个文档集。例如，图D5.1中的员工实体转换为员工文档集（employee），项目实体转换为项目文档集（project）；
  * 若文档模型中存在嵌套文档，则多个实体转换为一个文档集。例如，图D5.1中的员工实体和项目实体可以转换为一个文档集（emplyee-project）。

* 属性转换规则：
  * 实体的唯一属性、单值属性、多值属性转换成文档模式的属性，其中多值属性以数组的形式表示。假设文档模型中不存在嵌套文档，图D5.1中员工实体和项目实体的文档结构如下：

    ```SQL
    员工employee的文档结构
    {
      "sno": " ",   //工号
      "name": " ", 	//名字
      "skill": [" "," ", ""]  //技能
    }
    
    项目project的文档结构
    {
      "pno": " ",   //项目号
      "funds": " " 	//经费
    }
    ```
  
* 联系转换规则：
  * 若文档模型中不存在嵌套文档，则实体之间的1:1联系可以与任意一端的文档模式合并，合并端的文档模式中加入另一端实体的唯一性属性和联系本身的属性；实体之间的1:n联系与n端的文档模式合并，在n端的文档模式中加入1端实体的唯一性属性和联系本身的属性；实体之间的m:n联系转换为一个独立的文档模式，联系两端实体型的唯一性属性和联系本身的属性构成该文档模式的属性。例如，图D5.1中的“参与”联系是m:n联系，该联系转换成一个单独的文档集（work），work文档集中的文档结构如下：
      ```SQL
    work的文档结构
    {
      "sno": " ",   //员工工号
      "pno": " ", 	//项目号
      "working-hours": " " //工时
    }
    ```
  * 若文档模型中存在嵌套文档，嵌套文档的形式能自然地存储实体之间的一对一、一对多、多对多联系。如果实体之间是1:1联系，那么任意一端的实体嵌入另一端文档模式；如果实体之间是1:n联系，那么n端的实体嵌入1端的文档模式，以文档数组的形式表示；如果实体之间是m:n联系，那么任意一端的实体嵌入另一端文档模式，以文档数组的形式表示。例如，图D5.1中的员工和项目之间“参与”联系，将项目嵌入员工的文档结构如下：
      ```SQL
    employee-project的文档结构
    {
      "sno": " ",   //工号
      "name": " ", 	//名字
      "skill": [" "," ", ""]  //技能
      "project":[
        {
          "pno": " ",   //项目号
      	  "funds": " " ,	//经费
      	  "working-hours": " " //工时
        },
        ...
      ]
    }
    ```

## 文档结构设计

基于以上的规则，我们能够进行文档数据库的逻辑结构设计，确定某一特定应用数据库中的数据应该如何表示和组织。以下以本书前面章节提到的购物网站为例，展示文档数据库的逻辑结构设计过程。图D5.2展示了购物网站的ERD概念模型。

<center>
	<img src="fig/chD5.1-E-Rshopping.jpg" width="85%" alt="E-R" />
	<br>
	<div display: inline-block; padding : 2px>
		图 D5.2 购物网站ERD概念模型
	</div>
</center>


如果按“文档模型中不存在嵌套文档”的规则，可以得到以下的文档模式设计一：

```SQL
购物网站的文档模式设计一：
用户文档集：User{Uid, Uname, Uadd, Tel, Pref[]};
商品文档集：Product{Pid, Pname, Category, Price, Padd};
订单文档集：Order{Oid, Uid, Date}；
订单详情文档集：OrderLine{Oid, Pid, Quantity}；
```
* 首先，图D5.2中的用户、商品、订单三个实体分别转换成User、Product、Order三个文档集，实体的唯一性属性、单值属性和多值属性构成文档模式的属性。因此，生成用户文档集User{Uid, Uname, Uadd, Tel, Pref[]}，其中多值属性“爱好”用数组形式表示，商品文档集Product{Pid, Pname, Category, Price, Padd}，订单文档集Order{Oid, Date}；

* 然后，用户和订单之间的一对多联系通过将用户的用户号加入订单文档集中实现转换，实现联系转换的订单文档集模式为Order{Oid, Uid, Date}；

* 最后，商品和订单之间的多对多联系转换成订单详细文档集OrderLine，订单的唯一属性、商品的唯一属性以及联系的自身属性构成该文档集的属性，因此，订单详细文档集模式为OrderLine{Oid, Pid, Quantity}。

值得注意的是，除了文档模式设计中的属性之外，文档数据库会为每一个文档自动分配一个唯一标识的“\_id”属性。

如果按“文档模型中存在嵌套文档”的规则，可以得到以下三种文档模式设计：

```SQL
购物网站的文档模式设计二：
用户文档集：User{Uid, Uname, Uadd, Tel, Pref[]};
商品文档集：Product{Pid, Pname, Category, Price, Padd};
订单文档集：Order{Oid, Uid, Date,OrderLine[{Pid, Quantity}]}；

购物网站的文档模式设计三：
用户文档集：User{Uid, Uname, Uadd, Tel, Pref[], 
				Order[{
				  Oid, Date,OrderLine[{Pid,Quantity}]
				}]
			};
商品文档集：Product{Pid, Pname, Category, Price, Padd};

购物网站的文档模式设计四：
用户文档集：User{Uid, Uname, Uadd, Tel, Pref[], 
				Order[{
				  Oid, Date,
				  OrderLine[{
					  Product{Pid, Pname, Category, Price, Padd},
					  Quantity}]
				}]
			};
```

文档模式设计二在设计一的基础上，以嵌套文档的结构表示订单和商品之间的多对多联系。首先，将订单和商品之间的m:n联系转换成订单详细文档集OrderLine，订单详细文档集只包含商品的唯一属性和联系的自身属性，其模式为OrderLine{Pid,Quantity}，然后将OrderLine以文档数组的方式嵌入订单文档集中，因此订单文档集的模式为Order{Oid, Uid, Date,OrderLine[{Pid, Quantity}]}。

文档模式设计三在设计二的基础上，将订单文档集Order以文档数组的形式嵌入用户文档集。

文档模式设计四在设计三的基础上，将商品文档集Product以文档数组的形式嵌入用户文档集中的OrderLine。

那么，哪一种文档模式更合理呢？这需要根据购物网站的业务流程和应用功能进行判断。如果某种文档组织方式使得实现应用功能更加简单，性能更高，那么该文档组织方式则更优。

通过上述例子，我们发现文档数据库的结构设计相当灵活的，没有一套固定的标准，需要根据应用的实际需求选择最适合的文档模式。

## 合理利用数据冗余

为了能够进一步提高应用的性能，文档结构设计有时需要加入必要的冗余数据 。加入的冗余数据不会消耗过多的存储空间，不会产生额外的更新代价，且经常被查询，有助于提升查询效率。

例如，在购物网站中，商家需要经常统计某个商品的总销售额。基于购物网站的文档模式设计二，MongoDB数据库的查询语句如下所示：

```SQL
基于购物网站的文档模式设计二，统计某个商品的总销售，其查询语句如下：
db.Order.aggregate( [ 
        {
            $match:{"OrderLine.Pid":" "} /*查找包含某商品号的所有订单*/
        },  
        {
            $unwind:{"$OrderLine"} /*将OrderLine数组展开为单独的文档*/
        },  
        {
            $match:{"OrderLine.Pid":" "} /*再次过滤，确保只处理某商品*/
        },
        {
            $lookup:{
                "from":"Product", 
                "localFiled":"OrderLine.Pid", 
                "foreignField":"Pid", 
                "as"："ProductInfo"  
             } /*连接商品文档集Product，获取商品价格*/
        },  
        {
            $unwind:{"$ProductInfo"} /*展开ProductInfo数组，确保每个订单行对应一个商品信息*/
        },
        {
            $group:{
                "_id":"$OrderLine.Pid", /*按商品号分组*/
                "totalSales":{$sum:{$multiply:["$OrderLine.Quantity","$ProductInfo.Price"]}} /*计算商品的总销售额*/
             }
        }
] )
```

上述查询需要使用文档数据库的聚合计算，需要经过\$match，\$unwind，\$lookup，\$group等多个阶段的处理才能得到最终的结果。尤其在$lookup阶段，需要将订单文档集Order和商品文档集Product进行左外连接操作，执行该连接操作需要对商品文档集进行扫描，从而使得查询效率很低。

为了提高查询效率，我们可以在商品文档集Product中冗余存储商品的总销售额TotalSales属性。优化之后的文档模式设计二及其查询语句如下：

```SQL
优化之后的购物网站的文档模式设计二
用户文档集：User{Uid, Uname, Uadd, Tel, Pref[]};
商品文档集：Product{Pid, Pname, Category, Price, Padd, TotalSales};
订单文档集：Order{Oid, Uid, Date,OrderLine[{Pid, Quantity}]}；

统计某个商品的总销售额，其查询如下：
db.Prodect.find(
	{"Pid":" "},
    {"TotalSales":1}
)
每次产生新订单时，需要更新购买商品的总销售：
db.Product.updateOne(
    {"Pid":" "},
    {$inc: {"TotalSales":{$multiply:["$Price", "购买数量"]}}}
)
```

在商品文档集中冗余存储商品的总销售额TotalSales，能够避免文档数据库的聚合查询以及连接操作，从而使得查询语句的执行效率非常高。而冗余存储带来的代价是每次购买商品时需要同步更新该商品的总销售额。如果更新操作对应用性能的影响比较小，那么数据冗余时可接受的。

[**上一页<<**](chapterD2.1.md) | [**>>下一页**](chapterD3.2.md)



