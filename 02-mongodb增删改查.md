# 02-mongodb增删改查

既然是数据库，就离不开crud

create(创建)， read(读取)， update(更新)和 delete(删除)

MongoDB 不支持 SQL 但是支持自己的丰富的查询语言。

在 MongoDB 中，存储在集合中的每个文档都需要一个唯一的 *id 字段，作为主键。*

_如果插入的文档省略了该_id 字段，则 MongoDB 驱动程序将自动为该字段生成一个 ObjectId_id。

如果文档包含一个_id 字段，该_id 值在集合中必须是唯一的，以避免重复键错误。

在 MongoDB 中，插入操作针对单个集合。

MongoDB 中的所有写操作都是在单个文档的级别上进行的。

# 1.mongodb基础命令

## 1.默认数据库

```
0.数据库介绍
test:  登陆的时默认的库
admin: 系统预留库，Mongodb的系统管理库
local: 本地预留库，存储关键日志
config: 配置信息库，保存如分片的信息
```

## 2.数据库管理

```
> show dbs;   查看所有数据库
admin   0.000GB
config  0.000GB
local   0.000GB
yuc     0.000GB
> 

> show databases;
admin   0.000GB
config  0.000GB
local   0.000GB
yuc     0.000GB
> 


> 
> db  # 查看当前在哪个库
test
> 



# 切换数据库
> 
> use yuc
switched to db yuc
> 
> 
> db
yuc
> 


# 查看库下的集合，等于show tables

> use admin;
switched to db admin
> show collections;
system.version
> 


# 删除数据库，当初当前数据库。
> use yuc;
switched to db yuc
> 
> db.dropDatabase()
{ "dropped" : "yuc", "ok" : 1 }
> 
> 
> show dbs;
admin   0.000GB
config  0.000GB
local   0.000GB
> 


# 创建数据库
使用use 直接切换且创建数据库。

> use yuchao666;
switched to db yuchao666

> db
yuchao666



> show dbs;
admin   0.000GB
config  0.000GB
local   0.000GB
> 



在 MongoDB 中，集合只有在内容插入后才会创建! 就是说，创建集合(数据表)后要再插入一个文档(记录)，集合才会真正创建。

> db.chaoge.insert({"name":"www.yuchaoit.cn"})
WriteResult({ "nInserted" : 1 })
> 

> show dbs;
admin      0.000GB
config     0.000GB
local      0.000GB
yuchao666  0.000GB
> 

# 查看创建的库下的集合
> show collections
chaoge


查看表内数据
> db.chaoge.find()
{ "_id" : ObjectId("634a7b8ff18533aa0c0c29d6"), "name" : "www.yuchaoit.cn" }
>
```

## 3.mongodb注意点

```
1.mongo默认登录在test库
2.mongo不需要提前创建库、表、直接use切换就是创建库，直接插入数据自动创建表。
3.使用use切换库时，如果没有任何数据，实际上不会创建库，就是虚拟库，show dbs是看不到的
```

## 4.shell窗口执行mongo

```
非交互式操作
[root chaoge-linux ~]#echo 'show dbs' | mongo
MongoDB shell version v4.2.22
connecting to: mongodb://127.0.0.1:27017/?compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("c899ca20-55dc-459b-a378-410904a6ec69") }
MongoDB server version: 4.2.22
admin   0.000GB
config  0.000GB
local   0.000GB
yuc     0.000GB
bye
[root chaoge-linux ~]#
```

## 5.查看mongo命令帮助

```
help: 显示帮助。
db.help() 显示数据库方法的帮助。
db.<collection>.help() 显示收集方法的帮助， <collection>可以是现有的集合或不存在的集合的名称。
show dbs 打印服务器上所有数据库的列表。
use <db> 将当前数据库切换到<db>。该 mongoshell 变量 db 被设置为当前数据库。
show collections 打印当前数据库的所有集合的列表。
show users 打印当前数据库的用户列表。
show roles 打印用于当前数据库的用户定义和内置的所有角色的列表。
show profile 打印需要 1 毫秒或更多的五个最近的操作。有关详细信息，请参阅数据库分析器上的文档。
show databases 打印所有可用数据库的列表。
```

# 2.mongo文档增删改查

```
官网
https://www.mongodb.com/docs/manual/crud/

学数据库，路线就是
库管理
表管理
数据管理
```

## 2.0 集合管理

```
集合，类似于关系型数据库中的表，可以显示的创建，也可以隐式的创建，也就是如刚才超哥直接insert写入数据，集合自动也就创建了。


显示创建集合
# 当capped为true后，需要指定集合大小的值
# 6142800字节，6M、最大允许写入10000条文档
# 在插入文档时，MongoDB 首先检查固定集合的 size 字段，然后检查 max 字段。

db.createCollection("chaoge_linux",{capped:true,size:6142800,max:10000})

> show collections
chaoge_linux

# 查看集合信息

> db.chaoge_linux.stats()

# 插入文档
> db.chaoge_linux.insert({"mysite":"www.yuchaoit.cn"})
WriteResult({ "nInserted" : 1 })
> 
> db.chaoge_linux.find()
{ "_id" : ObjectId("634a8198d2a83b89e17c732d"), "mysite" : "www.yuchaoit.cn" }
> 


# 删除集合
# 如果成功删除选定集合，则 drop() 方法返回 true，否则返回 false。


> db.chaoge_linux.drop()
true
> 
> db.chaoge_linux.drop()
false
>
```

## 2.1 插入文档

mongodb提供的

库、集合、文档

三要素，如何写入文档数据，必然是核心知识。

本章节中我们将向大家介绍如何将数据插入到 MongoDB 的集合中。

文档的数据结构和 JSON 基本一样。

所有存储在集合中的数据都是 BSON 格式。

BSON 是一种类似 JSON 的二进制形式的存储格式，是 Binary JSON 的简称。

```
https://www.mongodb.com/docs/manual/tutorial/insert-documents/

MongoDB提供了将文档插入到集合中的以下方法： 
db.collection.insertOne() 将单个文档插入到集合中。 
db.collection.insertMany() 将多个 文档插入到集合中。 
db.collection.insert() 将单个文档或多个文档插入到集合中。 
db.collection.save() 根据文档参数更新现有文档或插入新文档　　

https://www.mongodb.com/docs/manual/reference/insert-methods/
```

### 插入单条数据

key加不加引号都可以

```
db.user_info.insertOne({"name":"yuchao","website":"www.yuchaoit.cn"})
db.user_info.insertOne({"name":"sanpang","address":"北京"})
db.user_info.insertOne({name:"goudan",age:"28"})


美化json
db.user_info.insertOne(
{
    "name":"于超老师",
    "age":28,
    "website":"www.yuchaoit.cn"
}
)

# 查询
> db.user_info.find()
{ "_id" : ObjectId("634a8d38d2a83b89e17c732e"), "name" : "yuchao", "website" : "www.yuchaoit.cn" }
{ "_id" : ObjectId("634a8d38d2a83b89e17c732f"), "name" : "sanpang", "address" : "北京" }
{ "_id" : ObjectId("634a8d3ad2a83b89e17c7330"), "name" : "goudan", "age" : "28" }
{ "_id" : ObjectId("634a8d6fd2a83b89e17c7331"), "name" : "于超老师", "age" : 28, "website" : "www.yuchaoit.cn" }
>
```

### 插入多条语句

```
db.user_info.insertMany(
[
    {name:"yuchao",age:"28",website:"www.yuchaoit.cn"},
  {name:"sanpang",age:"29"},
  {name:"ergou",addr:"北京"},
]
)
```

## 2.2 查询文档

MongoDB 查询文档使用 find() 方法。

find() 方法以非结构化的方式来显示所有文档。

```
db.collection.find()
```

如果你需要以易读的方式来读取数据，可以使用 pretty() 方法，语法格式如下：

```
>db.col.find().pretty()
```

### 简单查询

```
# 1. 查询单条SQL
select * from user_info limit 1;

> db.user_info.findOne()
{
    "_id" : ObjectId("634a8d38d2a83b89e17c732e"),
    "name" : "yuchao",
    "website" : "www.yuchaoit.cn"
}
> 

# 2.查询多条
select * from user_info

> db.user_info.find()
{ "_id" : ObjectId("634a8d38d2a83b89e17c732e"), "name" : "yuchao", "website" : "www.yuchaoit.cn" }
{ "_id" : ObjectId("634a8d38d2a83b89e17c732f"), "name" : "sanpang", "address" : "北京" }
{ "_id" : ObjectId("634a8d3ad2a83b89e17c7330"), "name" : "goudan", "age" : "28" }
{ "_id" : ObjectId("634a8d6fd2a83b89e17c7331"), "name" : "于超老师", "age" : 28, "website" : "www.yuchaoit.cn" }
{ "_id" : ObjectId("634a8e38d2a83b89e17c7332"), "name" : "goudan", "age" : "28" }
{ "_id" : ObjectId("634b7cfed2a83b89e17c7333"), "name" : "yuchao", "age" : "28", "website" : "www.yuchaoit.cn" }
{ "_id" : ObjectId("634b7cfed2a83b89e17c7334"), "name" : "sanpang", "age" : "29" }
{ "_id" : ObjectId("634b7cfed2a83b89e17c7335"), "name" : "ergou", "addr" : "北京" }
> 



# 3.按条件查询
例如
select * from user_info where name='yuchao';

> db.user_info.find({age:"28"})
{ "_id" : ObjectId("634a8d3ad2a83b89e17c7330"), "name" : "goudan", "age" : "28" }
{ "_id" : ObjectId("634a8e38d2a83b89e17c7332"), "name" : "goudan", "age" : "28" }
{ "_id" : ObjectId("634b7cfed2a83b89e17c7333"), "name" : "yuchao", "age" : "28", "website" : "www.yuchaoit.cn" }
> 
> db.user_info.find({name:"yuchao"})
{ "_id" : ObjectId("634a8d38d2a83b89e17c732e"), "name" : "yuchao", "website" : "www.yuchaoit.cn" }
{ "_id" : ObjectId("634b7cfed2a83b89e17c7333"), "name" : "yuchao", "age" : "28", "website" : "www.yuchaoit.cn" }
> 


# 4. 查询出文档的部分字段
如
select name,age from user_info where name='yuchao';

# 去掉主键显示 ，以及单独显示某字段
> db.user_info.find({name:"yuchao"},{name:1,_id:0})
{ "name" : "yuchao" }
{ "name" : "yuchao" }

# 单独去掉某个字段，就显示剩余字段
# 语法 find({条件1,条件2},{字段:1或0})



> db.user_info.find({name:"yuchao"},{_id:0})
{ "name" : "yuchao", "website" : "www.yuchaoit.cn" }
{ "name" : "yuchao", "age" : "28", "website" : "www.yuchaoit.cn" }
> 
> 

> db.user_info.find({name:"yuchao"},{age:0})
{ "_id" : ObjectId("634a8d38d2a83b89e17c732e"), "name" : "yuchao", "website" : "www.yuchaoit.cn" }
{ "_id" : ObjectId("634b7cfed2a83b89e17c7333"), "name" : "yuchao", "website" : "www.yuchaoit.cn" }
> 
> db.user_info.find({name:"yuchao"},{website:0})
{ "_id" : ObjectId("634a8d38d2a83b89e17c732e"), "name" : "yuchao" }
{ "_id" : ObjectId("634b7cfed2a83b89e17c7333"), "name" : "yuchao", "age" : "28" }
>
```

### 嵌套查询

```
官网文档
https://www.mongodb.com/docs/manual/tutorial/query-embedded-documents/#match-an-embedded-nested-document
```

测试数据

```
db.inventory.insertMany([
   { item: "journal", qty: 25, size: { h: 14, w: 21, uom: "cm" }, status: "A" },
   { item: "notebook", qty: 50, size: { h: 8.5, w: 11, uom: "in" }, status: "A" },
   { item: "paper", qty: 100, size: { h: 8.5, w: 11, uom: "in" }, status: "D" },
   { item: "planner", qty: 75, size: { h: 22.85, w: 30, uom: "cm" }, status: "D" },
   { item: "postcard", qty: 45, size: { h: 10, w: 15.25, uom: "cm" }, status: "A" }
])
```

嵌套查询

```
> db.inventory.find({"size.h":22.85})
{ "_id" : ObjectId("634b8f91d2a83b89e17c7339"), "item" : "planner", "qty" : 75, "size" : { "h" : 22.85, "w" : 30, "uom" : "cm" }, "status" : "D" }
> 

> 
> db.inventory.find({"size.h":22.85},{"status":1,item:1})
{ "_id" : ObjectId("634b8f91d2a83b89e17c7339"), "item" : "planner", "status" : "D" }
> 
> db.inventory.find({"size.h":22.85},{"status":1,item:1,_id:0})
{ "item" : "planner", "status" : "D" }
>
```

### 逻辑查询

https://www.mongodb.com/docs/v4.4/reference/operator/query-comparison/

官网资料

> 比较运算符

- $eq 匹配等于指定值的值。
- $gt 匹配大于指定值的值。
- $gte 匹配大于或等于指定值的值。
- $in 匹配数组中指定的任何值。
- $lt 匹配小于指定值的值。
- $lte 匹配小于或等于指定值的值。
- $ne 匹配不等于指定值的所有值。
- $nin不匹配数组中指定的值。

> 逻辑查询运算符

- $and 使用逻辑连接查询子句AND返回与两个子句的条件相匹配的所有文档。
- $not 反转查询表达式的效果，并返回与查询表达式不匹配的文档。
- $nor 使用逻辑连接查询子句NOR返回所有无法匹配两个子句的文档。
- $or 使用逻辑连接查询子句OR返回与任一子句的条件相匹配的所有文档

```
# and语句
select * from inventory where status="A" and qty< 30;

db.inventory.find(
{"status":"A","qty":{$lt:30}}
)

# and 案例2
# select * from inventory where status='A' AND size.uom='cm';

db.inventory.find(
{"status":"A","size.uom":{$eq:"cm"}}
)

或者
db.inventory.find(
{"status":"A","size.uom":"cm"}
)

# or 逻辑查询
select * from inventory where status=A or qty<30;

> db.inventory.find(
... {
...   $or:[
...     {qty:{$lt:30}},
...     {"status":"A"}
...   ]
... }
... )
{ "_id" : ObjectId("634b8f91d2a83b89e17c7336"), "item" : "journal", "qty" : 25, "size" : { "h" : 14, "w" : 21, "uom" : "cm" }, "status" : "A" }
{ "_id" : ObjectId("634b8f91d2a83b89e17c7337"), "item" : "notebook", "qty" : 50, "size" : { "h" : 8.5, "w" : 11, "uom" : "in" }, "status" : "A" }
{ "_id" : ObjectId("634b8f91d2a83b89e17c733a"), "item" : "postcard", "qty" : 45, "size" : { "h" : 10, "w" : 15.25, "uom" : "cm" }, "status" : "A" }
> 


# or 逻辑运算2
# 查询status是A或D的文档

> db.inventory.find(
... 
... {
... status: {$in:["A","D"]}
... }
... )
{ "_id" : ObjectId("634b8f91d2a83b89e17c7336"), "item" : "journal", "qty" : 25, "size" : { "h" : 14, "w" : 21, "uom" : "cm" }, "status" : "A" }
{ "_id" : ObjectId("634b8f91d2a83b89e17c7337"), "item" : "notebook", "qty" : 50, "size" : { "h" : 8.5, "w" : 11, "uom" : "in" }, "status" : "A" }
{ "_id" : ObjectId("634b8f91d2a83b89e17c7338"), "item" : "paper", "qty" : 100, "size" : { "h" : 8.5, "w" : 11, "uom" : "in" }, "status" : "D" }
{ "_id" : ObjectId("634b8f91d2a83b89e17c7339"), "item" : "planner", "qty" : 75, "size" : { "h" : 22.85, "w" : 30, "uom" : "cm" }, "status" : "D" }
{ "_id" : ObjectId("634b8f91d2a83b89e17c733a"), "item" : "postcard", "qty" : 45, "size" : { "h" : 10, "w" : 15.25, "uom" : "cm" }, "status" : "A" }
> 
> 

# in 逻辑运算
> db.inventory.find( { qty: { $in: [ 10, 25 ] } } )
{ "_id" : ObjectId("634b8f91d2a83b89e17c7336"), "item" : "journal", "qty" : 25, "size" : { "h" : 14, "w" : 21, "uom" : "cm" }, "status" : "A" }
> 
> 




# 逻辑查询 结合正则表达式
# 这些语句，都是为了查数据么，找规则提取数据
# 提取qty小于30，或者是item以p开头

> db.inventory.find( {
... status: "A",
... $or: [ { qty: { $lt: 30 } }, { item: /^p/ } ]
... } )
{ "_id" : ObjectId("634b8f91d2a83b89e17c7336"), "item" : "journal", "qty" : 25, "size" : { "h" : 14, "w" : 21, "uom" : "cm" }, "status" : "A" }
{ "_id" : ObjectId("634b8f91d2a83b89e17c733a"), "item" : "postcard", "qty" : 45, "size" : { "h" : 10, "w" : 15.25, "uom" : "cm" }, "status" : "A" }
> 
> 

# 结果再限制字段
> db.inventory.find( {  status: "A",  $or: [  { qty: { $lt: 30 } },  { item: /^p/ }   ]  },{     _id:0, status:1, qty:1, item:1 } )
{ "item" : "journal", "qty" : 25, "status" : "A" }
{ "item" : "postcard", "qty" : 45, "status" : "A" }
```

## 2.3 更新文档

https://www.mongodb.com/docs/v4.4/tutorial/update-documents/

MongoDB提供以下方法来更新集合中的文档：

```
https://www.mongodb.com/docs/v4.4/reference/update-methods/#update-methods


db.collection.updateOne(<filter>, <update>, <options>)即使可能有多个文档通过过滤条件匹配到，但是也最多也只更新一个文档

db.collection.updateMany(<filter>, <update>, <options>) 更新所有通过过滤条件匹配到的文档

db.collection.replaceOne(<filter>, <replacement>, <options>) 即使可能有多个文档通过过滤条件匹配到，但是也最多也只替换一个文档

db.collection.update()即使可能有多个文档通过过滤条件匹配到，但是也最多也只更新或者替换一个文档。
```

语法

```
db.inventory.updateOne({查询条件},{更改内容}）
```

测试数据

```
db.inventory.insertMany( [
   { item: "canvas", qty: 100, size: { h: 28, w: 35.5, uom: "cm" }, status: "A" },
   { item: "journal", qty: 25, size: { h: 14, w: 21, uom: "cm" }, status: "A" },
   { item: "mat", qty: 85, size: { h: 27.9, w: 35.5, uom: "cm" }, status: "A" },
   { item: "mousepad", qty: 25, size: { h: 19, w: 22.85, uom: "cm" }, status: "P" },
   { item: "notebook", qty: 50, size: { h: 8.5, w: 11, uom: "in" }, status: "P" },
   { item: "paper", qty: 100, size: { h: 8.5, w: 11, uom: "in" }, status: "D" },
   { item: "planner", qty: 75, size: { h: 22.85, w: 30, uom: "cm" }, status: "D" },
   { item: "postcard", qty: 45, size: { h: 10, w: 15.25, uom: "cm" }, status: "A" },
   { item: "sketchbook", qty: 80, size: { h: 14, w: 21, uom: "cm" }, status: "A" },
   { item: "sketch pad", qty: 95, size: { h: 22.85, w: 30.5, uom: "cm" }, status: "A" }
] );
```

### 更新单条文档

https://www.mongodb.com/docs/manual/tutorial/update-documents/#update-a-single-document

案例

```
db.collection.updateOne(
   <query>,
   { $set: { status: "D" }, $inc: { quantity: 2 } },
   ...
)
> db.inventory.find({"item":"paper"})
{ "_id" : ObjectId("634b8f91d2a83b89e17c7338"), "item" : "paper", "qty" : 100, "size" : { "h" : 8.5, "w" : 11, "uom" : "in" }, "status" : "D" }
{ "_id" : ObjectId("634bead2d2a83b89e17c7340"), "item" : "paper", "qty" : 100, "size" : { "h" : 8.5, "w" : 11, "uom" : "in" }, "status" : "D" }
> 



修改paper文档记录，只修改第一个，且添加了一个时间字段


> db.inventory.updateOne(
... {"item":"paper"},
... {
... $set:{"size.uom":"cm",status:"P"},
... $currentDate:{lastModified:true}
... }
... )
{ "acknowledged" : true, "matchedCount" : 1, "modifiedCount" : 1 }
> 
> db.inventory.find({"item":"paper"})
{ "_id" : ObjectId("634b8f91d2a83b89e17c7338"), "item" : "paper", "qty" : 100, "size" : { "h" : 8.5, "w" : 11, "uom" : "cm" }, "status" : "P", "lastModified" : ISODate("2022-10-16T11:32:16.994Z") }
{ "_id" : ObjectId("634bead2d2a83b89e17c7340"), "item" : "paper", "qty" : 100, "size" : { "h" : 8.5, "w" : 11, "uom" : "in" }, "status" : "D" }
>
```

### 更新多条文档

```
> db.inventory.find({"item":/^p/},{_id:0,item:1,status:1})
{ "item" : "paper", "status" : "P" }
{ "item" : "planner", "status" : "D" }
{ "item" : "postcard", "status" : "A" }
{ "item" : "paper", "status" : "D" }
{ "item" : "planner", "status" : "D" }
{ "item" : "postcard", "status" : "A" }
> 


db.inventory.updateMany(
   { "item": /^p/ },
   {
     $set: { status: "Z" }
   }
)

# 再次查询结果
> db.inventory.find({"item":/^p/},{_id:0,item:1,status:1})
{ "item" : "paper", "status" : "Z" }
{ "item" : "planner", "status" : "Z" }
{ "item" : "postcard", "status" : "Z" }
{ "item" : "paper", "status" : "Z" }
{ "item" : "planner", "status" : "Z" }
{ "item" : "postcard", "status" : "Z" }
> 


# 也可以批量添加更新事件字段
> db.inventory.find({"qty":{"$lt":50}})
{ "_id" : ObjectId("634b8f91d2a83b89e17c7336"), "item" : "journal", "qty" : 25, "size" : { "h" : 14, "w" : 21, "uom" : "cm" }, "status" : "A" }
{ "_id" : ObjectId("634b8f91d2a83b89e17c733a"), "item" : "postcard", "qty" : 45, "size" : { "h" : 10, "w" : 15.25, "uom" : "cm" }, "status" : "Z" }
{ "_id" : ObjectId("634bead2d2a83b89e17c733c"), "item" : "journal", "qty" : 25, "size" : { "h" : 14, "w" : 21, "uom" : "cm" }, "status" : "A" }
{ "_id" : ObjectId("634bead2d2a83b89e17c733e"), "item" : "mousepad", "qty" : 25, "size" : { "h" : 19, "w" : 22.85, "uom" : "cm" }, "status" : "P" }
{ "_id" : ObjectId("634bead2d2a83b89e17c7342"), "item" : "postcard", "qty" : 45, "size" : { "h" : 10, "w" : 15.25, "uom" : "cm" }, "status" : "Z" }
> 

# 批量更新
db.inventory.updateMany(
{"qty":{$lt:50}},
{$set:{"size.uom":"xxx","status":"P"},$currentDate:{"latestModified":true}}
)

> db.inventory.find({"qty":{"$lt":50}})
{ "_id" : ObjectId("634b8f91d2a83b89e17c7336"), "item" : "journal", "qty" : 25, "size" : { "h" : 14, "w" : 21, "uom" : "xxx" }, "status" : "P", "latestModified" : ISODate("2022-10-16T11:43:50.741Z") }
{ "_id" : ObjectId("634b8f91d2a83b89e17c733a"), "item" : "postcard", "qty" : 45, "size" : { "h" : 10, "w" : 15.25, "uom" : "xxx" }, "status" : "P", "latestModified" : ISODate("2022-10-16T11:43:50.741Z") }
{ "_id" : ObjectId("634bead2d2a83b89e17c733c"), "item" : "journal", "qty" : 25, "size" : { "h" : 14, "w" : 21, "uom" : "xxx" }, "status" : "P", "latestModified" : ISODate("2022-10-16T11:43:50.741Z") }
{ "_id" : ObjectId("634bead2d2a83b89e17c733e"), "item" : "mousepad", "qty" : 25, "size" : { "h" : 19, "w" : 22.85, "uom" : "xxx" }, "status" : "P", "latestModified" : ISODate("2022-10-16T11:43:50.741Z") }
{ "_id" : ObjectId("634bead2d2a83b89e17c7342"), "item" : "postcard", "qty" : 45, "size" : { "h" : 10, "w" : 15.25, "uom" : "xxx" }, "status" : "P", "latestModified" : ISODate("2022-10-16T11:43:50.741Z") }
>
```

### 增加字段

```
> db.inventory.find({"item":/^p/})
{ "_id" : ObjectId("634b8f91d2a83b89e17c7338"), "item" : "paper", "qty" : 100, "size" : { "h" : 8.5, "w" : 11, "uom" : "cm" }, "status" : "Z", "lastModified" : ISODate("2022-10-16T11:32:16.994Z") }
{ "_id" : ObjectId("634b8f91d2a83b89e17c7339"), "item" : "planner", "qty" : 75, "size" : { "h" : 22.85, "w" : 30, "uom" : "cm" }, "status" : "Z" }
{ "_id" : ObjectId("634b8f91d2a83b89e17c733a"), "item" : "postcard", "qty" : 45, "size" : { "h" : 10, "w" : 15.25, "uom" : "xxx" }, "status" : "P", "latestModified" : ISODate("2022-10-16T11:43:50.741Z") }
{ "_id" : ObjectId("634bead2d2a83b89e17c7340"), "item" : "paper", "qty" : 100, "size" : { "h" : 8.5, "w" : 11, "uom" : "in" }, "status" : "Z" }
{ "_id" : ObjectId("634bead2d2a83b89e17c7341"), "item" : "planner", "qty" : 75, "size" : { "h" : 22.85, "w" : 30, "uom" : "cm" }, "status" : "Z" }
{ "_id" : ObjectId("634bead2d2a83b89e17c7342"), "item" : "postcard", "qty" : 45, "size" : { "h" : 10, "w" : 15.25, "uom" : "xxx" }, "status" : "P", "latestModified" : ISODate("2022-10-16T11:43:50.741Z") }
> 


# 批量修改
> db.inventory.updateMany(
... {item:/^p/},
... {$set:{website:"www.yuchaoit.cn"}}
... )
{ "acknowledged" : true, "matchedCount" : 6, "modifiedCount" : 6 }
> 
> db.inventory.find({"item":/^p/})
{ "_id" : ObjectId("634b8f91d2a83b89e17c7338"), "item" : "paper", "qty" : 100, "size" : { "h" : 8.5, "w" : 11, "uom" : "cm" }, "status" : "Z", "lastModified" : ISODate("2022-10-16T11:32:16.994Z"), "website" : "www.yuchaoit.cn" }
{ "_id" : ObjectId("634b8f91d2a83b89e17c7339"), "item" : "planner", "qty" : 75, "size" : { "h" : 22.85, "w" : 30, "uom" : "cm" }, "status" : "Z", "website" : "www.yuchaoit.cn" }
{ "_id" : ObjectId("634b8f91d2a83b89e17c733a"), "item" : "postcard", "qty" : 45, "size" : { "h" : 10, "w" : 15.25, "uom" : "xxx" }, "status" : "P", "latestModified" : ISODate("2022-10-16T11:43:50.741Z"), "website" : "www.yuchaoit.cn" }
{ "_id" : ObjectId("634bead2d2a83b89e17c7340"), "item" : "paper", "qty" : 100, "size" : { "h" : 8.5, "w" : 11, "uom" : "in" }, "status" : "Z", "website" : "www.yuchaoit.cn" }
{ "_id" : ObjectId("634bead2d2a83b89e17c7341"), "item" : "planner", "qty" : 75, "size" : { "h" : 22.85, "w" : 30, "uom" : "cm" }, "status" : "Z", "website" : "www.yuchaoit.cn" }
{ "_id" : ObjectId("634bead2d2a83b89e17c7342"), "item" : "postcard", "qty" : 45, "size" : { "h" : 10, "w" : 15.25, "uom" : "xxx" }, "status" : "P", "latestModified" : ISODate("2022-10-16T11:43:50.741Z"), "website" : "www.yuchaoit.cn" }
>
```

## 2.4 删除数据

MongoDB提供以下方法来删除集合的文档：

https://www.mongodb.com/docs/manual/tutorial/remove-documents/

```
db.collection.remove() 删除单个文档或与指定过滤器匹配的所有文档。 参数{}，清空集合
db.collection.drop（）此方法获取受影响的数据库上的写入锁定，并将阻止其他操作，直到
其完成。 不加参数删除所有数据包括索引
db.collection.deleteOne() 最多删除与指定过滤器匹配的单个文档，即使多个文档可能与指定的过滤器匹配。
db.collection.deleteMany() 删除与指定过滤器匹配的所有文档。


测试数据
db.inventory.insertMany( [
   { item: "journal", qty: 25, size: { h: 14, w: 21, uom: "cm" }, status: "A" },
   { item: "notebook", qty: 50, size: { h: 8.5, w: 11, uom: "in" }, status: "P" },
   { item: "paper", qty: 100, size: { h: 8.5, w: 11, uom: "in" }, status: "D" },
   { item: "planner", qty: 75, size: { h: 22.85, w: 30, uom: "cm" }, status: "D" },
   { item: "postcard", qty: 45, size: { h: 10, w: 15.25, uom: "cm" }, status: "A" },
] );

> db
test
> show collections
inventory
user_info
> 


# 删除单条数据
> db.inventory.find({"status":"D"})
{ "_id" : ObjectId("634bef9dd2a83b89e17c7347"), "item" : "paper", "qty" : 100, "size" : { "h" : 8.5, "w" : 11, "uom" : "in" }, "status" : "D" }
{ "_id" : ObjectId("634bef9dd2a83b89e17c7348"), "item" : "planner", "qty" : 75, "size" : { "h" : 22.85, "w" : 30, "uom" : "cm" }, "status" : "D" }
> 
> 
> db.inventory.deleteOne({"status":"D"})
{ "acknowledged" : true, "deletedCount" : 1 }
> 
> db.inventory.find({"status":"D"})
{ "_id" : ObjectId("634bef9dd2a83b89e17c7348"), "item" : "planner", "qty" : 75, "size" : { "h" : 22.85, "w" : 30, "uom" : "cm" }, "status" : "D" }
> 


# 删除多条数据（删除status是A或D、或P的）
db.inventory.deleteMany(
{
$or:[
{"status":"A"},{"status":"D"},{"status":"P"},
]
}
)

# 删除集合
> db.user_info.drop()
true
> show collections
inventory
> 

# 删除整个库，删除当前库
db.dropDatabase()
```

# 3.mongodb索引

https://www.mongodb.com/docs/manual/indexes/

索引支持MongoDB中查询的有效执行。

如果没有索引，MongoDB必须执行集合扫描，即扫描集合中的每个文档，以选择那些符合查询语句的文档。

> 如果一个查询存在适当的索引，MongoDB可以使用该索引来限制它必须检查的文档数量。

索引是特殊的数据结构[1]，它以一种易于遍历的形式存储了集合数据集的一小部分。索引存储特定字段或字段集的值，按字段的值排序。索引项的排序支持高效的平等匹配和基于范围的查询操作。此外，MongoDB可以通过使用索引中的排序来返回排序的结果。

```
默认的_id索引

默认的_id索引
MongoDB在创建一个集合时，在_id字段上创建一个唯一的索引。_id索引可以防止客户插入两个在_id字段中具有相同值的文档。你不能在_id字段上放弃这个索引。
```

## 1. 查看执行计划

创建id字段

```
db.user_info.insertMany([
{"id":1,"name":"yuchao",website:"www.yuchaoit.cn"},
{"id":2,"name":"zhangsan",website:"www.yuchaoit.cn"},
{"id":3,"name":"lisi",website:"www.yuchaoit.cn"},
{"id":4,"name":"ergou",website:"www.yuchaoit.cn"}
]
)


> db.user_info.find()
{ "_id" : ObjectId("634cc4480d8a1deddc0c30c3"), "id" : 1, "name" : "yuchao", "website" : "www.yuchaoit.cn" }
{ "_id" : ObjectId("634cc4480d8a1deddc0c30c4"), "id" : 2, "name" : "zhangsan", "website" : "www.yuchaoit.cn" }
{ "_id" : ObjectId("634cc4480d8a1deddc0c30c5"), "id" : 3, "name" : "lisi", "website" : "www.yuchaoit.cn" }
{ "_id" : ObjectId("634cc4480d8a1deddc0c30c6"), "id" : 4, "name" : "ergou", "website" : "www.yuchaoit.cn" }
>
```

查看文档查询的过程

```
> db.user_info.find({"id":1}).explain()
{
    "queryPlanner" : {
        "plannerVersion" : 1,
        "namespace" : "test.user_info",
        "indexFilterSet" : false,
        "parsedQuery" : {
            "id" : {
                "$eq" : 1
            }
        },
        "queryHash" : "6DAB46EC",
        "planCacheKey" : "6DAB46EC",
        "winningPlan" : {
            "stage" : "COLLSCAN",  // 查询文档，走全表扫描
            "filter" : {
                "id" : {
                    "$eq" : 1
                }
            },
            "direction" : "forward"
        },
        "rejectedPlans" : [ ]
    },
    "serverInfo" : {
        "host" : "chaoge-linux",
        "port" : 27017,
        "version" : "4.2.22",
        "gitVersion" : "eef44cd56b1cc11e5771736fa6cb3077e0228be2"
    },
    "ok" : 1
}
>
```

## 2. 创建索引

走索引扫描

```
#索引类型
COLLSCAN  全表扫描
IXSCAN    索引扫描
```

创建索引

https://www.mongodb.com/docs/manual/reference/method/db.collection.createIndex/#mongodb-method-db.collection.createIndex

字段解释

```
#创建索引
db.user_info.createIndex(
  {
     id: 1  // 建id字段，且进行升序，排序 ，索引的类型
  },
  {
     background: true  // 创建索引，放入后台执行，不影响性能
  }
)
```

## 3.查询集合的索引

```
> 
> db.user_info.getIndexes()
[
    {
        "v" : 2,
        "key" : {
            "_id" : 1   // 默认索引，升序排序
        },
        "name" : "_id_", 
        "ns" : "test.user_info"
    },
    {
        "v" : 2,
        "key" : {
            "id" : 1  // 自建索引，升序
        },
        "name" : "id_1", 
        "ns" : "test.user_info",
        "background" : true
    }
]
>
```

## 4. 查询文档，查看执行计划

```
查询索引字段，走索引查询。
> db.user_info.find({"id":1}).explain()
{
    "queryPlanner" : {
        "plannerVersion" : 1,
        "namespace" : "test.user_info",
        "indexFilterSet" : false,
        "parsedQuery" : {
            "id" : {
                "$eq" : 1
            }
        },
        "queryHash" : "6DAB46EC",
        "planCacheKey" : "801B9D84",
        "winningPlan" : {
            "stage" : "FETCH",
            "inputStage" : {
                "stage" : "IXSCAN",  // 走索引查询了
                "keyPattern" : {
                    "id" : 1
                },
                "indexName" : "id_1", // 索引名 
                "isMultiKey" : false,
                "multiKeyPaths" : {
                    "id" : [ ]
                },
                "isUnique" : false,
                "isSparse" : false,
                "isPartial" : false,
                "indexVersion" : 2,
                "direction" : "forward",
                "indexBounds" : {
                    "id" : [
                        "[1.0, 1.0]"
                    ]
                }
            }
        },
        "rejectedPlans" : [ ]
    },
    "serverInfo" : {
        "host" : "chaoge-linux",
        "port" : 27017,
        "version" : "4.2.22",
        "gitVersion" : "eef44cd56b1cc11e5771736fa6cb3077e0228be2"
    },
    "ok" : 1
}
>
```

## 5. 删除索引

```
> db.user_info.dropIndex("id_1")
{ "nIndexesWas" : 2, "ok" : 1 }
> 

> db.user_info.getIndexes()
[
    {
        "v" : 2,
        "key" : {
            "_id" : 1
        },
        "name" : "_id_",
        "ns" : "test.user_info"
    }
]
>
```

# 4.mongodb工具组件

https://www.mongodb.com/docs/v3.4/reference/program/

```
[root chaoge-linux ~]#mongo
mongo         mongod        mongodump     mongoexport   mongofiles    mongoimport   mongoreplay   mongorestore  mongos        mongostat     mongotop
```

## mongo

mongo 是MongoDB的交互式JavaScript shell接口，为系统管理员提供强大的界面，也为开发 人员直接用数据库测试查询和操作的方式。

mongo还提供了一个功能齐全的JavaScript环境，与MongoDB一起使用。

## mongod

mongod 是 Mongodb 系统的主要守护进程，它处理数据请求，管理数据访问，并执行后台管理操作。

启动进程指定配置文件，控制数据库的行为

## mongos

mongos 对于“MongoDB Shard”，是用于处理来自应用层的查询的MongoDB分片配置的路由服务，并确定此数据在分片集群中的位置， 以完成这些操作。从应用程序的角度来看，一个 mongos实例与任何其他MongoDB实例的行为相同。

## mongostat

mongostat 实用程序可以快速概览当前正在运行的mongod 或mongos 实例的状态。

mongostat在功能上类似于UNIX / Linux文件系统实用程序vmstat，但提供有关的数据 mongod和mongos实例。

```
[root chaoge-linux ~]#mongostat 
insert query update delete getmore command dirty used flushes vsize   res qrw arw net_in net_out conn                time
    *0    *0     *0     *0       0     1|0  0.0% 0.0%       0 1.49G 78.0M 0|0 1|0   166b   38.7k    2 Oct 17 11:09:42.728
    *0    *0     *0     *0       0     0|0  0.0% 0.0%       0 1.49G 78.0M 0|0 1|0   111b   38.5k    2 Oct 17 11:09:43.728
    *0    *0     *0     *0       0     1|0  0.0% 0.0%       0 1.49G 78.0M 0|0 1|0   112b   38.6k    2 Oct 17 11:09:44.726
^C2022-10-17T11:09:45.560+0800    signal 'interrupt' received; forcefully terminating
[root chaoge-linux ~]#
```

## mongotop

mongotop提供了一种跟踪MongoDB实例读取和写入数据的时间量的方法。

mongotop 提供每个收集级别的统计信息。

默认情况下， mongotop每秒返回一次值

## mongoplog

mongooplog是一个简单的工具，可以从远程服务器的复制 oplog轮询操作，并将其应用于本地服务器。

此功能支持某些类型的实时迁移，这些迁移要求源服务器保持联机并在整个迁移过程中运行。通常，此命令将采用以下形式：

## mongoexport

mongoexport是一个实用程序，可以导出存储在MongoDB实例中的数据,生成一个JSON或 CSV

## mongoimport

mongoimport工具从由其他第三方导出工具创建或可能的扩展JSON， CSV或TSV导出导入内容

## bsondump

bsondump 转换BSON文件转换为我们可直接读的格式，包括JSON。 bsondump对于读取mongodump输出文件很有用。

`bsondump 是用于检查BSON文件的诊断工具，而不是用于数据摄取或其他应用程序使用的工具` 找个备份文件