#mongodb
##mongodb语法
mongodb查询的语法（大于，小于，大于或等于，小于或等于等等）  
https://www.cnblogs.com/Logan626/articles/4858728.html

1 ) . 大于，小于，大于或等于，小于或等于

$gt:大于
$lt:小于
$gte:大于或等于
$lte:小于或等于

例子：

db.collection.find({ "field" : { $gt: value } } ); // greater than : field > value
db.collection.find({ "field" : { $lt: value } } ); // less than : field < value
db.collection.find({ "field" : { $gte: value } } ); // greater than or equal to : field >= value
db.collection.find({ "field" : { $lte: value } } ); // less than or equal to : field <= value
如查询j大于3,小于4:

db.things.find({j : {$lt: 3}});
db.things.find({j : {$gte: 4}});
也可以合并在一条语句内:

db.collection.find({ "field" : { $gt: value1, $lt: value2 } } ); // value1 < field < value
2) 不等于 $ne

例子：

db.things.find( { x : { $ne : 3 } } );

3) in 和 not in ($in $nin)

语法：
db.collection.find( { "field" : { $in : array } } );
例子：

db.things.find({j:{$in: [2,4,6]}});
db.things.find({j:{$nin: [2,4,6]}});


4) 取模运算$mod

如下面的运算：
db.things.find( "this.a % 10 == 1")
可用$mod代替：

db.things.find( { a : { $mod : [ 10 , 1 ] } } )


5)  $all

$all和$in类似，但是他需要匹配条件内所有的值：

如有一个对象：

{ a: [ 1, 2, 3 ] }
下面这个条件是可以匹配的：

db.things.find( { a: { $all: [ 2, 3 ] } } );
但是下面这个条件就不行了：

db.things.find( { a: { $all: [ 2, 3, 4 ] } } );

6)  $size

$size是匹配数组内的元素数量的，如有一个对象：{a:["foo"]}，他只有一个元素：

下面的语句就可以匹配：db.things.find( { a : { $size: 1 } } );
官网上说不能用来匹配一个范围内的元素，如果想找$size<5之类的，他们建议创建一个字段来保存元素的数量。

You cannot use $size to find a range of sizes (for example: arrays with more than 1 element). If you need to query for a range, create an extra size field that you increment when you add elements.

7）$exists

$exists用来判断一个元素是否存在：

如：

db.things.find( { a : { $exists : true } } ); // 如果存在元素a,就返回
db.things.find( { a : { $exists : false } } ); // 如果不存在元素a，就返回

8)  $type

$type 基于 bson type来匹配一个元素的类型，像是按照类型ID来匹配，不过我没找到bson类型和id对照表。

db.things.find( { a : { $type : 2 } } ); // matches if a is a string
db.things.find( { a : { $type : 16 } } ); // matches if a is an int
9）正则表达式

mongo支持正则表达式，如：

db.customers.find( { name : /acme.*corp/i } ); // 后面的i的意思是区分大小写

10)  查询数据内的值

下面的查询是查询colors内red的记录，如果colors元素是一个数据,数据库将遍历这个数组的元素来查询。db.things.find( { colors : "red" } );

11) $elemMatch

如果对象有一个元素是数组，那么$elemMatch可以匹配内数组内的元素：
> t.find( { x : { $elemMatch : { a : 1, b : { $gt : 1 } } } } ) 
{ "_id" : ObjectId("4b5783300334000000000aa9"), 
"x" : [ { "a" : 1, "b" : 3 }, 7, { "b" : 99 }, { "a" : 11 } ]
}
$elemMatch : { a : 1, b : { $gt : 1 } } 所有的条件都要匹配上才行。

注意，上面的语句和下面是不一样的。
> t.find( { "x.a" : 1, "x.b" : { $gt : 1 } } )

$elemMatch是匹配{ "a" : 1, "b" : 3 }，而后面一句是匹配{ "b" : 99 }, { "a" : 11 } 


12)  查询嵌入对象的值

db.postings.find( { "author.name" : "joe" } );
注意用法是author.name，用一个点就行了。更详细的可以看这个链接： dot notation

举个例子：

> db.blog.save({ title : "My First Post", author: {name : "Jane", id : 1}})
如果我们要查询 authors name 是Jane的, 我们可以这样：

> db.blog.findOne({"author.name" : "Jane"})
如果不用点，那就需要用下面这句才能匹配：

db.blog.findOne({"author" : {"name" : "Jane", "id" : 1}})
下面这句：

db.blog.findOne({"author" : {"name" : "Jane"}})
是不能匹配的，因为mongodb对于子对象，他是精确匹配。

13) 元操作符 $not 取反

如：

db.customers.find( { name : { $not : /acme.*corp/i } } );db.things.find( { a : { $not : { $mod : [ 10 , 1 ] } } } ); mongodb还有很多函数可以用，如排序，统计等，请参考原文。

mongodb目前没有或(or)操作符，只能用变通的办法代替，可以参考下面的链接：

http://www.mongodb.org/display/DOCS/OR+operations+in+query+expressions

##MongoDB两阶段提交实现事务
https://blog.csdn.net/after_you/article/details/68059774


##mongodb分片片键的选择(持续更新中)
首先要了解项目的情况,检查使用情况

对集合进行分片时,要选择一个或者两个字段拆分数据,这个键叫做片键 一旦拥有对个分片,在修改片键几乎是不肯能的事情,因此选择合适的片键是非常重要的.

对集合分片之前要问自己集合问题

计划做多少分片`?拥有三个分片的集群要比1000个的更具有灵活性,随着集群变得越来越大 不应做那些需要查询所有分片的查询,因此几乎所有查询都需包含片键
分片是为了减少读写延迟么?延迟就是某个操作花费的时间.降低写延迟的方式通常是将请求发送到地理位置更近的服务器或者更强大的机器上

分片是为了增加读写吞吐量么? 吞吐量是指集群在同一时间能够处理的请求数量,增加吞吐量通常需要提高并行性,并确保请求被均衡的分布到各集群成员上

分片是为了增加系统资源么?如果是这样 可能会希望尽量保持工作集较小
根据这些问题来对不用的片键进行评估,并判断所选的片键是否适用于自己的情况,这样做能够提供所需要的目标查询么?能够按所需方式提高系统吞吐量或者减少读写延迟么?如需保持工作集的小巧,这样做可以打到要求么?
 
###升序片键
升序片键 有点类似于date字段或者objectId 是一种随着时间稳定增长的字段,自增长的主键是升序键的另一个例子.假如我们依据升序键做片键 如使用objectId的集合中的"_id"键 如果基于_id分片 那么集合会根据不同的_id范围被拆分为多个块,

假设要创建一个新的文档 会插入哪个块呢?答案是范围最接近$maxKey的快 也就是最大块. 这样每次添加的文档都会出现这个块中,这样会带来需要不量的属性 首先所有的写请求都会被路由到找这个分片中,该快是唯一一个不断增长和拆分的块,因为只有他接收写请求,随着数据的不断插入 当达到拆分阀值的时候  mongos 就会对该块进行拆分 所以该块不断拆分出小块

这种模式进程导致mongodb的数据均衡处理变得更为困难,因为所有的新快都死由同一个分片创建的,因此必须不断的将一些快移至其他分片,而不能像在一个比较均衡发布的系统中那样 只需要纠正那些比较小的不均衡就好了

###随机分发的片键
随机分发的键可以是用户名 邮件地址 udid(唯一设备标识符) md5散列值 或者是数据集中其他一些没有规律的键 随着数据的不断插入 数据的随机性一位置 新插入的数据会比较均衡的分布在不同的块中,由于写入数据是随机分发的 各个分片增长的速度应该大致相同,这就减少了需要进行迁移的次数
随机分发片键的唯一弊端在于:mongodb在随机访问超出ram大小的数据时效率不高.如果拥有足够多的ram或者不介意系统性能的话 使用随机片键在集群上分配负载时非常好的

###基于位置的片键
可以是用户的ip 经纬度 或者地址 位置片键不必也实际的物理位置字段相关 这里的位置比较抽象 数据会依据这个位置进行分组.无论如何 所有与该键值比较接近的文档都会保存在同一范围的块中,这样可以比较方便的将数据与相应的用户, 以及相关联的数据保存在一起
假如我们有一个集合文档按照IP地址进行分片 文档会依据IP地址被分成不同的块,冰水机分布在集群中
 如果希望特定范围的块出现在特定分片中  可以在分片中添加tag 然后为块指定相应的tag
sh.addShardTag("shard0000":"USPS")
sh.addShardTag("shard0001":"Apple")
sh.addShardTag("shard0002":"Apple")

然后创建下列规则 
sh.addTagRange("test.ips",{"ip":"056.000.000.000"},{"ip":"057.000.000.000"},"USPS")
sh.addTagRange("test.ips",{"ip":"017.000.000.000"},{"ip":"018.000.000.000"},"Apple")
均衡器在移动块时,会试图将这些范围的块移动到这些分片上,注意该过程不会立即生效,没有被打过标签的块仍会正常移动.

###片键策略
###散列片键
如果追求的是数据加载速度的极致,那么散列片键是最佳选择,散列片键可使其他任何键随机分发.因此打算在大量查询中使用升序键但同时又希望写入数据随机分发的话,散列片键会是非常好的选择.

弊端 是无法使用散列片键做指定目标的范围查询.
创建一个散列片键 首先要创建散列索引 db.users,ensureIndex({"username":"hashed"}) 然后对集合分片 sh.shardConllection("app.users",{"username":"hashed"})
使用散列片键存在着一定的局限性.首先不能使用unique选项.其次,与其他片键一样,不能使用数组字段.最后注意,浮点型的值会先被取整,然后才会进行散列,所以1和1.999会得到相同的散列值
 
###流水策略
如果有一些服务器比其他服务器更强大,我们会希望让这些强大的服务器处理更多负载,比如说 一个使用SSD的分片能够处理10被与其他机器的负载.幸运的是,我们有10个其他分片.可强制将所有新数据插入到SSD,然后让均衡器将旧的块移动到其他分片上 这样能够提供比转式磁盘更低的延迟
 
为了实现这个策略 需将最大范围的块分布在SSD上  为SSD指定一个标签  sh.addShardTag("shard-name","ssd")
将升序键的当前值一直到正无穷范围的块指定分布在SSD分片上,以便后续也日请求均被发到SSD分片上 sh.addTagRange("dbname.collName",{"_id":ObjectId()},{"_id":MaxKey},"ssd")
现在 所有插入的请求俊辉被路由到这个块上,这个块始终位于标签为ssd的分片上.
 
但是 除非修改标签范围,否则从升序键的当前值一直到正无穷的这个范围则被固定在了这回分片上  可以创建一个定时任务每天更新一次标签范围 如下
use config
var tag = db.tags.findOne{{"ns":"dbName.collName","max" : {"shardKey": MaxKey }}} // 获得拥有最大key的块 
tag.min.shardKey = ObjectId() //修改该块的最小键的值 为当前的objectId
db.tags.save(tag) //保存
这样前一天的块就可以被移动到其他分片上了
此策略的另一个弊端是 需要做一些修改才能进行拓展 如果写请求超出了SSD的处理能力 想要将负载均衡的分不到当前服务器和另一台服务器并不简单
 
如果没有高性能的服务器来处理插入流水 或者没有使用标签 那么就不要将升序键用作片键,否则所有请求都会被路由到同意分片上.
 
###多热点
单个mongod服务器在处理升序写请求时是最有效的,他和分片相冲突 写请求均匀分布在集群中 分片才是最高效的.这种技术会创建多个热点(最好每个分片都创建几个热点) 写请求于是会均衡的分布在集群内 而单个分片上则是以升序分布的

##MongoDB 分片键
 一. 概述

分片键确定集合文档在集群分片中的分布。分片键可以是集合文档中的单索引或者是混合索引。

MongoDB 使用分片键值的范围在集合中分区数据。每个范围定义一个分片键值不重叠并且关联一个块。

MongoDB 尝试在集群中的分片上均匀地分布块。分片键直接关系到块分布的有效性。

 重要：一旦对一个集合分片，分片键和分片值就不可改变。 如：不能给集合选择不同的分片键、不能更新分片键的值。         
 
二. 分片键规范

对集合分片，你必须使用 sh.shardCollection() 方法指定集合和分片键。
sh.shardCollection( namespace, key )
namespace 参数是 database.collection，指定目标集合完整的命名空间
关键参数由包含字段和该字段的索引遍历方向的文档组成。

三. 分片键索引
所有的分片集合必须有一个支撑 shard key 的索引。
如果集合为空，如果索引为空，则 sh.shardCollection() 在 shard key 上创建索引
如果集合不为空，你必须在使用 sh.shardCollection() 之前创建索引

如果删除分片键的最后一个有效索引，请通过仅在分片键上重新创建索引来恢复。


唯一索引
对于一个已分片的集合，只有 _id 字段索引和 shard key 上的单索引或者混合索引 是唯一的索引
你不能分片一个唯一索引在其他字段上的集合
你不能在一个已分片的集合上创建不同字段的唯一索引


通过使用shard key上的唯一索引，MongoDB可以强制shard键值的唯一性。 MongoDB对整个key组合，而不是分片密钥的单个组件实施唯一性。为了 shard 强制一致性，可以在 sh.shardCollection() 方法的 unique 参数传入 true 值：
如果集合为空，如果该集合的索引不存在则 sh.shardCollection() 在 shard key 上创建唯一索引
如果集合不为空，在使用 sh.shardCollection() 之前必须创建索引


四. 选择一个分片键
shard key 的选择影响着「分片集群均衡器」在可用分片上创建和分布 chunks。这影响着分片集群操作的整体效率和性能。

shard key 影响分片集群所使用的分片策略效率和性能

理想化的 shard key 可以让 documents 均匀地在集群中分布

分片键基数

分片键基数决定着均衡器可以创建的最大 chunk 个数。这会减少或者移除集群水平扩展的效率。


一个唯一 shard key 任何时候只能在最多一个 chunk 中出现。如果 shard key 的基数是 4，那么集群中 chunks 的个数不能超过 4，各存储一个唯一 shard key 值。这将集群中有效分片的数量限制为4 - 添加其他分片不会提供任何好处。

#【MongoDB】shard 片键选择

选择片键需要慎重，因为一旦选定就无法更改了。

常用的有两种选择：

（1）升序片键，常见的比如_id,date,来自其他数据库的自增主键等。如果使用升序片键，那么数据物理上会是连续的，有利于基于范围的查询，因为数据库一次拿进来若干的数据块，由于是连续排列的，那么后续的数据就不需要再通过io拿新的块。但是升序键有个明显的问题是，数据只会插入到最大块里。最开始只有一个块，然后随着数据的插入，块会分裂为两块，一个是范围值较小的，一个是较大的，后续的数据因为是增序的，所以只会插入到较大快内，所以，每一次的分裂，都会导致数据值较小的块没有任何写入，所有的写入只会集中到最大块，这会降低写入性能。而且，由于其他块分裂以后没有写入请求，那么这些块的空间利用率不高。

（2）随机片键，可以使用MongoDB自带的hash key机制，这样，所有的写入请求都会被均衡的转发到各个数据块。解决了上面写入效率低的问题。但是基于范围的查询效率变低。因为数据不是按照片键值物理排放，这样相邻的数据值可能位于不同的数据块，这样需要更多的IO次数拿块到内存。



有一个比较好的方法结合上述两者的优点，也就是复合片键策略。

片键由两部分构成，第一部分是一个随机值，但是值的度比较少，与分片数目大致一样或者常数倍。第二部分是升序值。这样，每一条数据会根据第一部分做分发，分发以后每一个片内部是升序的，这样查找的时候，可以按照分片来查，每一个分片内部是升序的，效率会变高。

比如第一部分是{A,B,C}, 第二部分升序，那么片键可能是A1 B2 C3 A4 B5 C6 A10 B15 C20这样。


