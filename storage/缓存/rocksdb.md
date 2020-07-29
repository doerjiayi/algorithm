
##RocksDB 介绍
RocksDB是facebook开源的NOSQL存储系统，其设计是基于Google开源的LevelDB，优化了LevelDB中存在的一些问题，其性能要比LevelDB强，设计与LevelDB极其类似。
LevelDB的开源发起者：Jeff Dean和Sanjay Ghemawat，这两位是Google公司重量级的工程师。
Jeff Dean是Google大规模分布式平台Bigtable和MapReduce主要设计和实现者。
Sanjay Ghemawat是Google大规模分布式平台GFS，Bigtable和MapReduce主要设计和实现工程师。
如果了解Bigtable的话，应该知道在这个影响深远的分布式存储系统中有两个核心的部分：Master Server和Tablet Server。其中Master Server做一些管理数据的存储以及分布式调度工作，实际的分布式数据存储以及读写操作是由Tablet Server完成的，而LevelDB则可以理解为一个简化版的Tablet Server。
RocksDB相对传统的关系数据库的一大改进是采用LSM树存储引擎。LSM树是非常有创意的一种数据结构，它和传统的B+树不太一样，下面先说说B+树。

###B+树

B+树根节点和枝节点分别记录每个叶子节点的最小值，并用一个指针指向叶子节点。叶子节点里每个键值都指向真正的数据块，每个叶子节点都有前指针和后指针，这是为了做范围查询时，叶子节点间可以直接跳转，从而避免再去回溯至枝和跟节点。
B+树最大的性能问题是会产生大量的随机IO，随着新数据的插入，叶子节点会慢慢分裂，逻辑上连续的叶子节点在物理上往往不连续，甚至分离的很远，但做范围查询时，会产生大量读随机IO。
对于大量的随机写也一样，举一个插入key跨度很大的例子，如7->1000->3->2000 … 新插入的数据存储在磁盘上相隔很远，会产生大量的随机写IO，低下的磁盘寻道速度严重影响性能。

###LSM树（Log-Structured Merge Tree）
LSM树而且通过批量存储技术规避磁盘随机写入问题。 LSM树的设计思想非常朴素, 它的原理是把一颗大树拆分成N棵小树， 它首先写入到内存中（内存没有寻道速度的问题，随机写的性能得到大幅提升），在内存中构建一颗有序小树，随着小树越来越大，内存的小树会flush到磁盘上。磁盘中的树定期可以做merge操作，合并成一棵大树，以优化读性能。


###LevelDB特点

LevelDB是一个持久化存储的KV系统，和Redis这种内存型的KV系统不同，LevelDB不会像Redis一样狂吃内存，而是将大部分数据存储到磁盘上。
LevleDb在存储数据时，是根据记录的key值有序存储的，就是说相邻的key值在存储文件中是依次顺序存储的，而应用可以自定义key大小比较函数。
LevelDB支持数据快照（snapshot）功能，使得读取操作不受写操作影响，可以在读操作过程中始终看到一致的数据。
LevelDB还支持数据压缩等操作，这对于减小存储空间以及增快IO效率都有直接的帮助。


###RocksDB对LevelDB的优化

增加了column family，这样有利于多个不相关的数据集存储在同一个db中，因为不同column family的数据是存储在不同的sst和memtable中，所以一定程度上起到了隔离的作用。
采用了多线程同时进行compaction的方法，优化了compact的速度。
增加了merge operator，优化了modify的效率
将flush和compaction分开不同的线程池，能有效的加快flush，防止stall。
增加了对write ahead log(WAL)的特殊管理机制，这样就能方便管理WAL文件，因为WAL是binlog文件。
RocksDB的整体结构：

rocksdb从3.0开始支持ColumnFamily的概念。每个columnfamilyl的meltable与sstable都是分开的，所以每一个column family都可以单独配置，所有column family共用同一个WA文件，可以保证跨column family写入时的原子性。

###RocksDB 写入与删除

写操作包含两个具体步骤：

首先是将这条KV记录以顺序写的方式追加到log文件末尾，因为尽管这是一个磁盘读写操作，但是文件的顺序追加写入效率是很高的，所以并不会导致写入速度的降低。
第二个步骤是:如果写入log文件成功，那么将这条KV记录插入内存中的Memtable中，Memtable只是一层封装，其内部其实是一个Key有序的SkipList列表，插入一条新记录的过程也很简单，即先查找合适的插入位置，然后修改相应的链接指针将新记录插入即可。完成这一步，写入记录就算完成了，所以一个插入记录操作涉及一次磁盘文件追加写和内存SkipList插入操作，这是为何RocksDB写入速度如此高效的根本原因。
删除操作与插入操作相同，区别是，插入操作插入的是Key:Value值，而删除操作插入的是“Key:删除标记”，并不真正去删除记录，而是后台Compaction的时候才去做真正的删除操作。

###RocksDB 读取记录

RocksDB首先会去查看内存中的Memtable，如果Memtable中包含key及其对应的value，则返回value值即可；如果在Memtable没有读到key，则接下来到同样处于内存中的Immutable Memtable中去读取，类似地，如果读到就返回，若是没有读到,那么会从磁盘中的SSTable文件中查找。
总的读取原则是这样的：首先从属于level 0的文件中查找，如果找到则返回对应的value值，如果没有找到那么到level 1中的文件中去找，如此循环往复，直到在某层SSTable文件中找到这个key对应的value为止（或者查到最高level，查找失败，说明整个系统中不存在这个Key)。
相对写操作，读操作处理起来要复杂很多。RocksDB为了提高读取速递，增加了读cache和Bloomfilter。

##RocksDB事务实现TransactionDB分析
基本概念
1. LSN (log sequence number)
RocksDB中的每一条记录(KeyValue)都有一个LogSequenceNumber(后面统称lsn)，从最初的0开始，每次写入加1。该值为逻辑量，区别于InnoDB的lsn为redo log物理写入字节量。

这个lsn在RocksDB内部的memtable中是单调递增的，在WriteAheadLog(WAL)中以WriteBatch为单位递增(count(batch.records)为单位)。

WriteBatch是一次RocksDB::Put()的原子操作集合，不同的WriteBatch间是遵循ACID特性(要么完全成功要么完全失败，并且相互隔离)，结构如下：

 WriteBatch :=
    sequence: fixed64
    count: fixed32
    data: record[count]
从RocksDB外部能看到的LSN是按WriteBatch递增的(LeaderWriter(或LastWriter)最后一次性更新)，所以进行snapshot读时，使用的就是此lsn。

注意: 在WAL中每条WriteBatch的lsn并不严格满足以下公式(比如2pc情况下):

lsn(WriteBatch[n]) < lsn(WriteBatch[n+1])，可能相等

2. Snapshot
Snapshot是RocksDB的快照，实际存储的就是一个lsn.

class SnapshotImpl {
 public:
  // 当前的lsn
  SequenceNumber number_;       
 private:
  SnapshotImpl* prev_;
  SnapshotImpl* next_;
  SnapshotList* list_; 
  // unix时间戳
  int64_t unix_time_;           
  // 是否属于Transaction(用于写冲突)
  bool is_write_conflict_boundary_;
};
查询时如果设置了snapshot为某个lsn, 那么对于此snapshot的读来说，只能看到lsn(key)<=lsn(snapshot)的key，大于该lsn的key是不可见的。

snapshot的创建和删除都需要由一个全局的DoubleLinkList (DBImpl::SnapshotList)管理，天然的根据创建时间(同样也是lsn大小)的关系排序，使用之后需要通过DBImpl::ReleaseSnapshot释放。snapshot还用于在RocksDB事务中实现不同的隔离级别。

3. 隔离级别
为了实现事务下的一致性非锁定读(读可以并发)，不同的数据库(引擎)实现了不同的读隔离级别。SQL规范标准中定义了如下四种：

 	ReadUncommited	ReadCommited	RepeatableRead	Serializable
Oracle	No	Yes	No	Yes
MySQL	Yes	Yes	Yes	Yes
RocksDB	No	Yes	Yes	No

ReadUncommitted 读取未提交内容，所有事务都可以看到其他未提交事务的执行结果。存在脏读。

ReadCommitted读取已提交内容，事务只能看见其他已经提交事务所做的改变，多次读取同一个记录可能包含其他事务已提交的更新。

RepeatableRead 可重读，确保事务读取数据时，多次操作会看到同样的数据行(InnoDB通过NextKeyLocking对btree索引加锁解决了幻读)。

Serializable串行化，强制事务之间进行排序，不会互相冲突。


大部分数据库(如MySQL InnoDB、RocksDB)，通过MVCC都可以实现上述的在非排它锁锁定情况下的多版本并发读。

RocksDB Transaction
简单的例子:

// 基本配置,事务相关操作需要TransactionDB句柄
Options options;
options.create_if_missing = true;
TransactionDBOptions txn_db_options;
TransactionDB* txn_db;
 
// 用支持事务的方式opendb
TransactionDB::Open(options, txn_db_options, kDBPath, &txn_db);
 
// 创建一个事务上下文, 类似MySQL的start transaction
Transaction* txn = txn_db->BeginTransaction(write_options);
// 直接写入新数据
txn->Put("abc", "def");
// ForUpdate写，类似MySQL的select ... for update
s = txn->GetForUpdate(read_options, "abc", &value); 
 
txn->Commit();      // or txn->Rollback();
 
RocksDB的一个事物操作，是通过事物内部申请一个WriteBatch实现的，所有commit之前的读都优先读该WriteBatch(保证了同一个事务内可以看到该事务之前的写操作)，写都直接写入该事务独有的WriteBatch中，提交时在依次写入WAL和memtable，依赖WriteBatch的原子性和隔离性实现了ACID。

###事务并发
不同的并发事务之间，如果存在数据冲突，会有如下情况：

事务都是读事务，无论操作的记录间是否有交集，都不会锁定。
事务包含读、写事务：
所有的读事务不会锁定，读到的数据取决于snapshot设置。
写事务之间如果不存在记录交集，不会锁定。
写事务之间如果存在记录交集，此时如果未设置snapshot，则交集部分的记录是可以串行提交的。如果设置了snapshot，则第一个写事务(写锁队列的head)会成功，其他写事务会失败(之前的事务修改了该记录的情况下)。

###独占写锁和写冲突
RocksDB事务写锁是基于Key Locking行锁的(实现上锁力度会粗一些)，所以在多个Transaction同时更新一条记录，会触发独占写锁定。如果还设置了snapshot的情况下，会触发写冲突分析。每个写操作(Put/Delete/Merge/GetForUpdate)开始之前，会进行写锁定，见TransactionLockMgr代码。如果存在记录有交集，写锁定会锁住一片key保证只有一个事物会独占写。




###死锁检测/超时
创建事务时 TransactionOptions.deadlock_detect 选项可以支持死锁检测(默认不开启，性能影响较大，尤其是热点记录场景下。依赖timeout机制解决死锁)。如果多个事务之间发生死锁，则当前检测到死锁的事物失败(可以回滚)。死锁检测是通过刚才提到的LockInfo中全局事物ID列表以和当前事务ID进行环检测实现，通过广度优先递归遍历当前事务ID依赖的事务ID，判断其是否指向自己，如果能递归的找到自己的ID则说明有环，发生死锁。deadlock_detect_depth参数可以指定检测的深度，防止过深的依赖。

Optimistic Transaction
相较于悲观锁，RocksDB也实现了一套乐观锁机制的OptimisticTransaction，接口上和Transaction是一致的。不过在写操作(Put/Delete/Merge/GetForUpdate)时，不会触发独占写锁和写冲突检测，而是在事务commit时("乐观"锁)，写入WAL时判断是否存在写冲突，而commit失败。这种方式的好处时，更新操作或者GetForUpdate()时，不用加独占写锁，省去了加锁的代价，乐观的认为没有写冲突，推迟到事务提交时一次性提交所有写入的key进行判断。

###MVCC
RocksDB实现的ReadCommited和RepeatableRead隔离级别，类似其他数据库引擎，都使用MVCC机制。例如MySQL的InnoDB，通过undo page实现了行记录的多版本，这样可以在不同的隔离级别下，看到不同时刻的行记录内容。不过undo需要undo页的存储空间以及redo日志的保护(redo写undo)，这跟其btree的in-place update有关，而RocksDB依靠其天然的AppendOnly，所有的写操作都是后期merge，自然地就是key的多版本(不同版本可能位于memtable,immemtable,sst)，所以RocksDB首先MVCC是很容易的，只需要通过snapshot(lsn)稍加限制即可实现。

例如需要读取比某个lsn小的历史版本，只需要在读取时指定一个带有这个lsn的snapshot，即可读到历史版本。所以，在需要一致性非锁定读读取操作时，默认ReadCommited只需要按照当前系统中最大的lsn读取(这个也是默认DB::Get()的行为)，即可读到已经提交的最新记录(提交到memtable后的记录一定是已经commit的记录，未commit之前记录保存在transaction的临时buffer里)。

在RepeatableRead下读数据是，需要指定该事务的读上界(即创建事务时的snapshot(lsn)或通过SetSnapshot指定的当时的lsn)，已提交的数据一定大于该snapshot(lsn)，即可实现可重复读。

可见snapshot对于MVCC有着很重要的意义：

snapshot可以实现不同隔离级别的非锁定读
snapshot可以用于写冲突检测
snapshot由全局的snapshot链表进行管理，在compaction时，会保留该链表中snapshot不被回收


##RocksDB系列一：RocksDB基础和入门
https://www.jianshu.com/p/061927761027

##RocksDB系列八：Bloom Filter
https://www.jianshu.com/p/e26b6e3df542
What is a Bloom Filter?
  在任意的keys集合中，应用一个算法并生成一个字节数组，这个字节数组就是Bloom filter。对于任意一个key，通过Bloom filter可以得出两个结论：1) 这个key有可能在集合中 2)这个key肯定不在集合中。
  在RocksDB引擎中，如果设置了filter policy的话，每个新创建的SST file都会包含一个Bloom filter，这个Bloom filter可以确定我们要查找的key是否有可能在这个SST file中。Bloom filter其实就是一个bit array。在计算时，会使用多个hash 函数来计算指定的key，得到多个position，然后将bit array中相应的position 置为1。在使用Bloom filter时，对于一个key，仍然使用相同的hash函数来计算得出多个position，然后分别去check bit array中相应位置的值。只要有一个位置上的值不为1，则这个key 肯定不在集合中，否则，这个key就有可能在集合中（很大概率在集合中，除非集合超级超级大时才会有多个key映射到相同的n个位置）。

LifeCycle
  RocksDB中，每个SST file都有相应的一个Bloom filter，这个Bloom filter是在SST file写入存储时创建的，Bloom filter数据存储在相应的SST file中。其他各层的SST file都是用相同的方法生成Bloom filter。
  Bloom filter只能从一个key集合中生成，所以我们不能combine多个Bloom filter。如果需要combine两个SST file，就会从combine之后的新的SST file中重新生成一个Bloom filter。
  当打开一个SST file时，相应的Bloom filter也会被打开然后load到内存中。当SST file关闭时，对应的Bloom filt
er就会从内存中remove。
 
##RocksDB系列十二：Checkpoints
https://www.jianshu.com/p/87a9c9be1c28
  Checkpoint是RocksDB的一个feature，主要支持对当前正在运行的数据库制作一个snapshot。Checkpoints是一个时间点上的snapshot。当使用Read-only模式打开的话，可以支持查询这个时间点上的数据，当使用Read-Write模式打开的话，可以作为一个可写的snapshot。Checkpoints可以作为全量或者新增备份的backup使用。
  给定一个RocksDB数据库，Checkpoint功能可以创建一个满足数据一致性的快照。如果snapshot所在的文件系统和DB file所在的文件系统相同的话，SST files会被硬链接，否则，就要全部拷贝过去，manifest和CURRENT files也会被拷贝过去。另外，如果有多个列族的话，在checkpoint的start和end时间段内的log文件也会被拷贝过去，这么做的目的是提供所有列族数据的一致性snapshot。

在创建checkpoints之前，需要创建一个checkpoint对象。
Status Create(DB* db, Checkpoint** checkpoint_ptr);
  给定一个checkpoint对象和目录，CreateCheckpoint 函数会在给定目录中创建数据库的满足数据一致性的snapshot。

Status CreateCheckpoint(const std::string& checkpoint_dir);
  函数参数中的目录不应该存在，这个目录会在函数中创建，另外，目录必须是一个绝对路径。checkpoint可以用作DB的一个read-only copy，也可以用作一个standalone DB。当打开了read/write时，SST file仍然是硬链接，后续如果文件被废弃后，硬链接也会被删除掉。如果用户不再使用这个snapshot了，用户可以直接删除这个数据目录来删除一个snapshot。

checkpoints广泛应用于MyRocks的在线备份。MyRocks是MySQL使用RocksDB作为存储引擎的版本。


##RocksDB系列二十二:RocksDB使用场景和特性
https://www.jianshu.com/p/3302be5542c7
  存储和访问数百PB的数据是一个非常大的挑战，开源的RocksDB就是FaceBook开放的一种嵌入式、持久化存储、KV型且非常适用于fast storage的存储引擎。
  传统的数据访问都是RPC，但是这样的话访问速度会很慢，不适用于面向用户的实时访问的场景。随着fast storage的流行，越来越多的应用可以通过在flash中管理数据并快速直接的访问数据。这些应用就需要使用到一种嵌入式的database。
  使用嵌入式的database的原因有很多。当数据请求频繁访问内存或者fast storage时，网路延时会增加响应时间，比如：访问数据中心网络耗时可能就耗费50ms，跟访问数据的耗时一样多，甚至更多。这意味着，通过RPC访问数据有可能是本地直接访问耗时的两倍。另外，机器的core数越来越多，storage-IOPS的访问频率也达到了每秒百万次，传统数据库的锁竞争和context 切换会成为提高storage-IOPS的瓶颈。所以需要一种容易扩展和针对未来硬件趋势可以定制化的database，RocksDB就是一种选择。
  RocksDB是基于Google的开源key value存储库LevelDB，主要满足以下目标：
1、适用于多cpu场景
  商业服务器一般会有很多cpu核，要开发一个随着CPU 核数吞吐量也随之增大的数据库是很困难的，更别提是线性的递增关系。但是，RocksDB是可以高效地运行在多核服务器上。一个优点是RocksDB提供的语义比传统的DBMS更简单。例如：RocksDB支持MVCC，但是仅限于只读的transaction。另一个优点是数据库在逻辑上分片为read-only path和read-write path。这两种方法可以降低锁竞争，而降低锁竞争是支持高并发负载的前提条件。
2、高校利用storage(更高的IOPS、高效的压缩、更少的写磨损)
  现在的存储设备都可以支持到每秒10w的随机读，如果有10块存储卡的话就可以支持每秒100w的随机读。RocksDB可以在这种快速存储上高效运行且不会成为性能瓶颈。
  和实时更新的B-tree相比，RocksDB有更好的压缩和更小的写放大。RocksDB由于压缩更优，所以占用更少的storage；由于更小的写放大，flash 设备可以更持久。
3、弹性架构，支持扩展
  RocksDB支持扩展。比如，我们可以新增一个merge operator，这样就可以使用write-only来替代read-modify-write。然而，read和Write是会增加存储的读写IOPS。在写频繁的负载下，这种措施可以降低IOPS。
4、支持IO-bound、in-memory、write-once
  IO-bound workload是指数据库大小远大于内存且频繁地访问storage。in-memory workload是指数据库数据都在内存中且仍然使用storage来持久化存储DB。write-once workload是指大部分的key都只会写入一次或者insert且没有更新操作。现在RocksDB很好支持IO-bound，要想更好地支持in-memory，需要做一些工作。支持write-once的话，还有很多遗留问题待解决。
  RocksDB不是一个分布式的DB，而是一个高效、高性能、单点的数据库引擎。RocksDB是一个持久化存储keys和values的c++ library。keys 和values可以是任意的字节流，且按照keys有序存储。后台的compaction会消除重复的和已删除的key。RocksDB的data以log-structured merge tree的形式存储。RocksDB支持原子的批量写入操作以及前向和后向遍历。
  RocksDB采用“可插拔式”的架构，所以很容易替换其中的组件，允许用户很容易在不同的负载和硬件设备上进行调优。


  比如，用户可以添加不同的压缩模块(snappy, zlib, bzip, etc)，且使用不同模块时不用修改源码。这可用于在不同负载下通过配置使用不同的压缩算法。同理，用户可以在compaction时加载个性化的compaction filter来处理keys，例如，可以实现DB的key的"expire-time"功能。RocksDB有可插拔式的API，所以应用可以设计个性化的数据结构来cache DB的写数据，典型应用就是prefix-hash，其中一部分key使用hash存储，剩下的key存储在B-tree。storage file的实现也可以定制开发，所以用户可以实现自己的storage file格式。
  RocksDB支持两种compaction style（level style和universal style）。这两种style可做读放大、写放大、空间放大之间做tradeoff。compaction也支持多线程，所以打的DB可以支持高性能的compaction。
  RocksDB也提供在线的增量备份接口，也支持bloom filters，这可以在range-scan时降低IOPS。
  RocksDB可以充分挖掘使用flash的IOPS，在随机读、随机写和bulk load时性能优于LevelDB。在随机写和bulk load时，性能优于LevelDB 10倍，在随机读时性能优于LevelDB 30%。
  LevelDB是单线程执行compaction，在特定的server workload下表现堪忧，但是RocksDB在IO-bound workload下性能明显优于LevelDB。在测试中发现，LevelDB发生频繁的write-stall，这严重影响了DB的99%延迟，另外也发现，把文件mmap到OS cache会引入读性能瓶颈。测试表明，应用不能充分使用flash的高性能，这是因为数据的带宽瓶颈引起了LevelDB的写放大。通过提高写速率和降低写放大，可以避免很多问题，同时提高RocksDB性能。
RocksDB的典型场景（低延时访问）:
1、需要存储用户的查阅历史记录和网站用户的应用
2、需要快速访问数据的垃圾检测应用
3、需要实时scan数据集的图搜索query
4、需要实时请求Hadoop的应用
5、支持大量写和删除操作的消息队列

##Rocksdb基本用法 
https://www.cnblogs.com/wanshuafe/p/11564148.html

rocksdb 用法简单介绍
RocksDB是使用C++编写的嵌入式kv存储引擎，其键值均允许使用二进制流。由Facebook基于levelDB开发， 提供向后兼容的levelDB API。
RocksDB针对Flash存储进行优化，延迟极小。RocksDB使用LSM存储引擎，纯C++编写。
打开一个数据库
rocksdb::DB* db;
rocksdb::Options options;
options.create_if_missing = true;
rocksdb::Status status = rocksdb::DB::Open(options, "/tmp/testdb", &db);
assert(status.ok());
往数据库写入kv键值对
s = db->Put(WriteOptions(), "k1", "v1");
向数据库查询k的value
s = db->Get(ReadOptions(), "k1", &v);
原子写入
rocksdb::WriteBatch batch;
batch.Delete("k1");
batch.Put("k2", "v2");
s = db->Write(rocksdb::WriteOptions(), &batch);
迭代器
db->NewIterator(rocksdb::ReadOptions());
可以通过在调用NewIterator的时候，给传入的option设定ReadOptions.iterate_upper_bound来为你的迭代范围设置一个上边界。
通过这个设定，rocksdb就不用继续查找这个key之后的内容了。在一些情况下，可以节省一些IO和计算。在特定的工作载荷下，
它带来的改善是显著的。这个选项可以同在正向和反向迭代。
rocksdb::Iterator* it = db->NewIterator(rocksdb::ReadOptions());
for (it->SeekToFirst(); it->Valid(); it->Next()) {
cout << it->key().ToString() << ": " << it->value().ToString() << endl;
}
事务操作
Status s = TransactionDB::Open(options,txn_db_options,kDbPathTran,&txn_db );
Transaction* txn = txn_db->BeginTransaction(WriteOptions());
获得事务，就可以进行相应的op操作，如下：
s = txn->Put("key", "value");
s = txn->Delete("key2");
然后对事务进行提交
s = txn->Commit();

