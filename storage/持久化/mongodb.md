
##性能
###Mongodb亿级数据量的性能测试
https://blog.csdn.net/weixin_33708432/article/details/86087114

##MongoDB数据库查询性能提高40倍
https://blog.csdn.net/pbymw8iwm/article/details/81284921

##数据库评测报告第二期：MongoDB-3.2
https://cloud.tencent.com/developer/article/1005453

##引擎
###MongoDB版本及存储引擎区别 
https://www.cnblogs.com/Sungeek/p/12485009.html

WiredTiger
基于BTree结构组织数据，相比MongoDB早期的MMAPv1存储引擎性能提升明显，且支持数据压缩，存储成本更低。
默认存储引擎，适用于大多数业务场景。

##隔离
###update isolated 
https://docs.mongodb.com/v3.2/reference/operator/update/isolated/#up._S_isolated

##锁
###MongoDB中的读写锁
https://www.cnblogs.com/duanxz/p/10737548.html

2. 锁的粒度

在 2.2 版本以前，一个mongodb实例一个写锁，多个读锁。也就是说mongod 只有全局锁(锁定一个server)；

在2.2-3.0的版本，一个数据库一个写锁，多个读锁。例如如果一个 mongod 实例上有 5 个库，如果只对一个库中的一个集合执行写操作，那么在写操作过程中，这个库被锁；而其它 4 个库不影响。相比 RDBMS 来说，这个粒度已经算很大了！

在3.0之后的版本，WiredTiger提供了文档（不是集合）级别的锁。

更新：MongoDB 3.4版本，写操作的锁定粒度在表中数据记录(document)级别，即使操作对象可能是多条数据，每条数据在被写入时都会被锁定，防止其他进程写入；但是写操作是非事务性的，即写入多条数据，即使当前写入操作还没有完成，前面已经写入的数据也可以被其他进程修改。除非指定了$isolated，一次写入操作影响的数据无法在本次操作结束之前被其他进程修改。
$isolated也是非事务性的，即如果写入过程出错，已经完成的写入操作不会被rollback；另外，$isolated需要额外的锁，无法用于sharded方式部署的集群。
官网文档链接

MongoDB高吞吐的原因：

MongoDB 没有完整事务支持，操作原子性只到单个 document 级别，所以通常操作粒度比较小；
MongoDB 锁实际占用时间是内存数据计算和变更时间，通常很快；
MongoDB 锁有一种临时放弃机制，当出现需要等待慢速 IO 读写数据时，可以先临时放弃，等 IO 完成之后再重新获取锁。
 

3. 如何查看锁的状态

db.serverStatus()
db.currentOp()
mongotop # 类似top命令，每秒刷新
mongostat
the MongoDB Monitoring Service (MMS)


4. 哪些操作会对数据库产生锁？

下表列出了常见数据库操作产生的锁。

操作	锁定类型
查询	读锁
通过cursor读取数据	读锁
插入数据	写锁
删除数据	写锁
修改数据	写锁
Map-reduce	读写锁均有，除非指定为non-atomic，部分mapreduce任务可以同时执行(猜测是生成的中间表不冲突的情况下)
添加index	通过前台API添加index，锁定数据库一段时间
db.eval()	写锁，同时阻塞其他运行在MongoDB上的JavaScript进程
eval	写锁，如果设定锁定选项是nolock，则不会有些锁，而且eval无法向数据库写入数据
aggregate()	读锁
 

附上原文：
Operation Lock Type
Issue a query Read lock
Get more data from a cursor Read lock
Insert data Write lock
Remove data Write lock
Update data Write lock
Map-reduce Read lock and write lock, unless operations are specified as non-atomic. Portions of map-reduce jobs can run concurrently.
Create an index Building an index in the foreground, which is the default, locks the database for extended periods of time.
db.eval() Write lock. db.eval() blocks all other JavaScript processes.
eval Write lock. If used with the nolock lock option, the eval option does not take a write lock and cannot write data to the database.
aggregate() Read lock

###Mongodb锁机制
https://blog.csdn.net/tang_jin2015/article/details/61192020

Mongodb使用读写锁来允许很多用户同时去读一个资源，比如数据库或者集合。读采用的是共享锁，写采用的是排它锁。

对于大部分的读写操作，WiredTiger使用的都是乐观锁，在全局、数据库、集合级别，WiredTiger使用的是意向锁。当引擎探测到两个操作之间发生了冲突，将会产生一个写冲突，mongodb将会重新执行操作。只有如删除集合等操作需要排它锁。



##分布式一致性协议
###分布式一致性协议在MongoDB选举中的应用
https://www.jianshu.com/p/916e5e443ad7

算法应用
MongoDB在实现时对Raft算法做了一些调整：

Secondary节点不只是被动接受，也会发起心跳监测
当Secondary节点发现集群中没有Primary或者自己priority比Primary高时发起选举
保留了优先级但只考虑自己和当前主结点的优先级，其他结点的优先级不决定选举与否，也不影响投票


对比新旧版本的选举：

Raft为MongoDB选举引入Term，取消Bully的选举锁，效率更高，更优雅的避免重复投票，减少投票等待时间。
Raft弱化了priority功能，可能出现非最高priority候选节点当选的情况，后续的心跳中会发现，并重新选举。
取消了veto，选举不一定需要等待心跳超时。
主节点的降级有自己发起，效率更高。


##Raft与MongoDB复制集协议比较
https://www.cnblogs.com/xybaby/p/10165564.html

##自增id
###mongodb实现主键自增
https://www.cnblogs.com/arcticBoiledWater/p/9681498.html

###Mongodb 自动增长 自增id 实现 
http://www.dotcoo.com/post-39.html
https://blog.csdn.net/cyuyan112233/article/details/19769291
https://www.runoob.com/mongodb/mongodb-autoincrement-sequence.html

db.user.save({
    uid: db.tt.findAndModify({
        update:{$inc:{'id':1}},
        query:{"name":"user"},
        upsert:true,
		new:true
    }).id,  
    username: "dotcoo",
    password:"dotcoo",
    info:"http://www.dotcoo.com/"
})

db.system.js.insert(
{_id:"getNextSequence",value:function getNextSequence(name) {
   var ret = db.counters.findAndModify(
          {
            query: { _id: name },
            update: { $inc: { seq: 1 } },
            new: true
          }
   );
   return ret.seq;
}
});

<?php
function mid($name, $db){
    $update = array('$inc'=>array("id"=>1));
    $query = array('name'=>$name);
    $command = array(
            'findandmodify'=>'ids', 'update'=>$update,
            'query'=>$query, 'new'=>true, 'upsert'=>true
    );
    $id = $db->command($command);
    return $id['value']['id'];
}
  
$conn = new Mongo();
$db = $conn->idtest;
$id = mid('user', $db);
$db->user->save(array('uid'=>$id, 'username'=>'kekeles', 'password'=>'kekeles', 'info'=>'http://www.dotcoo.com/  '));
$conn->close();
?>

##关于 mongodb 频繁写操作的性能问题
https://www.v2ex.com/amp/t/334192

##mongodb的update和findAndModify有什么区别？
https://sq.163yun.com/ask/question/129317702368153600

##mongoDB数据更新与操作符
https://blog.csdn.net/a903178574/article/details/102110862

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

##【MongoDB】shard 片键选择

选择片键需要慎重，因为一旦选定就无法更改了。

常用的有两种选择：

（1）升序片键，常见的比如_id,date,来自其他数据库的自增主键等。如果使用升序片键，那么数据物理上会是连续的，有利于基于范围的查询，因为数据库一次拿进来若干的数据块，由于是连续排列的，那么后续的数据就不需要再通过io拿新的块。但是升序键有个明显的问题是，数据只会插入到最大块里。最开始只有一个块，然后随着数据的插入，块会分裂为两块，一个是范围值较小的，一个是较大的，后续的数据因为是增序的，所以只会插入到较大快内，所以，每一次的分裂，都会导致数据值较小的块没有任何写入，所有的写入只会集中到最大块，这会降低写入性能。而且，由于其他块分裂以后没有写入请求，那么这些块的空间利用率不高。

（2）随机片键，可以使用MongoDB自带的hash key机制，这样，所有的写入请求都会被均衡的转发到各个数据块。解决了上面写入效率低的问题。但是基于范围的查询效率变低。因为数据不是按照片键值物理排放，这样相邻的数据值可能位于不同的数据块，这样需要更多的IO次数拿块到内存。



有一个比较好的方法结合上述两者的优点，也就是复合片键策略。

片键由两部分构成，第一部分是一个随机值，但是值的度比较少，与分片数目大致一样或者常数倍。第二部分是升序值。这样，每一条数据会根据第一部分做分发，分发以后每一个片内部是升序的，这样查找的时候，可以按照分片来查，每一个分片内部是升序的，效率会变高。

比如第一部分是{A,B,C}, 第二部分升序，那么片键可能是A1 B2 C3 A4 B5 C6 A10 B15 C20这样。

##mongodb语法
mongodb查询的语法（大于，小于，大于或等于，小于或等于等等）  https://www.cnblogs.com/Logan626/articles/4858728.html

##MongoDB两阶段提交实现事务
https://blog.csdn.net/after_you/article/details/68059774

##Mongodb副本集容灾
https://blog.csdn.net/sunbocong/article/details/78643457

##Mongodb集群的三种搭建方式
https://www.jianshu.com/p/09a78243bc8f

最近，对Mongodb的勒索案件层出不穷，大多数是对于Mongo没有进行安全认证的配置，导致了数据直接被攻击者拿到。我们今天讲讲如何配置Mongo集群，并进行安全配置。
Mongo有三种集群方式
1.Replica Set副本

2.Sharding分片

3.Master-slave主备
通常来说，我们用第1、2种较多，第3种官方并不推荐。下面，我们来讲解下这三种集群方式的搭建方式。本文，假设读者已经对Docker有所了解，我们整个过程使用Docker搭建Mongo的集群环境。


###2.Sharding分片集群

现假设我们有三台服务器，三台服务器上配置各配置一个分片，一个配置服务，一个Router服务。其中，每个分片，都是Replica Set
####1.数据容器Master
docker run -d -p 27018:27018 --name mongodb_shard_master -v /share/disk0/mongodb_cluster/mongodb_node/shard_server_master/data:/data/db mongo:latest mongod --shardsvr --port 27018 --replSet rs1 --dbpath /data/db

####2.数据容器Slaver
docker run -d -p 27118:27118 --name mongodb_shard_slaver -v /share/disk0/mongodb_cluster/mongodb_node/shard_server_slaver/data:/data/db mongo:latest mongod --shardsvr --port 27118 --replSet rs1 --dbpath /data/db

####3.数据容器Arbiter
docker run -d -p 27118:27118 --name mongodb_shard_arbiter -v /share/disk0/mongodb_cluster/mongodb_node/shard_server_arbiter/data:/data/db mongo:latest mongod --shardsvr --port 27118 --replSet rs1 --dbpath /data/db

####4.配置Sharding Replica Set
docker exec -it mongo_master /bin/sh
mongo --port 27018
rs.initiate()
rs.add("服务器IP: 27118")
rs.addArb("服务器IP: 27218")
rs.conf()
rs.status()
use admin
db.createUser( {
user: "admin",
pwd: "P@ssw0rd",
roles: [ { role: "root", db: "admin" } ]
});

####5.配置服务容器
docker run -t -i -d -p 27019:27019 --name mongodb_configrs -v /share/disk0/mongodb_cluster/mongodb_node/config_server/data:/data/db mongo:latest mongod --configsvr --replSet csrs --dbpath /data/db
这里--configsvr默认是27019端口
假设在服务器1上运行如下命令
docker exec -it mongodb_configrs /bin/sh
mongo --port 27019
rs.initiate()
rs.add("服务器2IP: 27019")
rs.addArb("服务器3IP: 27019")
rs.conf()
rs.status()

####6.路由服务容器
这里需要在服务器2、3上先运行配置服务容器
docker run -t -i -d -p 27020:27020 --name mongodb_router -v /share/disk0/mongodb_cluster/mongodb_node1/router_server/data:/data/db mongo:latest mongos --configdb csrs/服务器1IP:27019,服务器2IP:27019,服务器3IP:27019 --port 27020
这样，路由服务容器和配置服务容器建立了关系。然后通过路由服务添加分片到配置服务中
docker exec -it mongodb_router /bin/sh
mongo --port 27020
use admin
db.runCommand({addshard:"rs1/服务器1IP:27018"}) //会自动找到rs1中的备用和仲裁节点
db.runCommand({addshard:"rs2/服务器2IP:27018"})
db.runCommand({addshard:"rs3/服务器3IP:27018"})
db.runCommand({listshards:1})
use admin
db.createUser( {
user: "admin",
pwd: "P@ssword",
roles: [ { role: "root", db: "admin" } ]
});

####7.最后，通第一节中，删除分片容器和路由容器，在启动容器时，带上--auth。这样，连接Mongodb时，必须使用用户名密码登陆。
3.Master-Slave
这种模式比较简单，但是官方已经不推荐使用。只用于开发时调试可以使用。
服务器1
docker run -t -i -d -p 27018:27018 --name mongodb_master -v /share/disk0/mongodb_cluster/mongodb_node/master_server/data:/data/db mongo:latest mongod --port 27018 --master --dbpath /data/db
docker exec -it mongodb_master /bin/sh
use admin
db.createUser( {
user: "admin",
pwd: "P@ssword",
roles: [ { role: "root", db: "admin" } ]
});
docker stop mongodb_master
docker rm mongodb_master
docker run -t -i -d -p 27018:27018 --name mongodb_master -v /share/disk0/mongodb_cluster/mongodb_node/master_server/data:/data/db mongo:latest mongod --auth --port 27018 --master --dbpath /data/db
服务器2
docker run -t -i -d -p 27018:27018 --name mongodb_slave -v /share/disk0/mongodb_cluster/mongodb_node/slave_server/data:/data/db mongo:latest mongod --port 27018 --slave --source 服务器1IP: 27018 --dbpath /data/db
这里，同服务器1一样设置密码即可。


##高可用的MongoDB集群
https://www.jianshu.com/p/2825a66d6aed
大概介绍一下MongoDB集群的几种方式

###Master-Slave、Relica Set、Sharding，并做简单的演示。
使用集群的目的就是提高可用性。高可用性H.A.（High Availability）指的是通过尽量缩短因日常维护操作（计划）和突发的系统崩溃（非计划）所导致的停机时间，以提高系统和应用的可用性。它与被认为是不间断操作的容错技术有所不同。HA系统是目前企业防止核心计算机系统因故障停机的最有效手段。
HA的三种工作方式：
主从方式 （非对称方式）
工作原理：主机工作，备机处于监控准备状况；当主机宕机时，备机接管主机的一切工作，待主机恢复正常后，按使用者的设定以自动或手动方式将服务切换到主机上运行，数据的一致性通过共享存储系统解决。
双机双工方式（互备互援）
工作原理：两台主机同时运行各自的服务工作且相互监测情况，当任一台主机宕机时，另一台主机立即接管它的一切工作，保证工作实时，应用服务系统的关键数据存放在共享存储系统中。
集群工作方式（多服务器互备方式）
工作原理：多台主机一起工作，各自运行一个或几个服务，各为服务定义一个或多个备用主机，当某个主机故障时，运行在其上的服务就可以被其它主机接管

###主从架构（Master-Slave）
Mater-Slaves
主从架构一般用于备份或者做读写分离。由两种角色构成：
主(Master)
可读可写，当数据有修改的时候，会将oplog同步到所有连接的salve上去。
从(Slave)
只读不可写，自动从Master同步数据。
特别的，对于Mongodb来说，并不推荐使用Master-Slave架构，因为Master-Slave其中Master宕机后不能自动恢复，推荐使用Replica Set，后面会有介绍，除非Replica的节点数超过50，才需要使用Master-Slave架构，正常情况是不可能用那么多节点的。
还有一点，Master-Slave不支持链式结构，Slave只能直接连接Master。Redis的Master-Slave支持链式结构，Slave可以连接Slave，成为Slave的Slave。
下面演示一下搭建过程：
1>. 启动Master
mongod --port 2000 --master --dbpath masterdb/

2>. 启动Slave
mongod --port 2001 --slave --source 127.0.0.1:2000 --dbpath slavedb/

3>. 给Master里面导入数据，查看Master和Slave的数据。你会发现导入Master的数据同时也会在Slave中出现。
mongoimport --port 2000 -d test -c dataset dataset.json

mongo --port 2000 test
db.dataset.count()

> 25359

mongo --port 2001 test
db.dataset.count()

> 25359

4>. 试一下Master和Slave的写操作。你会发现，只有Master才可以对数据进行修改，Slave修改时候会报错。
mongo --port 2001 test
db.dataset.drop()
>  Error: drop failed: { "note" : "from execCommand", "ok" : 0, "errmsg" : "not master" }
mongoimport --port 2001 -d test -c dataset dataset.json
> Failed: error checking connected node type: no reachable servers

###副本集架构（Replica Set）
为了防止单点故障就需要引副本（Replication），当发生硬件故障或者其它原因造成的宕机时，可以使用副本进行恢复，最好能够自动的故障转移（failover）。有时引入副本是为了读写分离，将读的请求分流到副本上，减轻主（Primary）的读压力。而Mongodb的Replica Set都能满足这些要求。
Replica Set的一堆mongod的实例集合，它们有着同样的数据内容。包含三类角色：

主节点（Primary）
接收所有的写请求，然后把修改同步到所有Secondary。一个Replica Set只能有一个Primary节点，当Primar挂掉后，其他Secondary或者Arbiter节点会重新选举出来一个主节点。默认读请求也是发到Primary节点处理的，需要转发到Secondary需要客户端修改一下连接配置。

副本节点（Secondary）
与主节点保持同样的数据集。当主节点挂掉的时候，参与选主。

仲裁者（Arbiter）
不保有数据，不参与选主，只进行选主投票。使用Arbiter可以减轻数据存储的硬件需求，Arbiter跑起来几乎没什么大的硬件资源需求，但重要的一点是，在生产环境下它和其他数据节点不要部署在同一台机器上。
注意，一个自动failover的Replica Set节点数必须为奇数，目的是选主投票的时候要有一个大多数才能进行选主决策。

应用客户端
客户端连接单个mongod和副本集的操作是相同，只需要配置好连接选项即可，比如下面是node.js连接Replica Set的方式：
mongoose.connect('mongodb://[username:password@]host1[:port1][,host2[:port2],...[,hostN[:portN]]][/[database][?options]]' [, options]);

Primary和Secondary搭建的Replica Set

Primary和Secondary搭建的Replica Set
奇数个数据节点构成的Replica Set，下面演示精典的3个数据节点的搭建过程。
1> 启动3个数据节点，--relSet指定同一个副本集的名字
mongod --port 2001 --dbpath rs0-1 --replSet rs0
mongod --port 2002 --dbpath rs0-2 --replSet rs0
mongod --port 2003 --dbpath rs0-3 --replSet rs0

2> 连接到其中一个，配置Replica Set，同时正在执行rs.add的节点被选为Primary。开发环境中<hostname>指的是机器名，生产环境下就是机器的IP。
mongo --port 2001

rs.initiate()
rs.add("<hostname>:2002")
rs.add("<hostname>:2003")
rs.conf()

3> 连接Primary节点，导入数据成功。
mongoimport --port 2001 -d test -c dataset dataset.json
mongo --port 2001 test
db.dataset.count()
> 25359

4> 默认情况下，Secondary不能读和写。
mongo --port 2003 test
db.dataset.count()
> Error: count failed: { "note" : "from execCommand", "ok" : 0, "errmsg" : "not master" }

注意，其中Secondary宕机，不受影响，若Primary宕机，会进行重新选主：

自动Failover
使用Arbiter搭建Replica Set
偶数个数据节点，加一个Arbiter构成的Replica Set，下面演示精典的2个数据节点加一个仲裁者的搭建过程。
特别的，生产环境中的Arbiter节点，需要修改一下配置：
journal.enabled = false
smallFiles = true

使用Arbiter搭建Replica Set
1> 启动两个数据节点和一个Arbiter节点
mongod --port 2001 --dbpath rs0-1 --replSet rs0
mongod --port 2002 --dbpath rs0-2 --replSet rs0

mongod --port 2003 --dbpath arb --replSet rs0

2> 连接到其中一个，添加Secondary和Arbiter。当仅需要添加Aribiter的时候，只需连接当前Replica Set的Primary，然后执行rs.addArb。
mongo --port 2001

rs.initiate()
rs.add("<hostname>:2002")
rs.addArb("<hostname>:2003")
rs.conf()

###数据分片架构（Sharding）
当数据量比较大的时候，我们需要把数据分片运行在不同的机器中，以降低CPU、内存和IO的压力，Sharding就是这样的技术。数据库主要由两种方式做Sharding：纵向，横向，纵向的方式就是添加更多的CPU，内存，磁盘空间等。横向就是上面说的方式，如图所示：

MongoDB的Sharding架构：

MongoDB的Sharding架构
MongoDB分片架构中的角色：
数据分片（Shards）
保存数据，保证数据的高可用性和一致性。可以是一个单独的mongod实例，也可以是一个副本集。在生产环境下Shard是一个Replica Set，以防止该数据片的单点故障。所有Shard中有一个PrimaryShard，里面包含未进行划分的数据集合：

查询路由（Query Routers）
mongos的实例，客户端直接连接mongos，由mongos把读写请求路由到指定的Shard上去。一个Sharding集群，可以有一个mongos，也可以有多mongos以减轻客户端请求的压力。
配置服务器（Config servers）
保存集群的元数据（metadata），包含各个Shard的路由规则。
搭建一个有2个shard的集群
1> 启动两个数据分片节点。在此仅演示单个mongod的方式，Replica Set类似。
mongod --port 2001 --shardsvr --dbpath shard1/
mongod --port 2002 --shardsvr --dbpath shard2/


2> 启动配置服务器
mongod --port 3001 --dbpath cfg1/
mongod --port 3002 --dbpath cfg2/
mongod --port 3003 --dbpath cfg3/

3> 启动查询路由mongos服务器
mongos --port 5000 --configdb 127.0.0.1:3001,127.0.0.1:3002,127.0.0.1:3003

4> 连接mongos，为集群添加数据分片节点。
mongo --port 5000 amdmin

sh.addShard("127.0.0.1:2001")
sh.addShard("127.0.0.1:2002")

如果Shard是Replica Set，添加Shard的命令：
sh.addShard("rsname/host1:port,host2:port,...")

rsname - 副本集的名字

5> 可以连接mongos进行数据操作了。
mongo --port 5000 test

mongoimport.exe --port 5000 -d test dataset.json
> 25359

数据的备份和恢复
MongodDB的备份有多种方式，这里只简单介绍一下mongodump和mongorestore的用法。
1> 备份和恢复所有db
mongodump -h IP --port PORT -o BACKUPPATH

mongorestore -h IP --port PORT BACKUPPATH

2> 备份和恢复指定db
mongodump -h IP --port PORT -d DBNAME -o BACKUPPATH

mongorestore -h IP --port PORT  -d DBNAME BACKUPPATH
mongorestore -h IP --port PORT --drop -d DBNAME BACKUPPATH

3> 备份和恢复指定collection
mongodump -h IP --port PORT -d DBNAME -c COLLECTION -o xxx.bson

mongorestore -h IP --port PORT  -d DBNAME -c COLLECTION xxx.bson
mongorestore -h IP --port PORT --drop -d DBNAME -c COLLECTION xxx.bson

小结
MongoDB的集群能力还是很强的，搭建还算是简单。最关键的是要明白上面提到的3种架构的原理，才能用的得心应手。当然不限于MongoDB，或许其他数据库也多多少少支持类似的架构。
参考资料
百度百科： http://baike.baidu.com/view/2850255.htm 
MongodDB官网文档：http://docs.mongodb.org/

##MongoDB在58同城百亿量级数据下的应用实践
https://www.jianshu.com/p/ea35f248cc68

58同城作为中国最大的生活服务平台，涵盖了房产、招聘、二手、二手车、黄页等核心业务。58同城发展之初，大规模使用关系型数据库（SQL Server、MySQL等），随着业务扩展速度增加，数据量和并发量演变的越来越有挑战，此阶段58的数据存储架构也需要相应的调整以更好的满足业务快速发展的需求。
MongoDB经过几个版本的迭代，到2.0.0以后，变的越来越稳定，它具备的高性能、高扩展性、Auto-Sharding、Free-Schema、类SQL的丰富查询和索引等特性，非常诱惑，同时58同城在一些典型业务场景下使用MongoDB也较合适，2011年，我们开始使用MongoDB，逐步扩大了使用的业务线，覆盖了58帮帮、58交友、58招聘、信息质量等等多条业务线。

随着58每天处理的海量数据越来越大，并呈现不断增多的趋势，这为MongoDB在存储与处理方面带来了诸多的挑战。面对百亿量级的数据，我们该如何存储与处理，本文将详细介绍MongoDB遇到的问题以及最终如何“完美”解决。

本文详细讲述MongoDB在58同城的应用实践：MongoDB在58同城的使用情况；为什么要使用MongoDB；MongoDB在58同城的架构设计与实践；针对业务场景我们在MongoDB中如何设计库和表；数据量增大和业务并发，我们遇到典型问题及其解决方案；MongoDB如何监控。

MongoDB在58同城的使用情况
MongoDB在58同城的众多业务线都有大规模使用：58转转、58帮帮、58交友、58招聘、58信息质量、58测试应用等，如[图1]所示。


相关厂商内容
华为软件开发云专区全新上线！

Airbnb 的通用数据产品平台架构开发设计思路
Apache Beam 大规模乱序流数据处理
Apache Kafka：大数据的实时处理分析实战
阿里新一代实时计算引擎 Blink 典型的场景应用

图1 MongoDB典型的使用场景：转转

为什么要使用MongoDB？
MongoDB这个来源英文单词“humongous”，homongous这个单词的意思是“巨大的”、“奇大无比的”，从MongoDB单词本身可以看出它的目标是提供海量数据的存储以及管理能力。MongoDB是一款面向文档的NoSQL数据库，MongoDB具备较好的扩展性以及高可用性，在数据复制方面，支持Master-Slaver（主从）和Replica-Set（副本集）等两种方式。通过这两种方式可以使得我们非常方便的扩展数据。
MongoDB较高的性能也是它强有力的卖点之一，存储引擎使用的内存映射文件（MMAP的方式），将内存管理工作交给操作系统去处理。MMAP的机制，数据的操作写内存即是写磁盘，在保证数据一致性的前提下，提供了较高的性能。
除此之外，MongoDB还具备了丰富的查询支持、较多类型的索引支持以及Auto-Sharding的功能。在所有的NoSQL产品中，MongoDB对查询的支持是最类似于传统的RDBMS，这也使得应用方可以较快的从RDBMS转换到MonogoDB。
在58同城，我们的业务特点是具有较高的访问量，并可以按照业务进行垂直的拆分，在每个业务线内部通过MongoDB提供两种扩展机制，当业务存储量和访问量变大，我们可以较易扩展。同时我们的业务类型对事务性要求低，综合业务这几点特性，在58同城使用MongoDB是较合适的。
如何使用MongoDB？
MongoDB作为一款NoSQL数据库产品，Free Schema是它的特性之一，在设计我们的数据存储时，不需要我们固定Schema，提供给业务应用方较高的自由度。那么问题来了，Free Schema真的Free吗？
第一：Free Schema意味着重复Schema。在MongoDB数据存储的时候，不但要存储数据本身，Schema（字段key）本身也要重复的存储（例如：{“name”:”zhuanzhuan”, “infoid”:1,“infocontent”:”这个是转转商品”}），必然会造成存储空间的增大。
第二：Free Schema意味着All Schema，任何一个需要调用MongoDB数据存储的地方都需要记录数据存储的Schema，这样才能较好的解析和处理，必然会造成业务应用方的复杂度。
那么我们如何应对呢？在字段名Key选取方面，我们尽可能减少字段名Key的长度，比如：name字段名使用n来代替，infoid字段名使用i来代替，infocontent字段名使用c来代替（例如：{“n”:”zhuanzhuan”, “i”:1, “c”:”这个是转转商品”}）。使用较短的字段名会带来较差的可读性，我们通过在使用做字段名映射的方式（ #defineZZ_NAME ("n")），解决了这个问题；同时在数据存储方面我们启用了数据存储的压缩，尽可能减少数据存储的量。
MongoDB提供了自动分片（Auto-Sharding）的功能，经过我们的实际测试和线上验证，并没有使用这个功能。我们启用了MongoDB的库级Sharding；在CollectionSharding方面，我们使用手动Sharding的方式，水平切分数据量较大的文档。
MongoDB的存储文档必须要有一个“_id”字段，可以认为是“主键”。这个字段值可以是任何类型，默认一个ObjectId对象，这个对象使用了12个字节的存储空间，每个字节存储两位16进制数字，是一个24位的字符串。这个存储空间消耗较大，我们实际使用情况是在应用程序端，使用其他的类型（比如int）替换掉到，一方面可以减少存储空间，另外一方面可以较少MongoDB服务端生成“_id”字段的开销。
在每一个集合中，每个文档都有唯一的“_id”标示，来确保集中每个文档的唯一性。而在不同集合中，不同集合中的文档“_id”是可以相同的。比如有2个集合Collection_A和Collection_B，Collection_A中有一个文档的“_id”为1024，在Collection_B中的一个文档的“_id”也可以为1024。


###MongoDB集群部署
MongoDB集群部署我们采用了Sharding+Replica-Set的部署方式。整个集群有Shard Server节点（存储节点，采用了Replica-Set的复制方式）、Config Server节点（配置节点）、Router Server（路由节点、Arbiter Server（投票节点）组成。每一类节点都有多个冗余构成。满足58业务场景的一个典型MongoDB集群部署架构如下所示[图2]：


图2 58同城典型业务MongoDB集群部署架构
在部署架构中，当数据存储量变大后，我们较易增加Shard Server分片。Replica-Set的复制方式，分片内部可以自由增减数据存储节点。在节点故障发生时候，可以自动切换。同时我们采用了读写分离的方式，为整个集群提供更好的数据读写服务。


图3 Auto-Sharding MAY is not that Reliable
针对业务场景我们在MongoDB中如何设计库和表
MongoDB本身提供了Auto-Sharding的功能，这个智能的功能作为MongoDB的最具卖点的特性之一，真的非常靠谱吗（图3）？也许理想是丰满的，现实是骨干滴。
首先是在Sharding Key选择上，如果选择了单一的Sharding Key，会造成分片不均衡，一些分片数据比较多，一些分片数据较少，无法充分利用每个分片集群的能力。为了弥补单一Sharding Key的缺点，引入复合Sharing Key，然而复合Sharding Key会造成性能的消耗；
第二count值计算不准确的问题，数据Chunk在分片之间迁移时，特定数据可能会被计算2次，造成count值计算偏大的问题；
第三Balancer的稳定性&智能性问题，Sharing的迁移发生时间不确定，一旦发生数据迁移会造成整个系统的吞吐量急剧下降。为了应对Sharding迁移的不确定性，我们可以强制指定Sharding迁移的时间点，具体迁移时间点依据业务访问的低峰期。比如IM系统，我们的流量低峰期是在凌晨1点到6点，那么我们可以在这段时间内开启Sharding迁移功能，允许数据的迁移，其他的时间不进行数据的迁移，从而做到对Sharding迁移的完全掌控，避免掉未知时间Sharding迁移带来的一些风险。

###如何设计库（DataBase）？
我们的MongoDB集群线上环境全部禁用了Auto-Sharding功能。如上节所示，仅仅提供了指定时间段的数据迁移功能。线上的数据我们开启了库级的分片，通过db.runCommand({“enablesharding”: “im”});命令指定。并且我们通过db.runCommand({movePrimary:“im”, to: “sharding1”});命令指定特定库到某一固定分片上。通过这样的方式，我们保证了数据的无迁移性，避免了Auto-Sharding带来的一系列问题，数据完全可控，从实际使用情况来看，效果也较好。
既然我们关闭了Auto-Sharding的功能，就要求对业务的数据增加情况提前做好预估，详细了解业务半年甚至一年后的数据增长情况，在设计MongoDB库时需要做好规划：确定数据规模、确定数据库分片数量等，避免数据库频繁的重构和迁移情况发生。

那么问题来了，针对MongoDB，我们如何做好容量规划？

MongoDB集群高性能本质是MMAP机制，对机器内存的依赖较重，因此我们要求业务热点数据和索引的总量要能全部放入内存中，即：Memory > Index + Hot Data。一旦数据频繁地Swap，必然会造成MongoDB集群性能的下降。当内存成为瓶颈时，我们可以通过Scale Up或者Scale Out的方式进行扩展。

第二：我们知道MongoDB的数据库是按文件来存储的：例如：db1下的所有collection都放在一组文件内db1.0,db1.1,db1.2,db1.3……db1.n。数据的回收也是以库为单位进行的，数据的删除将会造成数据的空洞或者碎片，碎片太多，会造成数据库空间占用较大，加载到内存时也会存在碎片的问题，内存使用率不高，会造成数据频繁地在内存和磁盘之间Swap，影响MongoDB集群性能。因此将频繁更新删除的表放在一个独立的数据库下，将会减少碎片，从而提高性能。

第三：单库单表绝对不是最好的选择。原因有三：表越多，映射文件越多，从MongoDB的内存管理方式来看，浪费越多；同理，表越多，回写和读取的时候，无法合并IO资源，大量的随机IO对传统硬盘是致命的；单表数据量大，索引占用高，更新和读取速度慢。

第四：Local库容量设置。我们知道Local库主要存放oplog，oplog用于数据的同步和复制，oplog同样要消耗内存的，因此选择一个合适的oplog值很重要，如果是高插入高更
新，并带有延时从库的副本集需要一个较大的oplog值（比如50G）；如果没有延时从库，并且数据更新速度不频繁，则可以适当调小oplog值（比如5G）。总之，oplog值大小的设置取决于具体业务应用场景，一切脱离业务使用场景来设置oplog的值大小都是耍流氓。

###如何设计表（Collection）？
MongoDB在数据逻辑结构上和RDBMS比较类似，如图4所示：MongoDB三要素：数据库（DataBase）、集合（Collection）、文档（Document）分别对应RDBMS（比如MySQL）三要素：数据库（DataBase）、表（Table）、行（Row）。


图4 MongoDB和RDBMS数据逻辑结构对比
MongoDB作为一支文档型的数据库允许文档的嵌套结构，和RDBMS的三范式结构不同，我们以“人”描述为例，说明两者之间设计上的区别。“人”有以下的属性：姓名、性别、年龄和住址；住址是一个复合结构，包括：国家、城市、街道等。针对“人”的结构，传统的RDBMS的设计我们需要2张表：一张为People表[图5]，另外一张为Address表[图6]。这两张表通过住址ID关联起来（即Addess ID是People表的外键）。在MongoDB表设计中，由于MongoDB支持文档嵌套结构，我可以把住址复合结构嵌套起来，从而实现一个Collection结构[图7]，可读性会更强。

图 5 RDBMSPeople表设计

图 6 RDBMS Address表设计

图7 MongoDB表设计
MongoDB作为一支NoSQL数据库产品，除了可以支持嵌套结构外，它又是最像RDBMS的产品，因此也可以支持“关系”的存储。接下来会详细讲述下对应RDBMS中的一对一、一对多、多对多关系在MongoDB中我们设计和实现。
IM用户信息表，包含用户uid、用户登录名、用户昵称、用户签名等，是一个典型的一对一关系，在MongoDB可以采用类RDBMS的设计，我们设计一张IM用户信息表user：{_id:88, loginname:musicml, nickname:musicml,sign:love}，其中_id为主键，_id实际为uid。IM用户消息表，一个用户可以收到来自他人的多条消息，一个典型的一对多关系。
我们如何设计？
一种方案，采用RDBMS的“多行”式设计，msg表结构为：{uid，msgid，msg_content}，具体的记录为：123, 1, 你好；123，2，在吗。
另外一种设计方案，我们可以使用MongoDB的嵌套结构：{uid:123, msg:{[{msgid:1,msg_content:你好}，{msgid:2, msg_content:在吗}]}}。
采用MongoDB嵌套结构，会更加直观，但也存在一定的问题：更新复杂、MongoDB单文档16MB的限制问题。采用RDBMS的“多行”设计，它遵循了范式，一方面查询条件更灵活，另外通过“多行式”扩展性也较高。
在这个一对多的场景下，由于MongoDB单条文档大小的限制，我们并没采用MongoDB的嵌套结构，而是采用了更加灵活的类RDBMS的设计。
在User和Team业务场景下，一个Team中有多个User，一个User也可能属于多个Team，这种是典型的多对多关系。
在MongoDB中我们如何设计？一种方案我们可以采用类RDBMS的设计。一共三张表：Team表{teamid,teamname, ……}，User表{userid,username,……}，Relation表{refid, userid, teamid}。其中Team表存储Team本身的元信息，User表存储User本身的元信息，Relation表存储Team和User的所属关系。
在MongoDB中我们可以采用嵌套的设计方案：一种2张表：Team表{teamid,teamname,teammates:{[userid, userid, ……]}，存储了Team所有的User成员和User表{useid,usename,teams:{[teamid, teamid,……]}}，存储了User所有参加的Team。
在MongoDB Collection上我们并没有开启Auto-Shariding的功能，那么当单Collection数据量变大后，我们如何Sharding？对Collection Sharding 我们采用手动水平Sharding的方式，单表我们保持在千万级别文档数量。当Collection数据变大，我们进行水平拆分。比如IM用户信息表：{uid, loginname, sign, ……}，可用采用uid取模的方式水平扩展，比如：uid%64，根据uid查询可以直接定位特定的Collection，不用跨表查询。
通过手动Sharding的方式，一方面根据业务的特点，我们可以很好满足业务发展的情况，另外一方面我们可以做到可控、数据的可靠，从而避免了Auto-Sharding带来的不稳定因素。对于Collection上只有一个查询维度（uid），通过水平切分可以很好满足。
但是对于Collection上有2个查询维度，我们如何处理？比如商品表：{uid, infoid, info,……}，存储了商品发布者，商品ID，商品信息等。我们需要即按照infoid查询，又能支持按照uid查询。为了支持这样的查询需求，就要求infoid的设计上要特殊处理：infoid包含uid的信息（infoid最后8个bit是uid的最后8个bit），那么继续采用infoid取模的方式，比如：infoid%64，这样我们既可以按照infoid查询，又可以按照uid查询，都不需要跨Collection查询。
数据量、并发量增大，遇到问题及其解决方案
大量删除数据问题及其解决方案
我们在IM离线消息中使用了MongoDB，IM离线消息是为了当接收方不在线时，需要把发给接收者的消息存储下来，当接收者登录IM后，读取存储的离线消息后，这些离线消息不再需要。已读取离线消息的删除，设计之初我们考虑物理删除带来的性能损耗，选择了逻辑标识删除。IM离线消息Collection包含如下字段：msgid, fromuid, touid, msgcontent, timestamp, flag。其中touid为索引，flag表示离线消息是否已读取，0未读，1读取。
当IM离线消息已读条数积累到一定数量后，我们需要进行物理删除，以节省存储空间，减少Collection文档条数，提升集群性能。既然我们通过flag==1做了已读取消息的标示，第一时间想到了通过flag标示位来删除：db.collection.remove({“flag” :1}};一条简单的命令就可以搞定。表面上看很容易就搞定了？！实际情况是IM离线消息表5kw条记录，近200GB的数据大小。
悲剧发生了：晚上10点后部署删除直到早上7点还没删除完毕；MongoDB集群和业务监控断续有报警；从库延迟大；QPS/TPS很低；业务无法响应。事后分析原因：虽然删除命令db.collection.remove({“flag” : 1}};很简单，但是flag字段并不是索引字段，删除操作等价于全部扫描后进行，删除速度很慢，需要删除的消息基本都是冷数据，大量的冷数据进入内存中，由于内存容量的限制，会把内存中的热数据swap到磁盘上，造成内存中全是冷数据，服务能力急剧下降。
遇到问题不可怕，我们如何解决呢？首先我们要保证线上提供稳定的服务，采取紧急方案，找到还在执行的opid，先把此命令杀掉（kill opid），恢复服务。长期方案，我们首先优化了离线删除程序[图8]，把已读IM离线消息的删除操作，每晚定时从库导出要删除的数据，通过脚本按照objectid主键（_id）的方式进行删除，并且删除速度通过程序控制，从避免对线上服务影响。其次，我们通过用户的离线消息的读取行为来分析，用户读取离线消息时间分布相对比较均衡，不会出现比较密度读取的情形，也就不会对MongoDB的更新带来太大的影响，基于此我们把用户IM离线消息的删除由逻辑删除优化成物理删除，从而从根本上解决了历史数据的删除问题。

图8 离线删除优化脚本
大量数据空洞问题及其解决方案
MongoDB集群大量删除数据后（比如上节中的IM用户离线消息删除）会存在大量的空洞，这些空洞一方面会造成MongoDB数据存储空间较大，另外一方面这些空洞数据也会随之加载到内存中，导致内存的有效利用率较低，在机器内存容量有限的前提下，会造成热点数据频繁的Swap，频繁Swap数据，最终使得MongoDB集群服务能力下降，无法提供较高的性能。
通过上文的描述，大家已经了解MongoDB数据空间的分配是以DB为单位，而不是以Collection为单位的，存在大量空洞造成MongoDB性能低下的原因，问题的关键是大量碎片无法利用，因此通过碎片整理、空洞合并收缩等方案，我们可以提高MongoDB集群的服务能力。
那么我们如何落地呢？
方案一：我们可以使用MongoDB提供的在线数据收缩的功能，通过Compact命令（db.yourCollection.runCommand(“compact”);）进行Collection级别的数据收缩，去除Collectoin所在文件碎片。此命令是以Online的方式提供收缩，收缩的同时会影响到线上的服务，其次从我们实际收缩的效果来看，数据空洞收缩的效果不够显著。因此我们在实际数据碎片收缩时没有采用这种方案，也不推荐大家使用这种空洞数据的收缩方案。
既然这种数据方案不够好，我们可以采用Offline收缩的方案二：此方案收缩的原理是：把已有的空洞数据，remove掉，重新生成一份无空洞数据。那么具体如何落地？先预热从库；把预热的从库提升为主库；把之前主库的数据全部删除；重新同步；同步完成后，预热此库；把此库提升为主库。
具体的操作步骤如下：检查服务器各节点是否正常运行 (ps -ef |grep mongod)；登入要处理的主节点 /mongodb/bin/mongo--port 88888；做降权处理rs.stepDown()，并通过命令 rs.status()来查看是否降权；切换成功之后，停掉该节点；检查是否已经降权，可以通过web页面查看status，我们建议最好登录进去保证有数据进入，或者是mongostat 查看； kill 掉对应mongo的进程： kill 进程号；删除数据，进入对应的分片删除数据文件，比如： rm -fr /mongodb/shard11/；重新启动该节点，执行重启命令，比如：如:/mongodb/bin/mongod --config /mongodb/shard11.conf；通过日志查看进程；数据同步完成后，在修改后的主节点上执行命令 rs.stepDown() ，做降权处理。
通过这种Offline的收缩方式，我们可以做到收缩率是100%，数据完全无碎片。当然做离线的数据收缩会带来运维成本的增加，并且在Replic-Set集群只有2个副本的情况下，还会存在一段时间内的单点风险。通过Offline的数据收缩后，收缩前后效果非常明显，如[图9,图10]所示：收缩前85G存储文件，收缩后34G存储文件，节省了51G存储空间，大大提升了性能。


图9 收缩MongoDB数据库前存储数据大小


图10 收缩MongoDB数据库后存储数据大小
MongoDB集群监控
MongoDB集群有多种方式可以监控：mongosniff、mongostat、mongotop、db.xxoostatus、web控制台监控、MMS、第三方监控。我们使用了多种监控相结合的方式*，从而做到对MongoDB整个集群完全Hold住。
第一是mongostat[图11]，mongostat是对MongoDB集群负载情况的一个快照，可以查看每秒更新量、加锁时间占操作时间百分比、缺页中断数量、索引miss的数量、客户端查询排队长度（读|写)、当前连接数、活跃客户端数量(读|写)等。




图11 MongoDB mongostat监控
mongstat可以查看的字段较多，我们重点关注Locked、faults、miss、qr|qw等，这些值越小越好，最好都为0；locked最好不要超过10%；造成faults、miss原因主要是内存不够或者内冷数据频繁Swap，索引设置不合理；qr|qw堆积较多，反应了数据库处理慢，这时候我们需要针对性的优化。
第二是web控制台，和MongoDB服务一同开启，它的监听端口是MongoDB服务监听端口加上1000，如果MongoDB的监听端口33333，则Web控制台端口为34333。我们可以通过http://ip:port（http://8.8.8.8:34333）访问监控了什么[图12]：当前MongoDB所有的连接数、各个数据库和Collection的访问统计包括：Reads, Writes, Queries等、写锁的状态、最新的几百行日志文件。




图12 MongoDB Web控制台监控
第三是MMS（MongoDBMonitoring Service），它是2011年官方发布的云监控服务，提供可视化图形监控。工作原理如下：在MMS服务器上配置需要监控的MongoDB信息（ip/port/user/passwd等）；在一台能够访问你MongoDB服务的内网机器上运行其提供的Agent脚本；Agent脚本从MMS服务器获取到你配置的MongoDB信息；Agent脚本连接到相应的MongoDB获取必要的监控数据；Agent脚本将监控数据上传到MMS的服务器；登录MMS网站查看整理过后的监控数据图表。具体的安装部署，可以参考：http://mms.10gen.com。




图13 MongoDB MMS监控
第四是第三方监控，MongoDB开源爱好者和团队支持者较多，可以在常用监控框架上扩展，比如：zabbix，可以监控CPU负荷、内存使用、磁盘使用、网络状况、端口监视、日志监视等；nagios，可以监控监控网络服务（HTTP等）、监控主机资源（处理器负荷、磁盘利用率等）、插件扩展、报警发送给联系人（EMail、短信、用户定义方式）、手机查看方式；cacti，可以基于PHP,MySQL,SNMP及RRDTool开发的网络流量监测图形分析工具。
最后我要感谢公司和团队，在MongoDB集群的大规模实战中积累了宝贵的经验，才能让我有机会撰写了此文，由于MongoDB社区不断发展，特别是MongoDB 3.0，对性能、数据压缩、运维成本、锁级别、Sharding以及支持可插拔的存储引擎等的改进，MongoDB越来越强大。文中可能会存在一些不妥的地方，欢迎大家交流指正。
讲师介绍
孙玄，极客邦培训专家讲师，58同城系统架构师、技术委员会架构组主任、产品技术学院优秀讲师，58同城即时通讯、C2C技术负责人，负责58核心系统的架构以及优化工作。分布式系统存储专家，2007年开始从事大规模高性能分布式存储系统架构设计实现工作。涉及自主研发分布式存储系统、MongoDB、MySQL、Memcached、Redis等。前百度高级工程师，参与社区搜索部多个基础系统的设计与实现。


##MongoDB的水平扩展，你做对了吗？
https://www.jianshu.com/p/f33570f0cd30

配置 Shard 数据库
环境搭建好并且数据已经准备完毕以后，接下来的事情就是配置数据库并切分数据。方便起见，我们把用户分为三组，20 岁以下（junior)，20 到 40 岁（middle）和 40 岁以上（senior），为了节省篇幅，我在这里不过多的介绍如何使用 MongoDB 命令，按照下面的几条命令执行以后，我们的数据会按照用户年龄段拆分成若干个 chunk，并分发到不同的 shard cluster 中。如果对下面的命令不熟悉，可以查看 MongoDB 官方文档关于 Shard Zone/Chunk 的解释。
db.getSiblingDB('test').getCollection('users').createIndex({'user.age':1})
sh.setBalancerState(false)
sh.addShardTag('shard01', 'junior')
sh.addShardTag('shard02', 'middle')
sh.addShardTag('shard03', 'senior')
sh.addTagRange('test.users', {'user.age': MinKey}, {'user.age':20}, 'junior') sh.addTagRange('test.users', {'user.age': 21}, {'user.age':40}, 'middle') sh.addTagRange('test.users', {'user.age': 41}, {'user.age': MaxKey}, 'senior')
sh.enableSharding('test')
sh.shardCollection('test.users', {'user.age':1})
sh.setBalancerState(true)
从上面的命令中可以看出，我们首先要为 Shard Key 创建索引，之后禁止 Balancer 的运行，这么做的原因是不希望在 Shard Collection 的过程中还运行 Balancer。之后将数据按照年龄分成三组，分别标记为junior,middle，senior并把这三组分别分配到三个 Shard 集群中。 之后对 test 库中的 users collection 进行按用户年龄字段的切分操作，如果 Shard collection 成功返回，你会得到下面的输出结果：{ "collectionsharded" : "test.users", "ok" : 1 }。

关于 Shard 需要注意的几点
一旦你对一个 Colleciton 进行了 Shard 操作，你选择的 Shard Key 和它对应的值将成为不可变对象，所以：
你无法在为这个 collection 重新选择 Shard Key
你不能更新 Shard key 的取值
随后不要忘记，我们还需要将 Balancer 打开：sh.setBalancerState(true)。刚打开以后运行sh.isBalancerRunning()应当返回true，说明 Balancer 服务正在运行，他会调整 Chunk 在不同 Shards 服务器中的分配。一般 Balancer 会运行一段时间，因为他要对分组的数据重新分配到指定的 shard 服务器上，你可以通过sh.isBalancerRunning()命令查看 Balancer 是否正在运行。现在可以稍事休息一下喝杯咖啡或看看窗外的风景。
为了理解数据如何分布在 3 个 shard 集群中，我们有必要分析一下 chunk 和 zone 的划分，下图是在 dbKoda 上显示 Shard Cluster 统计数据，可以看到数据总共被分成 6 个 chunks，每个 shard 集群存储 2 个 chunk。

Chunk
我们已经知道 MongoDB 是通过 shard key 来对数据进行切分，被切分出来的数据被分配到若干个 chunks 中。一个 chunk 可以被认为是一台 shard 服务器中数据的子集，根据 shard key，每个 chunk 都有上下边界，在我们的例子中，边界值就是用户年龄。chunk 有自己的大小，数据不断插入到 mongos 的过程中，chunk 的大小会发生变化，chunk 的默认大小是 64M。当然 MongoDB 允许你对 chunk 的大小进行设置，你也可以把一个 chunk 切分成若干个小 chunk，或者合并多个 chunk。一般我不建议大家手动操作 chunk 的大小，或者在 mongos 层面切分或合并 chunk，除非真有合适的原因才去这么做。原因是，在数据不断插入到我们的集群中时，mongodb 中的 chunk 大小会发生很大的变化，当一个 chunk 的大小超过了最大值，mongo 会根据 shard key 对 chunk 进行切分，在必要的时候，一个 chunk 可能会被切分成多个小 chunk，大多数情况下这种自动行为已经满足了我们日常的业务需求，无需进行手动操作，另一点原因是当进行 chunk 切分后，直接的结果会导致数据分配的不均匀，此时 balancer 会被调用来进行数据重新分配，很多时候这个操作会运行很长时间，无形中导致了内部结构的负载平衡，因此不建议大家手动拆分。当然，理解 chunk 的分配原理还是有助于大家分析数据库性能的必要条件。我在这里不过多的将如何进行这些操作，有兴趣的读者可以参考 MongoDB 官方文档，上面有比较全面的解释。这里我只强调在进行 chunk 操作的时候，要注意一下几个方面，这些都是影响你 MongoDB 性能的关键因素。
如果存在大量体积很小的 chunk，他可以保证你的数据均匀的分布在 shard 集群中但是可能会导致频繁的数据迁移。这将加重 mongos 层面上的操作。
大的 chunk 会减少数据迁移，减轻网络负担，降低在 mongos 路由层面上的负载，但弊端是有可能导致数据在 shard 集群中分布的不均匀。
Balancer 会在数据分配不均匀的时候自动运行，那么 Balancer 是如何决定什么情况下需要进行数据迁移呢？答案是 Migration Thresholds，当 chunk 的数量在不同 shard replica 之间超过一个定值时，balancer 会自动运行，这个定值根据你的 shard 数量不同而不同。


Zones
可以说 chunk 是 MongoDB 在多个 shard 集群中迁移数据的最小单元，有时候数据的分配不会按照我们臆想的方向进行，就拿上面的例子来说，虽然我们选择了用户年龄作为 shard key，但是 MongoDB 并不会按照我们设想的那样来分配数据，如何进行数据分配就是通过 Zones 来实现。Zones 解决了 shard 集群与 shard key 之间的关系，我们可以按照 shard key 对数据进行分组，每一组称之为一个 Zone，之后把 Zone 在分配给不同的 Shard 服务器。一个 Shard 可以存储一个或多个 Zone，前提是 Zone 之间没有数据冲突。Balancer 在运行的时候会把在 Zone 里的 chunk 迁移到关联这个 Zone 的 shard 上。
理解了这些概念以后，我们对数据的分配就有了更清楚的认识。我们对前面提到的问题就有了充分的解释。表面上看，数据的分布貌似均匀，我们执行几个查询语句看看性能怎样。这里再次用到 dbKoda 中的 explain 视图。

你选择的 Shard Key 合适吗？
了解了数据是如何分布的以后，咱们再回过头来看看我们选择的 shard key 是否合理。细心的读者已经发现，上面运行的 explain 结果中存在一个问题，就是 shard3 存储了大量的数据，如果我们看一下每个年龄组的纪录个数，会发现 shard1、shard2、shard3 分别包括 198554, 187975, 593673，显然年龄大于 40 岁的用户占了大多数。这并不是我们希望的结果，因为 shard3 成为了集群中的一个瓶颈，数据库操作语句在 shard3 上运行的速度会大大超过另外两个 shard，这点从上面的 explain 结果中也可以看到，查询语句在 shard3 上的运行时间是另外两个 shard 的两倍以上。更重要的是，随着用户数量的不断增加，数据的分布也会出现显著变化，在系统运行一段时间以后，可能 shard2 的用户数超过 shard3，也有可能 shard1 称为存储数据量最多的服务器。这种数据不平衡是我们不希望看到的。原因在哪里呢？是不是觉得我们选择的用户年龄作为分组条件是一个不太理想的 key。那么什么样的 key 能够保证数据的均匀分布呢？接下来我们分析一下 shard key 的种类。
Ranged Shard Key
我们上面选择的年龄分组就是用的这种 shard key。根据 shard key 的取值，它把数据切分成连续的几个区间。取值相近的纪录会放进同一个 shard 服务器。好处是查询连续取值纪录时，查询效率可以得到保证。当数据库查询语句发送到 mongos 中时，mongos 会很快的找到目标 shard，而且不需要将语句发送到所有的 shard 上，一般只需要少量的 shard 就可以完成查询操作。缺点是不能保证数据的平均分配，在数据插入和修改时会产生比较严重的性能瓶颈。
Hashed Shard Key
于 Ranged Shard Key 对应的一种被称之为 Hashed Shard Key，它采用字段的索引哈希值作为 shard key 的取值，这样做可以保证数据的均匀分布。在 mongos 和各个 shard 集群之间存在一个哈希值计算方法，所有的数据在迁移时都是根据这个方法来计算数据应当被迁移到什么地方。当 mongos 接收到一条语句时，通常他会把这条语句广播到所有的 shard 上去执行。
有了上面的认识，我们如何在 Ranged 和 Shard 之间进行选择呢？下面两个属性是我们选择 shard key 的关键。
Shard Key Cardinality （集）
Cardinality指的是 shard key 可以取到的不同值的个数。他会影响到 Balancer 的运行，这个值也可以被看做是 Balancer 可以创建的最大 chunk 个数。以我们年龄字段为例，假如一个人的年龄在 100 岁以下，那么这个字段的 cardinality 可以取 100 个不同的值。对于一个唯一的年龄数据，不会出现在不同的 chunk 中。如果你选择的 Shard Key 的 cardinality 很小，比如只有 4 个，那么数据最多会被分发到 4 个不同的 shard 中，这样的结构也不适合服务器的水平扩展，因为不会有数据被分割到第五个 shard 服务器上。
Shard Key Frequency（频率）
Frequency指的是 shard key 的重复度，也就是对于一个字段，有多少取值相同的纪录。如果大部分数据的 shard key 取值相同，那么存储他们的 chunk 会成为数据库的一个瓶颈。而且，这些 chunk 也变成了不可再切分的 chunk，严重影响了数据库的水平扩展。在这种情况下应当考虑使用组合索引的方式来创建 shard key。所以，尽量选择低频率的字段作为 shard key。
Shard Key Increasing Monotonically （单调增长）
单调增长在这里的意思是在数据被切分以后，新增加的数据会按照其 shard key 取值向 shard 中插入，如果新增的数据的 key 值都是向最大值方向增加，那么这些新的数据会被插入到同一个 shard 服务器上。例如我们前面的用户年龄分组字段，如果系统的新增用户都是年龄大于 40 岁的，那么 shard3 将会存储所有的新增用户，shard3 会成为系统的性能瓶颈。在这种情况下，应当考虑使用 Hashed Shard Key。
重新设计 Shard Key
通过上面的分析我们可以得出结论，前面例子中的用户年龄字段是一个很糟糕的方案。有几个原因：
用户的年龄不是固定不变的，由于 shard key 是不可变字段，一旦确定下来以后不能进行修改，所以年龄字段显然不是很合适，毕竟没有年龄永远不增长的用户。
一个系统的用户在不同年龄阶段的分布是不一样的，对于像游戏、娱乐方面的应用可能会吸引年轻人多一些。而对于医疗、养生方面也许会有更多老年人关注。从这一点上说，这样的切分也是不恰当的。
选择年龄字段并没有考虑到未来用户增长方面带来的问题，有可能在数据切分的时候年龄是均匀分布的，但是系统运行一段时间以后有可能出现不平等的数据分布，这点会给数据维护带来很大的困扰。
那么我们应当如何进行选择呢？看一下用户表的所有属性可以发现，其中有一个created_at字段，它指的是纪录创建的时间戳。如果采用 Ranged Key 那么在数据增长方向上会出现单调增长问题，在分析一下发现这个字段重复的纪录不多，他有很高的cardinality和非常低的频率，这样 Harded key 就成为了很好的备选方案。
分析完理论以后咱们实践一下看看效果，不幸的是我们并不能修改 shard key，最好的方法就是备份数据，重新创建 shard 集群。创建和数据准备的过程我就不在重复了，你们可以根据前面的例子自己作一遍。
下图中是我新建的一个userscollection，并以created_at为索引创建了 Hashed Shard Key，注意
created_at必须是一个 hash index 才能成为 hashed shard key。下面是针对用户表的一次查询结果。

​从图中可以看到，explain 的结果表示了三个 shard 服务器基本上均匀分布了所有的数据，三个 shard 上执行时间也都基本均匀，在 500 到 700 多毫秒以内。还记得上面的几次查询结果吗？在数据比较多的 shard 上的运行时间在 1 到 2 毫秒。可以看到总的性能得到了显著提高。
选择完美的 Shard Key
在 shard key 的选择方面，我们需要考虑很多因素，有些是技术的，有些是业务层面的。通常来讲应当注意下面几点：
所有增删改查语句都可以发送到集群中所有的 shard 服务器中
任何操作只需要发送到与其相关的 shard 服务器中，例如一次删除操作不应当发送到没有包括要删除的数据的 shard 服务器上
权衡利弊，实际上没有完美的 shard key，只有选择 shard key 时应当注意和考虑的要素。不会出现一种 shard key 可以满足所有的增删改查操作。你需要从给你的应用场景中抽象出用来选择 shard key 的元素，考量这些要素并作出最后选择，例如：你的应用是处理读操作多还是写操作多？最常用的写操作场景是什么样子的？


##mongodb分布式集群搭建手记 
https://www.cnblogs.com/wangshuyang/p/11728067.html

二、配置说明
端口通讯
当前集群中存在shard、config、mongos共12个进程节点，端口矩阵编排如下：
编号  实例类型
1   mongos
2   mongos
3   mongos
4   config
5   config
6   config
7   shard1
8   shard1
9   shard1
10  shard2
11  shard2
12  shard2
内部鉴权
节点间鉴权采用keyfile方式实现鉴权，mongos与分片之间、副本集节点之间共享同一套keyfile文件。 官方说明
账户设置
管理员账户：admin/Admin@01，具有集群及所有库的管理权限
应用账号：appuser/AppUser@01，具有appdb的owner权限
关于初始化权限
keyfile方式默认会开启鉴权，而针对初始化安装的场景，Mongodb提供了localhost-exception机制，
可以在首次安装时通过本机创建用户、角色，以及副本集初始操作。

三、准备工作
1. 下载安装包
官方地址：https://www.mongodb.com/download-center
wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-3.6.3.tgz

2. 部署目录
解压压缩文件，将bin目录拷贝到目标路径/opt/local/mongo-cluster，参考以下命令：
tar -xzvf mongodb-linux-x86_64-rhel70-3.6.3.tgz
mkdir -p  /opt/local/mongo-cluster
cp -r mongodb-linux-x86_64-rhel70-3.6.3/bin  /opt/local/mongo-cluster

3. 创建配置文件
cd /opt/local/mongo-cluster
mkdir conf 
A. mongod 配置文件 mongo_node.conf
mongo_node.conf 作为mongod实例共享的配置文件，内容如下：
storage:
    engine: wiredTiger
    directoryPerDB: true
    journal:
        enabled: true
systemLog:
    destination: file
    logAppend: true
operationProfiling:
  slowOpThresholdMs: 10000
replication:
    oplogSizeMB: 10240
processManagement:
    fork: true
net:
    http:
      enabled: false
security:
    authorization: "enabled"
选项说明可参考这里
B. mongos 配置文件 mongos.conf
systemLog:
    destination: file
    logAppend: true
processManagement:
    fork: true
net:
    http:
      enabled: false
4. 创建keyfile文件
cd /opt/local/mongo-cluster
mkdir keyfile
openssl rand -base64 756 > mongo.key
chmod 400 mongo.key
mv mongo.key keyfile
mongo.key 采用随机算法生成，用作节点内部通讯的密钥文件

5. 创建节点目录
WORK_DIR=/opt/local/mongo-cluster
mkdir -p $WORK_DIR/nodes/config/n1/data
mkdir -p $WORK_DIR/nodes/config/n2/data
mkdir -p $WORK_DIR/nodes/config/n3/data

mkdir -p $WORK_DIR/nodes/shard1/n1/data
mkdir -p $WORK_DIR/nodes/shard1/n2/data
mkdir -p $WORK_DIR/nodes/shard1/n3/data

mkdir -p $WORK_DIR/nodes/shard2/n1/data
mkdir -p $WORK_DIR/nodes/shard2/n2/data
mkdir -p $WORK_DIR/nodes/shard2/n3/data

mkdir -p $WORK_DIR/nodes/mongos/n1
mkdir -p $WORK_DIR/nodes/mongos/n2
mkdir -p $WORK_DIR/nodes/mongos/n3
以config 节点1 为例，nodes/config/n1/data是数据目录，而pid文件、日志文件都存放于n1目录
以mongos 节点1 为例，nodes/mongos/n1 存放了pid文件和日志文件

四、搭建集群
1. Config副本集
按以下脚本启动3个Config实例
WORK_DIR=/opt/local/mongo-cluster
KEYFILE=$WORK_DIR/keyfile/mongo.key
CONFFILE=$WORK_DIR/conf/mongo_node.conf
MONGOD=$WORK_DIR/bin/mongod

$MONGOD --port 26001 --configsvr --replSet configReplSet --keyFile $KEYFILE --dbpath $WORK_DIR/nodes/config/n1/data --pidfilepath $WORK_DIR/nodes/config/n1/db.pid --logpath $WORK_DIR/nodes/config/n1/db.log --config $CONFFILE

$MONGOD --port 26002 --configsvr --replSet configReplSet --keyFile $KEYFILE --dbpath $WORK_DIR/nodes/config/n2/data --pidfilepath $WORK_DIR/nodes/config/n2/db.pid --logpath $WORK_DIR/nodes/config/n2/db.log --config $CONFFILE

$MONGOD --port 26003 --configsvr --replSet configReplSet --keyFile $KEYFILE --dbpath $WORK_DIR/nodes/config/n3/data --pidfilepath $WORK_DIR/nodes/config/n3/db.pid --logpath $WORK_DIR/nodes/config/n3/db.log --config $CONFFILE
待成功启动后，输出日志如下：
about to fork child process, waiting until server is ready for connections.
forked process: 4976
child process started successfully, parent exiting
此时通过ps 命令也可以看到3个启动的进程实例。
连接其中一个Config进程，执行副本集初始化
./bin/mongo --port 26001 --host 127.0.0.1
> MongoDB server version: 3.4.7
> cfg={
    _id:"configReplSet", 
    configsvr: true,
    members:[
        {_id:0, host:'127.0.0.1:26001'},
        {_id:1, host:'127.0.0.1:26002'}, 
        {_id:2, host:'127.0.0.1:26003'}
    ]};
rs.initiate(cfg);
其中configsvr:true指明这是一个用于分片集群的Config副本集。
关于副本集配置可参考这里

2. 创建分片
按以下脚本启动Shard1的3个实例
WORK_DIR=/opt/local/mongo-cluster
KEYFILE=$WORK_DIR/keyfile/mongo.key
CONFFILE=$WORK_DIR/conf/mongo_node.conf
MONGOD=$WORK_DIR/bin/mongod

echo "start shard1 replicaset"

$MONGOD --port 27001 --shardsvr --replSet shard1 --keyFile $KEYFILE --dbpath $WORK_DIR/nodes/shard1/n1/data --pidfilepath $WORK_DIR/nodes/shard1/n1/db.pid --logpath $WORK_DIR/nodes/shard1/n1/db.log --config $CONFFILE
$MONGOD --port 27002 --shardsvr --replSet shard1 --keyFile $KEYFILE --dbpath $WORK_DIR/nodes/shard1/n2/data --pidfilepath $WORK_DIR/nodes/shard1/n2/db.pid --logpath $WORK_DIR/nodes/shard1/n2/db.log --config $CONFFILE
$MONGOD --port 27003 --shardsvr --replSet shard1 --keyFile $KEYFILE --dbpath $WORK_DIR/nodes/shard1/n3/data --pidfilepath $WORK_DIR/nodes/shard1/n3/db.pid --logpath $WORK_DIR/nodes/shard1/n3/db.log --config $CONFFILE
待成功启动后，输出日志如下：
about to fork child process, waiting until server is ready for connections.
forked process: 5976
child process started successfully, parent exiting
此时通过ps 命令也可以看到3个启动的Shard进程实例。
连接其中一个Shard进程，执行副本集初始化
./bin/mongo --port 27001 --host 127.0.0.1
> MongoDB server version: 3.4.7
> cfg={
    _id:"shard1", 
    members:[
        {_id:0, host:'127.0.0.1:27001'},
        {_id:1, host:'127.0.0.1:27002'}, 
        {_id:2, host:'127.0.0.1:27003'}
    ]};
rs.initiate(cfg);
参考以上步骤，启动Shard2的3个实例进程，并初始化副本集。

3. 启动mongos路由
执行以下脚本启动3个mongos进程
WORK_DIR=/opt/local/mongo-cluster
KEYFILE=$WORK_DIR/keyfile/mongo.key
CONFFILE=$WORK_DIR/conf/mongos.conf
MONGOS=$WORK_DIR/bin/mongos

echo "start mongos instances"
$MONGOS --port=25001 --configdb configReplSet/127.0.0.1:26001,127.0.0.1:26002,127.0.0.1:26003 --keyFile $KEYFILE --pidfilepath $WORK_DIR/nodes/mongos/n1/db.pid --logpath $WORK_DIR/nodes/mongos/n1/db.log --config $CONFFILE
$MONGOS --port 25002 --configdb configReplSet/127.0.0.1:26001,127.0.0.1:26002,127.0.0.1:26003 --keyFile $KEYFILE --pidfilepath $WORK_DIR/nodes/mongos/n2/db.pid --logpath $WORK_DIR/nodes/mongos/n2/db.log --config $CONFFILE
$MONGOS --port 25003 --configdb configReplSet/127.0.0.1:26001,127.0.0.1:26002,127.0.0.1:26003 --keyFile $KEYFILE --pidfilepath $WORK_DIR/nodes/mongos/n3/db.pid --logpath $WORK_DIR/nodes/mongos/n3/db.log --config $CONFFILE
待成功启动后，通过ps命令看到mongos进程：
dbuser      7903    1  0 17:49 ?        00:00:00 /opt/local/mongo-cluster/bin/mongos --port=25001 --configdb configReplSet/127.0.0.1:26001,127.0.0.1:26002,127.0.0.1:26003 --keyFile /opt/local/mongo-cluster/keyfile/mongo.key --pidfilepath /opt/local/mongo-cluster/nodes/mongos/n1/db.pid --logpath /opt/local/mongo-cluster/nodes/mongos/n1/db.log --config /opt/local/mongo-cluster/conf/mongos.conf
dbuser      7928    1  0 17:49 ?        00:00:00 /opt/local/mongo-cluster/bin/mongos --port 25002 --configdb configReplSet/127.0.0.1:26001,127.0.0.1:26002,127.0.0.1:26003 --keyFile /opt/local/mongo-cluster/keyfile/mongo.key --pidfilepath /opt/local/mongo-cluster/nodes/mongos/n2/db.pid --logpath /opt/local/mongo-cluster/nodes/mongos/n2/db.log --config /opt/local/mongo-cluster/conf/mongos.conf
dbuser      7954    1  0 17:49 ?        00:00:00 /opt/local/mongo-cluster/bin/mongos --port 25003 --configdb configReplSet/127.0.0.1:26001,127.0.0.1:26002,127.0.0.1:26003 --keyFile /opt/local/mongo-cluster/keyfile/mongo.key --pidfilepath /opt/local/mongo-cluster/nodes/mongos/n3/db.pid --logpath /opt/local/mongo-cluster/nodes/mongos/n3/db.log --config /opt/local/mongo-cluster/conf/mongos.conf
接入其中一个mongos实例，执行添加分片操作：
./bin/mongo --port 25001 --host 127.0.0.1
mongos> MongoDB server version: 3.4.7
mongos> sh.addShard("shard1/127.0.0.1:27001")
{ "shardAdded" : "shard1", "ok" : 1 }
mongos> sh.addShard("shard2/127.0.0.1:27004")
{ "shardAdded" : "shard2", "ok" : 1 }
至此，分布式集群架构启动完毕，但进一步操作需要先添加用户。

4. 初始化用户
接入其中一个mongos实例，添加管理员用户
use admin
db.createUser({
    user:'admin',pwd:'Admin@01',
    roles:[
        {role:'clusterAdmin',db:'admin'},
        {role:'userAdminAnyDatabase',db:'admin'},
        {role:'dbAdminAnyDatabase',db:'admin'},
        {role:'readWriteAnyDatabase',db:'admin'}
]})
当前admin用户具有集群管理权限、所有数据库的操作权限。
需要注意的是，在第一次创建用户之后，localexception不再有效，接下来的所有操作要求先通过鉴权。
use admin
db.auth('admin','Admin@01')

检查集群状态
mongos> sh.status()
--- Sharding Status --- 
  sharding version: {
    "_id" : 1,
    "minCompatibleVersion" : 5,
    "currentVersion" : 6,
    "clusterId" : ObjectId("5aa39c3e915210dc501a1dc8")
}
  shards:
    {  "_id" : "shard1",  "host" : "shard1/127.0.0.1:27001,127.0.0.1:27002,127.0.0.1:27003",  "state" : 1 }
    {  "_id" : "shard2",  "host" : "shard2/127.0.0.1:27004,127.0.0.1:27005,127.0.0.1:27006",  "state" : 1 }
  active mongoses:
    "3.4.7" : 3
autosplit:
    Currently enabled: yes
    
集群用户
分片集群中的访问都会通过mongos入口，而鉴权数据是存储在config副本集中的，即config实例中system.users数据库存储了集群用户及角色权限配置。mongos与shard实例则通过内部鉴权(keyfile机制)完成，因此shard实例上可以通过添加本地用户以方便操作管理。在一个副本集上，只需要在Primary节点上添加用户及权限，相关数据会自动同步到Secondary节点。
关于集群鉴权https://docs.mongodb.com/manual/core/security-users/?spm=a2c4e.11153940.blogcont617224.15.77552d7eSApnpI#sharded-cluster-users
在本案例中，我们为两个分片副本集都添加了本地admin用户。
通过mongostat工具可以显示集群所有角色：
          host insert query update delete getmore command dirty used flushes mapped vsize  res faults qrw arw net_in net_out conn    set repl                time
127.0.0.1:27001    *0    *0    *0    *0      0    6|0  0.1% 0.1%      0        1.49G 44.0M    n/a 0|0 0|0  429b  56.1k  25 shard1  PRI Mar 10 19:05:13.928
127.0.0.1:27002    *0    *0    *0    *0      0    7|0  0.1% 0.1%      0        1.43G 43.0M    n/a 0|0 0|0  605b  55.9k  15 shard1  SEC Mar 10 19:05:13.942
127.0.0.1:27003    *0    *0    *0    *0      0    7|0  0.1% 0.1%      0        1.43G 43.0M    n/a 0|0 0|0  605b  55.9k  15 shard1  SEC Mar 10 19:05:13.946
127.0.0.1:27004    *0    *0    *0    *0      0    6|0  0.1% 0.1%      0        1.48G 43.0M    n/a 0|0 0|0  546b  55.8k  18 shard2  PRI Mar 10 19:05:13.939
127.0.0.1:27005    *0    *0    *0    *0      0    6|0  0.1% 0.1%      0        1.43G 42.0M    n/a 0|0 0|0  540b  54.9k  15 shard2  SEC Mar 10 19:05:13.944
127.0.0.1:27006    *0    *0    *0    *0      0    6|0  0.1% 0.1%      0        1.46G 44.0M    n/a 0|0 0|0  540b  54.9k  17 shard2  SEC Mar 10 19:05:13.936    

五、数据操作
在案例中，创建appuser用户、为数据库实例appdb启动分片。
use appdb
db.createUser({user:'appuser',pwd:'AppUser@01',roles:[{role:'dbOwner',db:'appdb'}]})
sh.enableSharding("appdb")
创建集合book，为其执行分片初始化。
use appdb
db.createCollection("book")
db.device.ensureIndex({createTime:1})
sh.shardCollection("appdb.book", {bookId:"hashed"}, false, { numInitialChunks: 4} )
继续往device集合写入1000W条记录，观察chunks的分布情况

use appdb
var cnt = 0;
for(var i=0; i<1000; i++){
    var dl = [];
    for(var j=0; j<100; j++){
        dl.push({
                "bookId" : "BBK-" + i + "-" + j,
                "type" : "Revision",
                "version" : "IricSoneVB0001",
                "title" : "Jackson's Life",
                "subCount" : 10,
                "location" : "China CN Shenzhen Futian District",
                "author" : {
                      "name" : 50,
                      "email" : "RichardFoo@yahoo.com",
                      "gender" : "female"
                },
                "createTime" : new Date()
            });
      }
      cnt += dl.length;
      db.book.insertMany(dl);
      print("insert ", cnt);
}
执行db.book.getShardDistribution(),输出如下：
Shard shard1 at shard1/127.0.0.1:27001,127.0.0.1:27002,127.0.0.1:27003
data : 13.41MiB docs : 49905 chunks : 2
estimated data per chunk : 6.7MiB
estimated docs per chunk : 24952

Shard shard2 at shard2/127.0.0.1:27004,127.0.0.1:27005,127.0.0.1:27006
data : 13.46MiB docs : 50095 chunks : 2
estimated data per chunk : 6.73MiB
estimated docs per chunk : 25047

Totals
data : 26.87MiB docs : 100000 chunks : 4
Shard shard1 contains 49.9% data, 49.9% docs in cluster, avg obj size on shard : 281B
Shard shard2 contains 50.09% data, 50.09% docs in cluster, avg obj size on shard : 281B

##MongoDB的容量规划及硬件配置 
https://www.cnblogs.com/williamjie/p/9298031.html

##MongoDB分片（Sharding）技术
http://www.cecdns.com/post/45616.html

##使用哈希片键对集合分片
https://mongoing.com/docs/tutorial/shard-collection-with-a-hashed-shard-key.html
https://mongoing.com/docs/reference/glossary.html#term-chunk
>>>>>>> branch 'master' of https://github.com/doerjiayi/algorithm.git

##mongodb分片介绍—— 基于范围（数值型）的分片 或者 基于哈希的分片 
https://www.cnblogs.com/bonelee/p/6282142.html

mongodb分片介绍—— 基于范围（数值型）的分片 或者 基于哈希的分片 
数据分区
MongoDB中数据的分片是以集合为基本单位的,集合中的数据通过 片键 被分成多部分.
片键
对集合进行分片时,你需要选择一个 片键 , shard key 是每条记录都必须包含的,且建立了索引的单个字段或复合字段,MongoDB按照片键将数据划分到不同的 数据块 中,并将 数据块 均衡地分布到所有分片中.为了按照片键划分数据块,MongoDB使用 基于范围的分片方式 或者 基于哈希的分片方式。
以范围为基础的分片
对于 基于范围的分片 ,MongoDB按照片键的范围把数据分成不同部分.假设有一个数字的片键:想象一个从负无穷到正无穷的直线,每一个片键的值都在直线上画了一个点.MongoDB把这条直线划分为更短的不重叠的片段,并称之为 数据块 ,每个数据块包含了片键在一定范围内的数据.
在使用片键做范围划分的系统中,拥有”相近”片键的文档很可能存储在同一个数据块中,因此也会存储在同一个分片中.

基于哈希的分片
对于 基于哈希的分片 ,MongoDB计算一个字段的哈希值,并用这个哈希值来创建数据块.
在使用基于哈希分片的系统中,拥有”相近”片键的文档 很可能不会 存储在同一个数据块中,因此数据的分离性更好一些.

基于范围的分片方式与基于哈希的分片方式性能对比
基于范围的分片方式提供了更高效的范围查询,给定一个片键的范围,分发路由可以很简单地确定哪个数据块存储了请求需要的数据,并将请求转发到相应的分片中.
不过,基于范围的分片会导致数据在不同分片上的不均衡,有时候,带来的消极作用会大于查询性能的积极作用.比如,如果片键所在的字段是线性增长的,一定时间内的所有请求都会落到某个固定的数据块中,最终导致分布在同一个分片中.在这种情况下,一小部分分片承载了集群大部分的数据,系统并不能很好地进行扩展.
与此相比,基于哈希的分片方式以范围查询性能的损失为代价,保证了集群中数据的均衡.哈希值的随机性使数据随机分布在每个数据块中,因此也随机分布在不同分片中.但是也正由于随机性,一个范围查询很难确定应该请求哪些分片,通常为了返回需要的结果,需要请求所有分片.
均衡
The balancer is a background process that manages chunk migrations. The balancer can run from any of themongos instances in a cluster.
当集群中数据的不均衡发生时,均衡器会将数据块从数据块数目最多的分片迁移到数据块最少的分片上,举例来讲:如果集合 users 在 分片1 上有100个数据块,在 分片2 上有50个数据块,均衡器会将数据块从 分片1一直向 分片2 迁移,一直到数据均衡为止.
The shards manage chunk migrations as a background operation between an origin shard and a destination shard. During a chunk migration, the destination shard is sent all the current documents in the chunk from theorigin shard. Next, the destination shard captures and applies all changes made to the data during the migration process. Finally, the metadata regarding the location of the chunk on config server is updated.
If there’s an error during the migration, the balancer aborts the process leaving the chunk unchanged on the origin shard. MongoDB removes the chunk’s data from the origin shard after the migration completes successfully.

##MongoDB 分片集群技术

https://www.jianshu.com/p/8d9d3adb289d?from=timeline&isappinstalled=0

分片集群的构造
    （1）mongos ：数据路由，和客户端打交道的模块。mongos本身没有任何数据，他也不知道该怎么处理这数据，去找config server
（2）config server：所有存、取数据的方式，所有shard节点的信息，分片功能的一些配置信息。可以理解为真实数据的元数据。
（3）shard：真正的数据存储位置，以chunk为单位存数据。
Mongos本身并不持久化数据，Sharded cluster所有的元数据都会存储到Config Server，而用户的数据会议分散存储到各个shard。Mongos启动后，会从配置服务器加载元数据，开始提供服务，将用户的请求正确路由到对应的碎片。
Mongos的路由功能
　　当数据写入时，MongoDB Cluster根据分片键设计写入数据。
　　当外部语句发起数据查询时，MongoDB根据数据分布自动路由至指定节点返回数据。
2.2 集群中数据分布
2.2.1 Chunk是什么
　　在一个shard server内部，MongoDB还是会把数据分为chunks，每个chunk代表这个shard server内部一部分数据。chunk的产生，会有以下两个用途：
Splitting：当一个chunk的大小超过配置中的chunk size时，MongoDB的后台进程会把这个chunk切分成更小的chunk，从而避免chunk过大的情况
Balancing：在MongoDB中，balancer是一个后台进程，负责chunk的迁移，从而均衡各个shard server的负载，系统初始1个chunk，chunk size默认值64M,生产库上选择适合业务的chunk size是最好的。ongoDB会自动拆分和迁移chunks。
分片集群的数据分布（shard节点）
（1）使用chunk来存储数据
（2）进群搭建完成之后，默认开启一个chunk，大小是64M，
（3）存储需求超过64M，chunk会进行分裂，如果单位时间存储需求很大，设置更大的chunk
（4）chunk会被自动均衡迁移。
2.2.2 chunksize的选择
　　适合业务的chunksize是最好的。
　　chunk的分裂和迁移非常消耗IO资源；chunk分裂的时机：在插入和更新，读数据不会分裂。
chunksize的选择：
　　小的chunksize：数据均衡是迁移速度快，数据分布更均匀。数据分裂频繁，路由节点消耗更多资源。大的chunksize：数据分裂少。数据块移动集中消耗IO资源。通常100-200M
2.2.3 chunk分裂及迁移


随着数据的增长，其中的数据大小超过了配置的chunk size，默认是64M，则这个chunk就会分裂成两个。数据的增长会让chunk分裂得越来越多


这时候，各个shard 上的chunk数量就会不平衡。这时候，mongos中的一个组件balancer  就会执行自动平衡。把chunk从chunk数量最多的shard节点挪动到数量最少的节点。
chunkSize 对分裂及迁移的影响
　　MongoDB 默认的 chunkSize 为64MB，如无特殊需求，建议保持默认值；chunkSize 会直接影响到 chunk 分裂、迁移的行为。
　　chunkSize 越小，chunk 分裂及迁移越多，数据分布越均衡；反之，chunkSize 越大，chunk 分裂及迁移会更少，但可能导致数据分布不均。
　　chunkSize 太小，容易出现 jumbo chunk（即shardKey 的某个取值出现频率很高，这些文档只能放到一个 chunk 里，无法再分裂）而无法迁移；chunkSize 越大，则可能出现 chunk 内文档数太多（chunk 内文档数不能超过 250000 ）而无法迁移。
　　chunk 自动分裂只会在数据写入时触发，所以如果将 chunkSize 改小，系统需要一定的时间来将 chunk 分裂到指定的大小。
　　chunk 只会分裂，不会合并，所以即使将 chunkSize 改大，现有的 chunk 数量不会减少，但 chunk 大小会随着写入不断增长，直到达到目标大小。
2.3 数据区分
2.3.1 分片键shard key
　　MongoDB中数据的分片是、以集合为基本单位的，集合中的数据通过片键（Shard key）被分成多部分。其实片键就是在集合中选一个键，用该键的值作为数据拆分的依据。
　　所以一个好的片键对分片至关重要。片键必须是一个索引，通过sh.shardCollection加会自动创建索引（前提是此集合不存在的情况下）。一个自增的片键对写入和数据均匀分布就不是很好，因为自增的片键总会在一个分片上写入，后续达到某个阀值可能会写到别的分片。但是按照片键查询会非常高效。
　　随机片键对数据的均匀分布效果很好。注意尽量避免在多个分片上进行查询。在所有分片上查询，mongos会对结果进行归并排序。
　　对集合进行分片时，你需要选择一个片键，片键是每条记录都必须包含的，且建立了索引的单个字段或复合字段，MongoDB按照片键将数据划分到不同的数据块中，并将数据块均衡地分布到所有分片中。
　　为了按照片键划分数据块，MongoDB使用基于范围的分片方式或者 基于哈希的分片方式。
注意：
        分片键是不可变。
        分片键必须有索引。
        分片键大小限制512bytes。
        分片键用于路由查询。
        MongoDB不接受已进行collection级分片的collection上插入无分片
        键的文档（也不支持空值插入）


##MongoDB中_id(ObjectId)生成 
https://www.cnblogs.com/btgyoyo/p/7156589.html
MongoDB 中我们经常会接触到一个自动生成的字段："_id"，类型为ObjectId。
之前我们使用MySQL等关系型数据库时，主键都是设置成自增的。但在分布式环境下，这种方法就不可行了，会产生冲突。为此，mongodb采用了一个称之为ObjectId的类型来做主键。ObjectId是一个12字节的 BSON 类型字符串。按照字节顺序，一次代表：
4字节：UNIX时间戳 
3字节：表示运行MongoDB的机器 
2字节：表示生成此_id的进程 
3字节：由一个随机数开始的计数器生成的值 
从ObjectId的构造上来看，内部就嵌入了时间类型。我们肯定可以从中获取时间信息：即插入此文档时的时间
a = new ObjectId() 
ObjectId(“53102b43bf1044ed8b0ba36b”) 
a.getTimestamp() 
ISODate(“2014-02-28T06:22:59Z”) 
根据时间构造ObjectId
 
上例是直接使用MongoDB提供的新建方法来构造ObjectId的，我们自己可不可以通过字符串来构造呢？看下例：
// 使用Date的字符串构造方法生成日期 
// 然后使用Date对象的getTime获取毫秒数，再除以1000得到标准时间戳
a = new Date(“2012-12-12 00:00:00”).getTime()/1000 
1355241600 
// 获取时间戳的标准十六进制表示 
a = a.toString(16) 
50c75880
 MongoDB默认在ObjectId上建立索引，是按照插入时间排序的。我们可以使用此索引进行查询和排序。

db.col.insert({“num”:1}) 
db.col.insert({“num”:2}) 
db.col.insert({“num”:3}) 
db.col.find().pretty() 
{ “_id” : ObjectId(“53102fb4bf1044ed8b0ba36c”), “num” : 1 } 
{ “_id” : ObjectId(“53102fb9bf1044ed8b0ba36d”), “num” : 2 } 
{ “_id” : ObjectId(“53102fbabf1044ed8b0ba36e”), “num” : 3 }

 
// 按照_id升序，即按照插入时间升序
db.col.find().sort({“_id”:1}).pretty() 
{ “_id” : ObjectId(“53102fb4bf1044ed8b0ba36c”), “num” : 1 } 
{ “_id” : ObjectId(“53102fb9bf1044ed8b0ba36d”), “num” : 2 } 
{ “_id” : ObjectId(“53102fbabf1044ed8b0ba36e”), “num” : 3 }
 
// 按照_id降序，即按照插入时间降序
db.col.find().sort({“_id”:-1}).pretty() 
{ “_id” : ObjectId(“53102fbabf1044ed8b0ba36e”), “num” : 3 } 
{ “_id” : ObjectId(“53102fb9bf1044ed8b0ba36d”), “num” : 2 } 
{ “_id” : ObjectId(“53102fb4bf1044ed8b0ba36c”), “num” : 1 }
 
// 抽取num = 2的ObjectId用来过滤
num2 = ObjectId(“53102fb9bf1044ed8b0ba36d”) 
ObjectId(“53102fb9bf1044ed8b0ba36d”)
 
// 找出插入时间在num2之后的数据
db.col.find({ “_id”:{$gt:num2}}).pretty() 
{ “_id” : ObjectId(“53102fbabf1044ed8b0ba36e”), “num” : 3 }

##Mongodb 与 MySQL对比 
https://www.cnblogs.com/web-fusheng/p/6884759.html
无论是MongoDB还是MySQL，都存在着主键的定义。
对于MongoDB来说，其主键名叫”_id”，在生成数据的时候，如果用户不主动为其分配一个主键的话，MongoDB会自动为其生成一个随机分配的值。
在MySQL中，主键的指定是在MySQL插入数据时指明PRIMARY KEY来定义的。当没有指定主键的时候，另一种工具 —— 索引，相当于替代了主键的功能。索引可以为空，也可以有重复，另外有一种不允许重复的索引叫惟一索引。如果既没有指定主键也没有指定索引的话，MySQL会自动为数据创建一个。
 
1.       数据库的平均插入速率：MongoDB不指定_id插入 > MySQL不指定主键插入 > MySQL指定主键插入 > MongoDB指定_id插入。
2.       MongoDB在指定_id与不指定_id插入时速度相差很大，而MySQL的差别却小很多。
 
分析：
1.         在指定_id或主键时，两种数据库在插入时要对索引值进行处理，并查找数据库中是否存在相同的键值，这会减慢插入的速率。
2.         在MongoDB中，指定索引插入比不指定慢很多，这是因为，MongoDB里每一条数据的_id值都是唯一的。当在不指定_id插入数据的时候，其_id是系统自动计算生成的。MongoDB通过计算机特征值、时间、进程ID与随机数来确保生成的_id是唯一的。而在指定_id插入时，MongoDB每插一条数据，都需要检查此_id可不可用，当数据库中数据条数太多的时候，这一步的查询开销会拖慢整个数据库的插入速度。
3.         MongoDB会充分使用系统内存作为缓存，这是一种非常优秀的特性。我们的测试机的内存有64G，在插入时，MongoDB会尽可能地在内存快写不进去数据之后，再将数据持久化保存到硬盘上。这也是在不指定_id插入的时候，MongoDB的效率遥遥领先的原因。但在指定_id插入时，当数据量一大内存装不下时，MongoDB就需要将磁盘中的信息读取到内存中来查重，这样一来其插入效率反而慢了。
4.         MySQL不愧是一种非常稳定的数据库，无论在指定主键还是在不指定主键插入的情况下，其效率都差不了太多。
 
插入稳定性分析
插入稳定性是指，随着数据量的增大，每插入一定量数据时的插入速率情况。
在本次测试中，我们把这个指标的规模定在10w，即显示的数据是在每插入10w条数据时，在这段时间内每秒钟能插入多少条数据。
先呈现四张图上来：
 
1.       MongoDB指定_id插入：
 
 
2.       MongoDB不指定_id插入：

 
3.       MySQL指定PRIMARY KEY插入：

 
4.       MySQL不指定PRIMARY KEY插入：

 
总结：
1.       整体上的插入速度还是和上一回的统计数据类似：MongoDB不指定_id插入 > MySQL不指定主键插入 > MySQL指定主键插入 > MongoDB指定_id插入。
2.       从图中可以看出，在指定主键插入数据的时候，MySQL与MongoDB在不同数据数量级时，每秒插入的数据每隔一段时间就会有一个波动，在图表中显示成为规律的毛刺现象。而在不指定插入数据时，在大多数情况下插入速率都比较平均，但随着数据库中数据的增多，插入的效率在某一时段有瞬间下降，随即又会变稳定。
3.       整体上来看，MongoDB的速率波动比MySQL的严重，方差变化较大。
4.       MongoDB在指定_id插入时，当插入的数据变多之后，插入效率有明显地下降。在其他三种的插入测试中，从开始到结束，其插入的速率在大多数的时候都固定在一个标准上。
 
分析：
1.       毛刺现象是因为，当插入的数据太多的时候，MongoDB需要将内存中的数据写进硬盘，MySQL需要重新分表。这些操作每当数据库中的数据达到一定量级后就会自动进行，因此每隔一段时间就会有一个明显的毛刺。
2.       MongoDB毕竟还是新生事物，其稳定性没有已应用多年的MySQL优秀。
3.       MongoDB在指定_id插入的时候，其性能的下降还是很厉害的。
1.       在读取的数据规模不大时，MongoDB的查询速度真是一骑绝尘，甩开MySQL好远好远。
2.       在查询的数据量逐渐增多的时候，MySQL的查询速度是稳步下降的，而MongoDB的查询速度却有些起伏。
 
分析：
1.       如果MySQL没有经过查询优化的话，其查询速度就不要跟MongoDB比了。MongoDB可以充分利用系统的内存资源，我们的测试机器内存是64GB的，内存越大MongoDB的查询速度就越快，毕竟磁盘与内存的I/O效率不是一个量级的。
2.       本次实验的查询的数据也是随机生成的，因此所有待查询的数据都存在MongoDB的内存缓存中的概率是很小的。在查询时，MongoDB需要多次将内存中的数据与磁盘进行交互以便查找，因此其查询速率取决于其交互的次数。这样就存在这样一种可能性，尽管待查询的数据数目较多，但这段随机生成的数据被MongoDB以较少的次数从磁盘中取出。因此，其查询的平均速度反而更快一些。这样看来，MongoDB的查询速度波动也处在一个合理的范围内。
3.       MySQL的稳定性还是毋庸置疑的。
 
结论
1. 相比较MySQL，MongoDB数据库更适合那些读作业较重的任务模型。MongoDB能充分利用机器的内存资源。如果机器的内存资源丰富的话，MongoDB的查询效率会快很多。
2. 在带”_id”插入数据的时候，MongoDB的插入效率其实并不高。如果想充分利用MongoDB性能的话，推荐采取不带”_id”的插入方式，然后对相关字段作索引来查询。
1. MongoDB适合那些对数据库具体数据格式不明确或者数据库数据格式经常变化的需求模型，而且对开发者十分友好。
2. MongoDB官方就自带一个分布式文件系统，可以很方便地部署到服务器机群上。MongoDB里有一个Shard的概念，就是方便为了服务器分片使用的。每增加一台Shard，MongoDB的插入性能也会以接近倍数的方式增长，磁盘容量也很可以很方便地扩充。
3. MongoDB还自带了对map-reduce运算框架的支持，这也很方便进行数据的统计。
 
MongoDB的缺陷
1. 事务关系支持薄弱。这也是所有NoSQL数据库共同的缺陷，不过NoSQL并不是为了事务关系而设计的，具体应用还是很需求。
2. 稳定性有些欠缺，这点从上面的测试便可以看出。
3. MongoDB一方面在方便开发者的同时，另一方面对运维人员却提出了相当多的要求。业界并没有成熟的MongoDB运维经验，MongoDB中数据的存放格式也很随意，等等问题都对运维人员的考验。

##详解mongodb——架构模式、持久化原理和数据文件存储原理 值得收藏
https://www.cnblogs.com/web-fusheng/p/6884759.html

架构模式
Replica set：复制集，mongodb的架构方式之一 ，通常是三个对等的节点构成一个“复制集”集群，有“primary”和secondary等多中角色（稍后详细介绍），其中primary负责读写请求，secondary可以负责读请求，这有配置决定，其中secondary紧跟primary并应用write操作；如果primay失效，则集群进行“多数派”选举，选举出新的primary，即failover机制，即HA架构。复制集解决了单点故障问题，也是mongodb垂直扩展的最小部署单位，当然sharding cluster中每个shard节点也可以使用Replica set提高数据可用性。

打开今日头条，查看更多精彩图片
Sharding cluster：分片集群，数据水平扩展的手段之一；replica set这种架构的缺点就是“集群数据容量”受限于单个节点的磁盘大小，如果数据量不断增加，对它进行扩容将时非常苦难的事情，所以我们需要采用Sharding模式来解决这个问题。将整个collection的数据将根据sharding key被sharding到多个mongod节点上，即每个节点持有collection的一部分数据，这个集群持有全部数据，原则上sharding可以支撑数TB的数据。

系统配置：
1）建议mongodb部署在linux系统上，较高版本，选择合适的底层文件系统（ext4），开启合适的swap空间
2）无论是MMAPV1或者wiredTiger引擎，较大的内存总能带来直接收益。
3）对数据存储文件关闭“atime”（文件每次access都会更改这个时间值，表示文件最近被访问的时间），可以提升文件访问效率。
4）ulimit参数调整，这个在基于网络IO或者磁盘IO操作的应用中，通常都会调整，上调系统允许打开的文件个数（ulimit -n 65535）。

持久化原理
持久化为了保证数据永久保存不丢失。MongoDB具有高度可配置的持久化设置，从完全没有任何保证到完全持久化。
mongodb与mysql不同，mysql的每一次更新操作都会直接写入硬盘，但是mongo不会，做为内存型数据库，数据操作会先写入内存，然后再会持久化到硬盘中去，那么mongo是如何持久化的呢
mongodb在启动时，专门初始化一个线程不断循环（除非应用crash掉），用于在一定时间周期内来从defer队列中获取要持久化的数据并写入到磁盘的journal(日志)和mongofile(数据)处，当然因为它不是在用户添加记录时就写到磁盘上，所以按mongodb开发者说，它不会造成性能上的损耗，因为看过代码发现，当进行CUD操作时，记录(Record类型)都被放入到defer队列中以供延时批量（groupcommit）提交写入，但相信其中时间周期参数是个要认真考量的参数，系统为90毫秒，如果该值更低的话，可能会造成频繁磁盘操作，过高又会造成系统宕机时数据丢失过。
数据文件存储原理（Data Files storage，MMAPV1引擎）
1、Data Files
mongodb的数据将会保存在底层文件系统中，比如我们dbpath设定为“/data/db”目录，我们创建一个database为“test”，collection为“sample”，然后在此collection中插入数条documents。我们查看dbpath下生成的文件列表

可以看到test这个数据库目前已经有6个数据文件（data files），每个文件以“database”的名字 + 序列数字组成，序列号从0开始，逐个递增，数据文件从16M开始，每次扩张一倍（16M、32M、64M、128M...），在默认情况下单个data file的最大尺寸为2G，如果设置了smallFiles属性（配置文件中）则最大限定为512M；mongodb中每个database最多支持16000个数据文件，即约32T，如果设置了smallFiles则单个database的最大数据量为8T。
2、Namespace文件
对于namespace文件，比如“test.ns”文件，默认大小为16M，此文件中主要用于保存“collection”、index的命名信息，比如collection的“属性”信息、每个索引的属性类型等，如果你的database中需要存储大量的collection（比如每一小时生成一个collection，在数据分析应用中），那么我们可以通过配置文件“nsSize”选项来指定
3、journal文件
journal日志为mongodb提供了数据保障能力，它本质上与mysql binlog没有太大区别，用于当mongodb异常crash后，重启时进行数据恢复；这归结于mongodb的数据持久写入磁盘是滞后的。默认情况下，“journal”特性是开启的，特别在production环境中，我们没有理由来关闭它。（除非，数据丢失对应用而言，是无关紧要的）
一个mongodb实例中所有的databases共享journal文件。
对于write操作而言，首先写入journal日志，然后将数据在内存中修改（mmap），此后后台线程间歇性的将内存中变更的数据flush到底层的data files中，时间间隔为60秒（参见配置项“syncPeriodSecs”）；write操作在journal文件中是有序的，为了提升性能，write将会首先写入journal日志的内存buffer中，当buffer数据达到100M或者每隔100毫秒，buffer中的数据将会flush到磁盘中的journal文件中；如果mongodb异常退出，将可能导致最多100M数据或者最近100ms内的数据丢失，flush磁盘的时间间隔有配置项“commitIntervalMs”决定，默认为100毫秒。mongodb之所以不能对每个write都将journal同步磁盘，这也是对性能的考虑，mysql的binlog也采用了类似的权衡方式。开启journal日志功能，将会导致write性能有所降低，可能降低5~30%，因为它直接加剧了磁盘的写入负载，我们可以将journal日志单独放置在其他磁盘驱动器中来提高写入并发能力（与data files分别使用不同的磁盘驱动器）。
如果你希望数据尽可能的不丢失，可以考虑：
1）减小commitIntervalMs的值
2）每个write指定“write concern”中指定“j”参数为true
3）最佳手段就是采用“replica set”架构模式，通过数据备份方式解决，同时还需要在“write concern”中指定“w”选项，且保障级别不低于“majority”。

mongodb现在应用很广，还是很有用的，例如我们生产环境就是用5台服务器专门去做mongodb分片集群，对mongodb有兴趣的朋友可以多了解下这方面。


##mongodb系列之-解读journal
https://blog.csdn.net/t594362122/article/details/52813272

   mongodb的journal，简单来说就是用于数据故障恢复和持久化数据的，它以日志方式来记录。从1.8版本开始有此功能，2.0开始默认打开此功能，但32位的系统是默认关闭的。
    journal除了故障恢复的作用之外，还可以提高写入的性能，批量提交（batch-commit），journal一般默认100ms刷新一次，在这个过程中，所有的写入都可以一次提交，是单事务的，全部成功或者全部失败，刷新时间，可以更改，范围是2-300ms。
       当系统非正常情况下突然挂掉，再次启动时候mongodb就会从journal日志中恢复数据，而确保数据不丢失，最多丢失ms级别的数据（个人臆想）

  使用db.serverStatus（）可以查看状态：
dur:{
		commits:30,
		journaledMB:0.270336,
		writeToDataFilesMB:0.329578,
		compression:0.7986268873651776,
		commotsInWriteLock:0,
		earlyCommits:0,
		timeMs:{
			dt:3077,
			prepLogBuffer:0,
			writeToJournal:4,
			writeToDataFiles:3,
			remapPrivateView:3
		}
}
 
dur.timeMS.prepLogBuffer：从privateView映射到Logbuffer的时间。
dur.timeMS.writeToJournal：从logbuffer刷新到journalfile 的时间。
dur.timeMS.writeToDataFiles：从journalbuffer映射到MMF，然后从MMF刷新到磁盘的时间，文件系统和磁盘会影响写入性能。 
dur.timeMS.remapPrivateView：重新映射数据到PrivateView的时间，越小性能越好。
所以说开启journal后会使用更多内存，因为journal会另外使用一块内存区域（即：PrivateView）

## mongodb复杂条件查询 (or与and)
https://blog.csdn.net/tjbsl/article/details/80620303 

要实现管理数据的如下SQL形式：
关系数据库：select * from  where（state1=11 and state2=22） or  value >300
首先使用MongoDB的方式查询：
分为以下几个步骤实现：
步骤一：实现 （state1=11 and state2=22）
db.getCollection('testOrAnd').find(
    {$and:[{"state1":11},{"state2":22}]}
    )
步骤二：使用or形式实现 value >300
db.getCollection('testOrAnd').  find(
    { $or:[{"value":{$gte:300}}] }
    )
步骤三：将步骤一参数拼接到步骤二or条件
db.getCollection('testOrAnd').
    find({$or:
           [
            {$and:[{"state1":11},{"state2":22}]},{"value":{$gte:300}}
           ]
         })

最终实现结果


##MongoDB安装部署及操作 
https://www.cnblogs.com/jiangyatao/p/11156390.html
用户授权认证
use admin
db.createUser(
	{
		user: "admin",
		pwd: "123456",
		roles:[ { role: "root", db:"admin"}]}
)
--------------role里的角色可以选：
              Built-In Roles（内置角色）：
             .1. 数据库用户角色：read、readWrite;
              .2. 数据库管理角色：dbAdmin、dbOwner、userAdmin；
              .3. 集群管理角色：clusterAdmin、clusterManager、clusterMonitor、hostManager；
              .4. 备份恢复角色：backup、restore；
              .5. 所有数据库角色：readAnyDatabase、readWriteAnyDatabase、userAdminAnyDatabase、dbAdminAnyDatabase
              .6. 超级用户角色：root 
                // 这里还有几个角色间接或直接提供了系统超级用户的访问（dbOwner 、userAdmin、userAdminAnyDatabase）
              .7. 内部角色：__system
              具体角色： 
                Read：允许用户读取指定数据库
                readWrite：允许用户读写指定数据库
                backup,retore:在进行备份、恢复时可以单独指定的角色，在db.createUser()方法中roles里面的db必须写成是admin库，要不然会 报错
                dbAdmin：允许用户在指定数据库中执行管理函数，如索引创建、删除，查看统计或访问system.profile
                userAdmin：允许用户向system.users集合写入，可以找指定数据库里创建、删除和管理用户
                clusterAdmin：只在admin数据库中可用，赋予用户所有分片和复制集相关函数的管理权限。
                readAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读权限
                readWriteAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读写权限
                userAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的userAdmin权限，
                dbAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的dbAdmin权限。
                root：只在admin数据库中可用。超级账号，超级权限
修改配置文件
vim /opt/mongo_cluster/mongo_27017/conf/mongodb.conf   
 添加
security:
  authorization: enabled

重启服务
mongod -f /opt/mongo_cluster/mongo_27017/conf/mongodb.conf --shutdown
mongod -f /opt/mongo_cluster/mongo_27017/conf/mongodb.conf

登录
mongo
show dbs 
mongo -uadmin -p 
show dbs


##10分钟完成MongoDB的容量规划及硬件配置
https://mongoing.com/archives/878

##MongoDB实战-分片概念和原理
https://blog.csdn.net/wanght89/article/details/77842336

分片一个集合
   MongoDB的分片是基于范围的。也就是说分片集合里的每个文档都必须落在指定键的某个值范围里。MongoDB使用所谓的分片键（shard key）让每个文档在这些范围里找到自己的位置。（其他的分布式数据库里可能使用分区键partition key或分布键 distribution key来代替分片键这个术语）从假想的电子表格管理应用程序里拿出一个实例文档，这样能更好地理解分片键：

{
_id:ObjectId("4d6e9b89b600c2c196442c21")
filename:"spreadsheet-1"
updated_at:ISODate("2017-09-04T19:22:54.845z")
username:"banks"
data:"raw documnet data"
}

    在对该集合进行分片时，必须将其中的一个或多个字段申明为分片键。如果选择_id，那么文档会基于对象ID的范围进行分布。但是，处于一些原因，你要基于username和_id声明一个复合分片键：因此，这些范围通常会表示Wie一系列用户名
   现在你需要理解块（chunk）的概念，它是位于一个分片中的一段连续的分片键范围。举例来说，可以假设docs集合分布在两个分片A和B上，它被分成下表所示的多个块。每个块的范围都由起始值和终止值来标识。

起始值	终止值	分片
-无穷大	abbot	B
abbot	dayton	A
dayton	harris	B
harris	norris	A
norris	无穷大	B

     粗略扫视上表后，你会发现一个重要的、有些违反直觉的属性：虽然每个单独的块都表示一段连续范围的数据，但这些块能出现在任意分片上。关于块，第二个要点是它们是种逻辑上的东西，而非物理上的。换言之，块并不表示磁盘上连续的文档。从一定程度上来说，如果一个从harris开始到norris结束的块存在于分片A上，那么就认为可以在分片A的docs集合里找到分片键落在这个范围内的文档。这个集合里那些文档的排列没有任何必然关系。

拆分与迁移
     分片机制的重点是块的拆分（spliting）与迁移（migration）

      首先，考虑一下块拆分的思想。在初始化分片集群时，只存在一个块，这个块的范围涵盖了整个分片集合。那该如何发展到有多个块的分片集群呢？答案就是块大小达到某个阈值是就会对块进行拆分。默认的块的最大块尺寸时64MB或者100000个文档，先达到哪个标准就以哪个标准为准。在向新的分片集群添加数据时，原始的块最终会达到某个阈值，触发块的拆分。这是一个简单的操作，基本就是把原来的范围一分为二，这样就有两个块，每个块都有相同数量的文档。

     请注意，块的拆分是个逻辑操作。当MongoDB进行块拆分时，它只是修改块的元数据就能让一个块变为两个。因此，拆分一个块并不影响分片集合里文档的物理顺序。也就是说拆分既简单又快捷。 

     你可以回想一下，设计分片系统时最大的一个困难就是保证数据始终均匀分布。MongoDB的分片集群是通过在分片中移动块来实现均衡的。我们称之为迁移，这是一个真实的物理操作。

    迁移是由名为均衡器（balancer）的软件进程管理的，它的任务就是确保数据在各个分片中保持均匀变化。通过追踪各分片上块的数量，就能实现这个功能。虽然均衡的触发会随总数据量的不同而变化，但是通常来说，当集群中拥有块最多的分片与拥有块最少的分片的块数相差大于8时，均衡器就会发起一次均衡处理。在均衡过程中，块会从块较多的分片迁移到块较少非分片上，直到两个分片的块数大致相等为止。

