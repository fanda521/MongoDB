# MongoDB命令操作手册

## 数据库操作

### 创建数据库

```mysql
语法：use 数据库名字
脚下留心：刚创建的数据库，使用查看命令，结果中不会列出来，需要插入数据后才能显示出来。
> show databases;
admin   0.000GB
config  0.000GB
local   0.000GB
> use test1
switched to db test1
> db
test1
> db.test.insert({"name":"jeffrey","age":25})
WriteResult({ "nInserted" : 1 })
> show databases;
admin   0.000GB
config  0.000GB
local   0.000GB
test1   0.000GB
```

### 查看数据库

```mysql
语法：show databases
> show databases;
admin   0.000GB
config  0.000GB
local   0.000GB
test1   0.000GB
```

### 删除数据库

```mysql
语法：db.dropDatabase()
> show databases;
admin   0.000GB
config  0.000GB
local   0.000GB
test1   0.000GB

> db.dropDatabase()
{ "dropped" : "test1", "ok" : 1 }
>

> show databases;
admin   0.000GB
config  0.000GB
local   0.000GB
>

```

## 集合操作

### 创建集合

```mysql
语法格式：

db.createCollection(name, options)
参数说明：

name: 要创建的集合名称
options: 可选参数, 指定有关内存大小及索引的选项
options 可以是如下参数：

字段	类型	描述
capped	布尔	（可选）如果为 true，则创建固定集合。固定集合是指有着固定大小的集合，当达到最大值时，它会自动覆盖最早的文档。
当该值为 true 时，必须指定 size 参数。
autoIndexId	布尔	3.2 之后不再支持该参数。（可选）如为 true，自动在 _id 字段创建索引。默认为 false。
size	数值	（可选）为固定集合指定一个最大值，即字节数。
如果 capped 为 true，也需要指定该字段。
max	数值	（可选）指定固定集合中包含文档的最大数量。
在插入文档时，MongoDB 首先检查固定集合的 size 字段，然后检查 max 字段。

> db.createCollection("mycol",{capped : true,autoIndexId : true,size : 102400,max:1000})
{
        "note" : "The autoIndexId option is deprecated and will be removed in a future release",
        "ok" : 1
}
>

```

### 查看集合

```mysql
语法：show collections

> show collections
mycol
>
```



### 删除集合

```mysql
语法：db.集合名.drop()

> db.mycol.drop()
true
> show collections
>
```

## 文档操作

### 插入文档

```mysql
MongoDB 使用 insert() 或 save() 方法向集合中插入文档，语法如下：

db.COLLECTION_NAME.insert(document)
或
db.COLLECTION_NAME.save(document)
save()：如果 _id 主键存在则更新数据，如果不存在就插入数据。该方法新版本中已废弃，可以使用 db.collection.insertOne() 或 db.collection.replaceOne() 来代替。
insert(): 若插入的数据主键已经存在，则会抛 org.springframework.dao.DuplicateKeyException 异常，提示主键重复，不保存当前数据。
3.2 版本之后新增了 db.collection.insertOne() 和 db.collection.insertMany()。

db.collection.insertOne() 用于向集合插入一个新文档，语法格式如下：

db.collection.insertOne(
   <document>,
   {
      writeConcern: <document>
   }
)
db.collection.insertMany() 用于向集合插入一个多个文档，语法格式如下：

db.collection.insertMany(
   [ <document 1> , <document 2>, ... ],
   {
      writeConcern: <document>,
      ordered: <boolean>
   }
)
参数说明：

document：要写入的文档。
writeConcern：写入策略，默认为 1，即要求确认写操作，0 是不要求。
ordered：指定是否按顺序写入，默认 true，按顺序写入。


> db.test1.insert([{"name":"jeffrey","age":25},{"name":"mark","age":22}])
BulkWriteResult({
        "writeErrors" : [ ],
        "writeConcernErrors" : [ ],
        "nInserted" : 2,
        "nUpserted" : 0,
        "nMatched" : 0,
        "nModified" : 0,
        "nRemoved" : 0,
        "upserted" : [ ]
})
> db.test1.find()
{ "_id" : ObjectId("6097683e9f03b1598dd46cc3"), "name" : "jeffrey", "age" : 25 }
{ "_id" : ObjectId("6097683e9f03b1598dd46cc4"), "name" : "mark", "age" : 22 }
>


> db.test1.insertOne({"name":"jeffrey","age":25})
{
        "acknowledged" : true,
        "insertedId" : ObjectId("6097687b9f03b1598dd46cc5")
}
> db.test1.find()
{ "_id" : ObjectId("6097683e9f03b1598dd46cc3"), "name" : "jeffrey", "age" : 25 }
{ "_id" : ObjectId("6097683e9f03b1598dd46cc4"), "name" : "mark", "age" : 22 }
{ "_id" : ObjectId("6097687b9f03b1598dd46cc5"), "name" : "jeffrey", "age" : 25 }
>

> db.test1.insertMany([{"name":"jeffrey","age":15},{"name":"mark1","age":22}])
{
        "acknowledged" : true,
        "insertedIds" : [
                ObjectId("609768ce9f03b1598dd46cc6"),
                ObjectId("609768ce9f03b1598dd46cc7")
        ]
}
> db.test1.find()
{ "_id" : ObjectId("6097683e9f03b1598dd46cc3"), "name" : "jeffrey", "age" : 25 }
{ "_id" : ObjectId("6097683e9f03b1598dd46cc4"), "name" : "mark", "age" : 22 }
{ "_id" : ObjectId("6097687b9f03b1598dd46cc5"), "name" : "jeffrey", "age" : 25 }
{ "_id" : ObjectId("609768ce9f03b1598dd46cc6"), "name" : "jeffrey", "age" : 15 }
{ "_id" : ObjectId("609768ce9f03b1598dd46cc7"), "name" : "mark1", "age" : 22 }
>


```

### 更新文档

```mysql
update() 方法用于更新已存在的文档。语法格式如下：

db.collection.update(
   <query>,
   <update>,
   {
     upsert: <boolean>,
     multi: <boolean>,
     writeConcern: <document>
   }
)
参数说明：

query : update的查询条件，类似sql update查询内where后面的。
update : update的对象和一些更新的操作符（如$,$inc...）等，也可以理解为sql update查询内set后面的
upsert : 可选，这个参数的意思是，如果不存在update的记录，是否插入objNew,true为插入，默认是false，不插入。
multi : 可选，mongodb 默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。
writeConcern :可选，抛出异常的级别。

没有使用$set的会覆盖
> db.test1.find()
{ "_id" : ObjectId("6097683e9f03b1598dd46cc3"), "name" : "jeffrey", "age" : 25 }
{ "_id" : ObjectId("6097683e9f03b1598dd46cc4"), "name" : "mark", "age" : 22 }
{ "_id" : ObjectId("6097687b9f03b1598dd46cc5"), "name" : "jeffrey", "age" : 25 }
{ "_id" : ObjectId("609768ce9f03b1598dd46cc6"), "name" : "jeffrey", "age" : 15 }
{ "_id" : ObjectId("609768ce9f03b1598dd46cc7"), "name" : "mark1", "age" : 22 }
> db.test1.update({"name":"mark1"},{"name":"mark2"})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.test1.find()
{ "_id" : ObjectId("6097683e9f03b1598dd46cc3"), "name" : "jeffrey", "age" : 25 }
{ "_id" : ObjectId("6097683e9f03b1598dd46cc4"), "name" : "mark", "age" : 22 }
{ "_id" : ObjectId("6097687b9f03b1598dd46cc5"), "name" : "jeffrey", "age" : 25 }
{ "_id" : ObjectId("609768ce9f03b1598dd46cc6"), "name" : "jeffrey", "age" : 15 }
{ "_id" : ObjectId("609768ce9f03b1598dd46cc7"), "name" : "mark2" }
>

使用$set
> db.test1.update({"name":"mark"},{$set:{"name":"mark2"}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.test1.find()
{ "_id" : ObjectId("6097683e9f03b1598dd46cc3"), "name" : "jeffrey", "age" : 25 }
{ "_id" : ObjectId("6097683e9f03b1598dd46cc4"), "name" : "mark2", "age" : 22 }
{ "_id" : ObjectId("6097687b9f03b1598dd46cc5"), "name" : "jeffrey", "age" : 25 }
{ "_id" : ObjectId("609768ce9f03b1598dd46cc6"), "name" : "jeffrey", "age" : 15 }
{ "_id" : ObjectId("609768ce9f03b1598dd46cc7"), "name" : "mark2" }
>


```

### 删除文档

```mysql
db.collection.remove(
   <query>,
   {
     justOne: <boolean>,
     writeConcern: <document>
   }
)
参数说明：

query :（可选）删除的文档的条件。
justOne : （可选）如果设为 true 或 1，则只删除一个文档，如果不设置该参数，或使用默认值 false，则删除所有匹配条件的文档。
writeConcern :（可选）抛出异常的级别。

> db.test1.find()
{ "_id" : ObjectId("6097683e9f03b1598dd46cc3"), "name" : "jeffrey", "age" : 25 }
{ "_id" : ObjectId("6097683e9f03b1598dd46cc4"), "name" : "mark2", "age" : 22 }
{ "_id" : ObjectId("6097687b9f03b1598dd46cc5"), "name" : "jeffrey", "age" : 25 }
{ "_id" : ObjectId("609768ce9f03b1598dd46cc6"), "name" : "jeffrey", "age" : 15 }
{ "_id" : ObjectId("609768ce9f03b1598dd46cc7"), "name" : "mark2" }
> db.test1.remove({"name":"mark2"},{justOne:true})
WriteResult({ "nRemoved" : 1 })
> db.test1.find()
{ "_id" : ObjectId("6097683e9f03b1598dd46cc3"), "name" : "jeffrey", "age" : 25 }
{ "_id" : ObjectId("6097687b9f03b1598dd46cc5"), "name" : "jeffrey", "age" : 25 }
{ "_id" : ObjectId("609768ce9f03b1598dd46cc6"), "name" : "jeffrey", "age" : 15 }
{ "_id" : ObjectId("609768ce9f03b1598dd46cc7"), "name" : "mark2" }
>
```

### 查询文档

```mysql
MongoDB 查询数据的语法格式如下：

db.collection.find(query, projection)
query ：可选，使用查询操作符指定查询条件
projection ：可选，使用投影操作符指定返回的键。查询时返回文档中所有键值， 只需省略该参数即可（默认省略）。
如果你需要以易读的方式来读取数据，可以使用 pretty() 方法，语法格式如下：

>db.col.find().pretty()
pretty() 方法以格式化的方式来显示所有文档。

MongoDB 与 RDBMS Where 语句比较
如果你熟悉常规的 SQL 数据，通过下表可以更好的理解 MongoDB 的条件语句查询：

操作	格式	范例	RDBMS中的类似语句
等于	{<key>:<value>}	db.col.find({"by":"菜鸟教程"}).pretty()	where by = '菜鸟教程'
小于	{<key>:{$lt:<value>}}	db.col.find({"likes":{$lt:50}}).pretty()	where likes < 50
小于或等于	{<key>:{$lte:<value>}}	db.col.find({"likes":{$lte:50}}).pretty()	where likes <= 50
大于	{<key>:{$gt:<value>}}	db.col.find({"likes":{$gt:50}}).pretty()	where likes > 50
大于或等于	{<key>:{$gte:<value>}}	db.col.find({"likes":{$gte:50}}).pretty()	where likes >= 50
不等于	{<key>:{$ne:<value>}}	db.col.find({"likes":{$ne:50}}).pretty()	where likes != 50

> db.test1.find({"name":"jeffrey"},{"age":1}).pretty()
{ "_id" : ObjectId("6097683e9f03b1598dd46cc3"), "age" : 25 }
{ "_id" : ObjectId("6097687b9f03b1598dd46cc5"), "age" : 25 }
{ "_id" : ObjectId("609768ce9f03b1598dd46cc6"), "age" : 15 }
> db.test1.find({"name":"jeffrey"},{"age":0}).pretty()
{ "_id" : ObjectId("6097683e9f03b1598dd46cc3"), "name" : "jeffrey" }
{ "_id" : ObjectId("6097687b9f03b1598dd46cc5"), "name" : "jeffrey" }
{ "_id" : ObjectId("609768ce9f03b1598dd46cc6"), "name" : "jeffrey" }
>

and 多个条件
> db.test1.find({"name":"jeffrey","age":25},{"age":1}).pretty()
{ "_id" : ObjectId("6097683e9f03b1598dd46cc3"), "age" : 25 }
{ "_id" : ObjectId("6097687b9f03b1598dd46cc5"), "age" : 25 }
>
小于，小于等于
> db.test1.find({"age":{$lt:25}},{"age":1}).pretty()
{ "_id" : ObjectId("609768ce9f03b1598dd46cc6"), "age" : 15 }
> db.test1.find({"age":{$lte:25}},{"age":1}).pretty()
{ "_id" : ObjectId("6097683e9f03b1598dd46cc3"), "age" : 25 }
{ "_id" : ObjectId("6097687b9f03b1598dd46cc5"), "age" : 25 }
{ "_id" : ObjectId("609768ce9f03b1598dd46cc6"), "age" : 15 }
>

or
> db.test1.find({$or:[{"name":"mark2"},{"age":{$lt:25}}]}).pretty()
{
        "_id" : ObjectId("609768ce9f03b1598dd46cc6"),
        "name" : "jeffrey",
        "age" : 15
}
{ "_id" : ObjectId("609768ce9f03b1598dd46cc7"), "name" : "mark2" }
>

or &  and

> db.test1.find({"name":"jeffrey",$or:[{"name":"mark2"},{"age":{$lt:25}}]}).pretty()
{
        "_id" : ObjectId("609768ce9f03b1598dd46cc6"),
        "name" : "jeffrey",
        "age" : 15
}
>



```

### 模糊查询

```mysql
模糊查询

查询 title 包含"教"字的文档：

db.col.find({title:/教/})
查询 title 字段以"教"字开头的文档：

db.col.find({title:/^教/})
查询 titl e字段以"教"字结尾的文档：

db.col.find({title:/教$/})

> db.test1.find({"name":/je/}).pretty()
{
        "_id" : ObjectId("6097683e9f03b1598dd46cc3"),
        "name" : "jeffrey",
        "age" : 25
}
{
        "_id" : ObjectId("6097687b9f03b1598dd46cc5"),
        "name" : "jeffrey",
        "age" : 25
}
{
        "_id" : ObjectId("609768ce9f03b1598dd46cc6"),
        "name" : "jeffrey",
        "age" : 15
}
>

> db.test1.find({"name":/rk2$/}).pretty()
{ "_id" : ObjectId("609768ce9f03b1598dd46cc7"), "name" : "mark2" }
>
```

## $type

### 简单实用

```mysql
MongoDB 中可以使用的类型如下表所示：

类型	数字	备注
Double	1	 
String	2	 
Object	3	 
Array	4	 
Binary data	5	 
Undefined	6	已废弃。
Object id	7	 
Boolean	8	 
Date	9	 
Null	10	 
Regular Expression	11	 
JavaScript	13	 
Symbol	14	 
JavaScript (with scope)	15	 
32-bit integer	16	 
Timestamp	17	 
64-bit integer	18	 
Min key	255	Query with -1.
Max key	127	 


> db.test1.find({"name":{$type:2}})
{ "_id" : ObjectId("6097683e9f03b1598dd46cc3"), "name" : "jeffrey", "age" : 25 }
{ "_id" : ObjectId("6097687b9f03b1598dd46cc5"), "name" : "jeffrey", "age" : 25 }
{ "_id" : ObjectId("609768ce9f03b1598dd46cc6"), "name" : "jeffrey", "age" : 15 }
{ "_id" : ObjectId("609768ce9f03b1598dd46cc7"), "name" : "mark2" }
>
```

## Limit与Skip

### limit

```mysql
> db.test1.find().limit(2)
{ "_id" : ObjectId("6097683e9f03b1598dd46cc3"), "name" : "jeffrey", "age" : 25 }
{ "_id" : ObjectId("6097687b9f03b1598dd46cc5"), "name" : "jeffrey", "age" : 25 }
>
```

### Skip

```mysql
> db.test1.find().skip(1)
{ "_id" : ObjectId("6097687b9f03b1598dd46cc5"), "name" : "jeffrey", "age" : 25 }
{ "_id" : ObjectId("609768ce9f03b1598dd46cc6"), "name" : "jeffrey", "age" : 15 }
{ "_id" : ObjectId("609768ce9f03b1598dd46cc7"), "name" : "mark2" }
>
```

### limit & skip

```mysql
> db.test1.find().skip(1).limit(2)
{ "_id" : ObjectId("6097687b9f03b1598dd46cc5"), "name" : "jeffrey", "age" : 25 }
{ "_id" : ObjectId("609768ce9f03b1598dd46cc6"), "name" : "jeffrey", "age" : 15 }
>
```

## 排序

### 简单使用

```mysql
sort()方法基本语法如下所示：

>db.COLLECTION_NAME.find().sort({KEY:1})
1:升序 -1：降序
> db.test1.find().sort({"age":1,"name":-1})
{ "_id" : ObjectId("609768ce9f03b1598dd46cc7"), "name" : "mark2" }
{ "_id" : ObjectId("609768ce9f03b1598dd46cc6"), "name" : "jeffrey", "age" : 15 }
{ "_id" : ObjectId("6097683e9f03b1598dd46cc3"), "name" : "jeffrey", "age" : 25 }
{ "_id" : ObjectId("6097687b9f03b1598dd46cc5"), "name" : "jeffrey", "age" : 25 }
>


```

## 索引

### 创建索引

```
createIndex()方法基本语法格式如下所示：

>db.collection.createIndex(keys, options)

```

| createIndex() 接收可选参数，可选参数列表如下： |               |                                                              |
| ---------------------------------------------- | ------------- | ------------------------------------------------------------ |
| Parameter                                      | Type          | Description                                                  |
| background                                     | Boolean       | 建索引过程会阻塞其它数据库操作，background可指定以后台方式创建索引，即增加 "background" 可选参数。 "background" 默认值为**false**。 |
| unique                                         | Boolean       | 建立的索引是否唯一。指定为true创建唯一索引。默认值为**false**. |
| name                                           | string        | 索引的名称。如果未指定，MongoDB的通过连接索引的字段名和排序顺序生成一个索引名称。 |
| dropDups                                       | Boolean       | **3.0+版本已废弃。**在建立唯一索引时是否删除重复记录,指定 true 创建唯一索引。默认值为 **false**. |
| sparse                                         | Boolean       | 对文档中不存在的字段数据不启用索引；这个参数需要特别注意，如果设置为true的话，在索引字段中不会查询出不包含对应字段的文档.。默认值为 **false**. |
| expireAfterSeconds                             | integer       | 指定一个以秒为单位的数值，完成 TTL设定，设定集合的生存时间。 |
| v                                              | index version | 索引的版本号。默认的索引版本取决于mongod创建索引时运行的版本。 |
| weights                                        | document      | 索引权重值，数值在 1 到 99,999 之间，表示该索引相对于其他索引字段的得分权重。 |
| default_language                               | string        | 对于文本索引，该参数决定了停用词及词干和词器的规则的列表。 默认为英语 |
| language_override                              | string        | 对于文本索引，该参数指定了包含在文档中的字段名，语言覆盖默认的language，默认值为 language. |

### 查看索引

```mysql
个数
> db.test1.totalIndexSize()
36864
>
 查看数据库中所有索引db.system.indexes.find()
 
 查看集合中的索引getIndexes()
 > db.test1.getIndexes()
[ { "v" : 2, "key" : { "_id" : 1 }, "name" : "_id_" } ]
>

> db.test2.getIndexes()
[ { "v" : 2, "key" : { "_id" : 1 }, "name" : "_id_" } ]
> db.test2.createIndex({name:1})
{
        "createdCollectionAutomatically" : false,
        "numIndexesBefore" : 1,
        "numIndexesAfter" : 2,
        "ok" : 1
}
> db.test2.getIndexes()
[
        {
                "v" : 2,
                "key" : {
                        "_id" : 1
                },
                "name" : "_id_"
        },
        {
                "v" : 2,
                "key" : {
                        "name" : 1
                },
                "name" : "name_1"
        }
]
>
```

### 删除索引

```mysql
> db.test2.createIndex({age:1},{unique:1})
{
        "createdCollectionAutomatically" : false,
        "numIndexesBefore" : 2,
        "numIndexesAfter" : 3,
        "ok" : 1
}
> db.test2.getIndexes()
[
        {
                "v" : 2,
                "key" : {
                        "_id" : 1
                },
                "name" : "_id_"
        },
        {
                "v" : 2,
                "key" : {
                        "name" : 1
                },
                "name" : "name_1"
        },
        {
                "v" : 2,
                "unique" : true,
                "key" : {
                        "age" : 1
                },
                "name" : "age_1"
        }
]
>
删除指定的索引dropIndex()
> db.test2.dropIndex("age_1")
{ "nIndexesWas" : 3, "ok" : 1 }
> db.test2.getIndexes()
[
        {
                "v" : 2,
                "key" : {
                        "_id" : 1
                },
                "name" : "_id_"
        },
        {
                "v" : 2,
                "key" : {
                        "name" : 1
                },
                "name" : "name_1"
        }
]
>
删除所有索引dropIndexes()
> db.test2.dropIndexes()
{
        "nIndexesWas" : 3,
        "msg" : "non-_id indexes dropped for collection",
        "ok" : 1
}
> db.test2.getIndexes()
[ { "v" : 2, "key" : { "_id" : 1 }, "name" : "_id_" } ]
>
注意：不会删除系统自动创建的索引
```



