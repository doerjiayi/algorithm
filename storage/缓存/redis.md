#redis
Redis系列八：redis主从复制和哨兵
https://www.cnblogs.com/leeSmall/p/8398401.html 

Redis Cluster流程原理
https://blog.csdn.net/fouy_yun/article/details/81590252  
https://blog.csdn.net/qq2430/article/details/80716313  浅谈Redis Cluster


##Redis-Cluster
美团在Redis上踩过的一些坑-5.redis cluster遇到的一些问题
https://blog.csdn.net/carlosfu/article/details/84749049 
我们从Redis-Cluster beta版 RC1~4 到现在的3.0-release均没有遇到什么大问题（线上维护600个实例）。

redis cluster 扩容
https://segmentfault.com/a/1190000016359421

Redis集群节点的选举（实验）
https://blog.51cto.com/13690439/2119973 

Redis Cluster
https://segmentfault.com/a/1190000017578588  

redis cluster 扩容
https://segmentfault.com/a/1190000016359421 

##浅谈Redis中的Rehash机制
https://blog.csdn.net/cqk0100/article/details/80400811  

##Redis五种数据类型
https://blog.csdn.net/zh15732621679/article/details/80614091 

##redis集群核心原理:gossip通信、jedis Smart定位、主备切换
https://blog.csdn.net/r_p_j/article/details/78813265

##《Redis设计与实现总结》
https://blog.csdn.net/qq_39843374/article/details/80979936

##《Redis设计与实现》学习笔记
https://www.cnblogs.com/mengchunchen/p/9025139.html

4.4 rehash（重新散列）
　　随着操作进行，哈希表保存的键值对会增加或减少，为了让哈希表的负载因子（load factor）维持在一个合理范围，当一个哈希表保存的键太多或者太少，需要对哈希表进行扩展或者收缩。扩展或收缩哈希表的过程，就称为rehash。
　　rehash步骤如下：
　　1、给字典的ht[1]申请存储空间，大小取决于要进行的操作，以及ht[0]当前键值对的数量（ht[0].used）。假设当前ht[0].used=x。
　　　　如果是扩展，则ht[1]的值是第一个大于等于x*2的2n的值。例如x是30，则ht[1]的大小是第一个大于等于30*2的2n的值，即64。
　　　　如果是收缩，则ht[1]的值是第一个大于等于x的2n的值。例如x是30，则ht[1]的大小是第一个大于等于30的2n的值，即32。
　　2、将保存在ht[0]上面的所有键值对，rehash到ht[1]，即对每个键重新采用哈希算法的方式计算哈希值和索引值，再放到相应的ht[1]的表格指定位置。
　　3、当ht[0]的所有键值对都rehash到ht[1]后，释放ht[0]，并将ht[1]设置为ht[0]，再新建一个空的ht[1]，用于下一次rehash。
 
　　rehash条件：
　　负载因子（load factor）计算：
　　load_factor =ht[0].used / ht[0].size，即负载因子大小等于当前哈希表的键值对数量，除以当前哈希表的大小。
 
　　扩展：
　　当以下任一条件满足，哈希表会自动进行扩展操作：
　　1）服务器目前没有在执行BGSAVE或者BGREWRITEAOF命令，且负载因子大于等于1。
　　2）服务器目前正在在执行BGSAVE或者BGREWRITEAOF命令，且负载因子大于等于5。


第4章 字典
　　字典，又称符号表、关联数组、映射，是一种保存键值对的抽象数据结构。
　　每个键（key）和唯一的值（value）关联，键是独一无二的，通过对键的操作可以对值进行增删改查。
　　redis中字典应用广泛，对redis数据库的增删改查就是通过字典实现的。即redis数据库的存储，和大部分关系型数据库不同，不采用B+tree进行处理，而是采用hash的方式进行处理。
　　字典还是hash键的底层实现之一。
　　当hash键包含了许多元素，或者元素是比较长的字符串的时候，就会用到字典作为hash键的底层实现。

4.1 字典的实现
redis的字典，底层是使用哈希表实现，每个哈希表有多个哈希节点，每个哈希节点保存了一个键值对。
1、哈希表

 1 /*
 2  * 哈希表
 3  *
 4  * 每个字典都使用两个哈希表，从而实现渐进式 rehash 。
 5  */
 6 typedef struct dictht {
 7     
 8     // 哈希表数组
 9     dictEntry **table;
10 
11     // 哈希表大小
12     unsigned long size;
13     
14     // 哈希表大小掩码，用于计算索引值
15     // 总是等于 size - 1
16     unsigned long sizemask;
17 
18     // 该哈希表已有节点的数量
19     unsigned long used;
20 
21 } dictht;
 
　　其中，table是一个数组，里面的每个元素指向dictEntry（哈希表节点）结构的指针，dictEntry结构是键值对的结构；
　　size表示哈希表的大小，也是table数组的大小；
　　used表示table目前已有的键值对节点数量；
　　sizemask一直等于size-1，该值与哈希值一起决定一个属性应该放到table的哪个位置。
　　大小为4的空哈希表结构如下图（左边一列的图）所示：

3、字典

 1 /*
 2  * 字典
 3  */
 4 typedef struct dict {
 5 
 6     // 类型特定函数
 7     dictType *type;
 8 
 9     // 私有数据
10     void *privdata;
11 
12     // 哈希表
13     dictht ht[2];
14 
15     // rehash 索引
16     // 当 rehash 不在进行时，值为 -1
17     int rehashidx; /* rehashing not in progress if rehashidx == -1 */
18 
19     // 目前正在运行的安全迭代器的数量
20     int iterators; /* number of iterators currently running */
21 
22 } dict;

 
　　type用于存放用于处理特定类型的处理函数；
　　privdata用于存放私有数据，保存传给type内的函数的数据；
　　rehash是一个索引，当没有在rehash进行时，值是-1；
　　ht是包含两个项的数组，每个项是一个哈希表，一般情况下只是用ht[0]，只有在对ht[0]进行rehash时，才会使用ht[1]。

4.2 哈希算法
　　要将新的键值对加到字典，程序要先对键进行哈希算法，算出哈希值和索引值，再根据索引值，把包含新键值对的哈希表节点放到哈希表数组指定的索引上。
　　redis实现哈希的代码是：
　　hash =dict->type->hashFunction(key);
　　index = hash& dict->ht[x].sizemask;
　　算出来的结果中，index的值是多少，则key会落在table里面的第index个位置（第一个位置index是0）。
　　其中，redis的hashFunction，采用的是murmurhash2算法，是一种非加密型hash算法，其具有高速的特点。

4.3 键冲突解决
　　当两个或者以上的键被分配到哈希表数组的同一个索引上，则称这些键发生了冲突。
　　为了解决此问题，redis采用链地址法。被分配到同一个索引上的多个节点可以用单链表连接起来。
　　因为没有指向尾节点的指针，所以总是将新节点加在表头的位置。（O(1)时间）

4.4 rehash（重新散列）
　　随着操作进行，哈希表保存的键值对会增加或减少，为了让哈希表的负载因子（load factor）维持在一个合理范围，当一个哈希表保存的键太多或者太少，需要对哈希表进行扩展或者收缩。扩展或收缩哈希表的过程，就称为rehash。
　　rehash步骤如下：
　　1、给字典的ht[1]申请存储空间，大小取决于要进行的操作，以及ht[0]当前键值对的数量（ht[0].used）。假设当前ht[0].used=x。
　　　　如果是扩展，则ht[1]的值是第一个大于等于x*2的2n的值。例如x是30，则ht[1]的大小是第一个大于等于30*2的2n的值，即64。
　　　　如果是收缩，则ht[1]的值是第一个大于等于x的2n的值。例如x是30，则ht[1]的大小是第一个大于等于30的2n的值，即32。
　　2、将保存在ht[0]上面的所有键值对，rehash到ht[1]，即对每个键重新采用哈希算法的方式计算哈希值和索引值，再放到相应的ht[1]的表格指定位置。
　　3、当ht[0]的所有键值对都rehash到ht[1]后，释放ht[0]，并将ht[1]设置为ht[0]，再新建一个空的ht[1]，用于下一次rehash。
 
　　rehash条件：
　　负载因子（load factor）计算：
　　load_factor =ht[0].used / ht[0].size，即负载因子大小等于当前哈希表的键值对数量，除以当前哈希表的大小。
 
　　扩展：
　　当以下任一条件满足，哈希表会自动进行扩展操作：
　　1）服务器目前没有在执行BGSAVE或者BGREWRITEAOF命令，且负载因子大于等于1。
　　2）服务器目前正在在执行BGSAVE或者BGREWRITEAOF命令，且负载因子大于等于5。
 
　　收缩：
　　当负载因子小于0.1时，redis自动开始哈希表的收缩工作。

4.5 渐进式rehash
　　redis对ht[0]扩展或收缩到ht[1]的过程，并不是一次性完成的，而是渐进式、分多次的完成，以避免如果哈希表中存有大量键值对，一次性复制过程中，占用资源较多，会导致redis服务停用的问题。
　　渐进式rehash过程如下：
　　1、为ht[1]分配空间，让字典同时持有ht[0]和ht[1]两张哈希表。
　　2、将字典中的rehashidx设置成0，表示正在rehash。rehashidx的值默认是-1，表示没有在rehash。
　　3、在rehash进行期间，程序处理正常对字典进行增删改查以外，还会顺带将ht[0]哈希表上，rehashidx索引上，所有的键值对数据rehash到ht[1]，并且rehashidx的值加1。
　　4、当某个时间节点，全部的ht[0]都迁移到ht[1]后，rehashidx的值重新设定为-1，表示rehash完成。
 
　　渐进式rehash采用分而治之的工作方式，将哈希表的迁移工作所耗费的时间，平摊到增删改查中，避免集中rehash导致的庞大计算量。
　　在rehash期间，对哈希表的查找、修改、删除，会先在ht[0]进行。
　　如果ht[0]中没找到相应的内容，则会去ht[1]查找，并进行相关的修改、删除操作。而增加的操作，会直接增加到ht[1]中，目的是让ht[0]只减不增，加快迁移的速度。

4.6 总结
　　字典在redis中广泛应用，包括数据库和hash数据结构。
　　每个字典有两个哈希表，一个是正常使用，一个用于rehash期间使用。
　　当redis计算哈希时，采用的是MurmurHash2哈希算法。
　　哈希表采用链地址法避免键的冲突，被分配到同一个地址的键会构成一个单向链表。
　　在rehash对哈希表进行扩展或者收缩过程中，会将所有键值对进行迁移，并且这个迁移是渐进式的迁移。

第5章 跳跃表
　　跳跃表（skiplist）是一种有序的数据结构，它通过每个节点中维持多个指向其他节点的指针，从而实现快速访问。
　　跳跃表平均O(logN)，最坏O(N)，支持顺序遍历查找。
　　在redis中，有序集合（sortedset）的其中一种实现方式就是跳跃表。
　　当有序集合的元素较多，或者集合中的元素是比较常的字符串，则会使用跳跃表来实现。

5.1 跳跃表实现
跳跃表是由各个跳跃表节点组成。

 1 /* ZSETs use a specialized version of Skiplists */
 2 /*
 3  * 跳跃表节点
 4  */
 5 typedef struct zskiplistNode {
 6 
 7     // 成员对象
 8     robj *obj;
 9 
10     // 分值
11     double score;
12 
13     // 后退指针
14     struct zskiplistNode *backward;
15 
16     // 层
17     struct zskiplistLevel {
18 
19         // 前进指针
20         struct zskiplistNode *forward;
21 
22         // 跨度
23         unsigned int span;
24 
25     } level[];
26 
27 } zskiplistNode;

 1 /*
 2  * 跳跃表
 3  */
 4 typedef struct zskiplist {
 5 
 6     // 表头节点和表尾节点
 7     struct zskiplistNode *header, *tail;
 8 
 9     // 表中节点的数量
10     unsigned long length;
11 
12     // 表中层数最大的节点的层数
13     int level;
14 
15 } zskiplist;

上图最左边就是跳跃表的结构：
　　header和tail：是跳跃表节点的头结点和尾节点，
　　length：是跳跃表的长度（即跳跃表节点的数量，不含头结点），
　　level：表示层数中最大节点的层数（不计算表头结点）。
　　因此，获取跳跃表的表头、表尾、最大层数、长度的时间复杂度都是O(1)。
 
跳跃表节点:
　　层：节点中用L1,L2表示各层，每个层都有两个属性，前进指针（forward）和跨度（span）。每个节点的层高是1到32的随机数
　　前进指针：用于访问表尾方向的节点，便于跳跃表正向遍历节点的时候，查找下一个节点位置；
　　跨度：记录前进指针所指的节点和当前节点的距离，用于计算排位，访问过程中，将沿途访问的所有层的跨度累计起来，得到的结果就是跳跃表的排位。
　　后退指针：节点中用BW来表示，其指向当前节点的前一个节点，用于反向遍历时候使用。每次只能后退至前一个节点。
　　分值：各节点中的数字就是分值，跳跃表中，节点按照分值从小到大排列。
　　成员对象：各个节点中，o1，o2是节点所保存的成员对象。是一个指针，指向一个字符串对象。
　　表头节点也有后退指针，分值，成员对象，因为不会被用到，所以图中省略。
　　分值可以相同，成员对象必须唯一。
　　分值相同时，按照成员对象的字典序从小到大排。

第6章 整数集合
　　整数集合（intset）是集合键的底层实现之一，当一个集合只包含整数值元素，并且这个集合的元素数量不多时，Redis就会使用整数集合作为集合键的底层实现。
　　它可以保存类型为int16_t、int32_t或者int64_t的整数值，并且保证集合中不会出现重复元素。
 

1 typedef struct intset {
2     // 编码方式
3     uint32_t encoding;
4     // 集合包含的元素数量
5     uint32_t length;
6     // 保存元素的数组
7     int8_t contents[];
8 } intset;

##redis为什么不选择C++作为实现语言而选择C语言来自己实现简单动态字符串(SDS)，链表，字典(map)等？
http://www.imooc.com/wenda/detail/558629

##redis为什么不使用btree而是使用skiplist
https://blog.csdn.net/linyilong3/article/details/103319541

##为啥 Redis 使用跳表而不是红黑树 
https://www.cnblogs.com/everlose/p/13034513.html

redis的sortset为什么使用的是跳表，而不使用红黑树？
https://blog.csdn.net/weixin_40413961/article/details/102303460


##redis命中率计算
https://blog.csdn.net/liuxiao723846/article/details/51445425

[root@im2 deploy]# telnet 192.168.11.71 29000  
Trying 192.168.11.71...
Connected to 192.168.11.71.
Escape character is '^]'.
info

 Memory
used_memory:2972808
used_memory_human:2.84M
used_memory_rss:14344192
used_memory_rss_human:13.68M
used_memory_peak:3716744
used_memory_peak_human:3.54M
total_system_memory:8182030336
total_system_memory_human:7.62G
used_memory_lua:37888
used_memory_lua_human:37.00K 

 
expired_keys:213
evicted_keys:0
keyspace_hits:176913
keyspace_misses:453078

##Redis有效时间设置及时间过期处理 
https://www.cnblogs.com/lxwphp/p/11319934.html  

redis 设置过期时间
https://www.cnblogs.com/guoyu1/p/12264636.html

##REDIS缓存穿透，缓存击穿，缓存雪崩原因+解决方案 
https://www.cnblogs.com/xichji/p/11286443.html

二、初认识
缓存穿透：key对应的数据在数据源并不存在，每次针对此key的请求从缓存获取不到，请求都会到数据源，从而可能压垮数据源。比如用一个不存在的用户id获取用户信息，不论缓存还是数据库都没有，若黑客利用此漏洞进行攻击可能压垮数据库。

缓存击穿：key对应的数据存在，但在redis中过期，此时若有大量并发请求过来，这些请求发现缓存过期一般都会从后端DB加载数据并回设到缓存，这个时候大并发的请求可能会瞬间把后端DB压垮。

缓存雪崩：当缓存服务器重启或者大量缓存集中在某一个时间段失效，这样在失效的时候，也会给后端系统(比如DB)带来很大压力。

三、缓存穿透解决方案
一个一定不存在缓存及查询不到的数据，由于缓存是不命中时被动写的，并且出于容错考虑，如果从存储层查不到数据则不写入缓存，这将导致这个不存在的数据每次请求都要到存储层去查询，失去了缓存的意义。
有很多种方法可以有效地解决缓存穿透问题，最常见的则是采用布隆过滤器，将所有可能存在的数据哈希到一个足够大的bitmap中，一个一定不存在的数据会被 这个bitmap拦截掉，从而避免了对底层存储系统的查询压力。另外也有一个更为简单粗暴的方法（我们采用的就是这种），如果一个查询返回的数据为空（不管是数据不存在，还是系统故障），我们仍然把这个空结果进行缓存，但它的过期时间会很短，最长不超过五分钟。

##帮你解读什么是Redis缓存穿透和缓存雪崩（包含解决方案）
https://baijiahao.baidu.com/s?id=1655304940308056733&wfr=spider&for=pc
一、缓存穿透
1、概念
缓存穿透的概念很简单，用户想要查询一个数据，发现redis内存数据库没有，也就是缓存没有命中，于是向持久层数据库查询。发现也没有，于是本次查询失败。当用户很多的时候，缓存都没有命中，于是都去请求了持久层数据库。这会给持久层数据库造成很大的压力，这时候就相当于出现了缓存穿透。
这里需要注意和缓存击穿的区别，缓存击穿，是指一个key非常热点，在不停的扛着大并发，大并发集中对这一个点进行访问，当这个key在失效的瞬间，持续的大并发就穿破缓存，直接请求数据库，就像在一个屏障上凿开了一个洞。
为了避免缓存穿透其实有很多种解决方案。下面介绍几种。
2、解决方案
（1）布隆过滤器
布隆过滤器是一种数据结构，垃圾网站和正常网站加起来全世界据统计也有几十亿个。网警要过滤这些垃圾网站，总不能到数据库里面一个一个去比较吧，这就可以使用布隆过滤器。假设我们存储一亿个垃圾网站地址。
可以先有一亿个二进制比特，然后网警用八个不同的随机数产生器（F1,F2, …,F8） 产生八个信息指纹（f1, f2, …, f8）。接下来用一个随机数产生器 G 把这八个信息指纹映射到 1 到1亿中的八个自然数 g1, g2, …,g8。最后把这八个位置的二进制全部设置为一。过程如下：

有一天网警查到了一个可疑的网站，想判断一下是否是XX网站，首先将可疑网站通过哈希映射到1亿个比特数组上的8个点。如果8个点的其中有一个点不为1，则可以判断该元素一定不存在集合中。
那这个布隆过滤器是如何解决redis中的缓存穿透呢？很简单首先也是对所有可能查询的参数以hash形式存储，当用户想要查询的时候，使用布隆过滤器发现不在集合中，就直接丢弃，不再对持久层查询。

这个形式很简单。

（2）、缓存空对象
当存储层不命中后，即使返回的空对象也将其缓存起来，同时会设置一个过期时间，之后再访问这个数据将会从缓存中获取，保护了后端数据源；

但是这种方法会存在两个问题：
如果空值能够被缓存起来，这就意味着缓存需要更多的空间存储更多的键，因为这当中可能会有很多的空值的键；
即使对空值设置了过期时间，还是会存在缓存层和存储层的数据会有一段时间窗口的不一致，这对于需要保持一致性的业务会有影响。

二、缓存雪崩
1、概念
缓存雪崩是指，缓存层出现了错误，不能正常工作了。于是所有的请求都会达到存储层，存储层的调用量会暴增，造成存储层也会挂掉的情况。

2、解决方案
（1）redis高可用
这个思想的含义是，既然redis有可能挂掉，那我多增设几台redis，这样一台挂掉之后其他的还可以继续工作，其实就是搭建的集群。
（2）限流降级
这个解决方案的思想是，在缓存失效后，通过加锁或者队列来控制读数据库写缓存的线程数量。比如对某个key只允许一个线程查询数据和写缓存，其他线程等待。
（3）数据预热
数据加热的含义就是在正式部署之前，我先把可能的数据先预先访问一遍，这样部分可能大量访问的数据就会加载到缓存中。在即将发生大并发访问前手动触发加载缓存不同的key，设置不同的过期时间，让缓存失效的时间点尽量均匀。


##一文看懂 Redis5 搭建集群  
https://my.oschina.net/ruoli/blog/2252393 

Why Redis 4.0?
https://www.liangzl.com/get-article-detail-5391.html

如何提高缓存命中率（Redis）
https://blog.csdn.net/weixin_34238633/article/details/85968131
https://github.com/junegunn/redis-stat

redis命中率不高问题排查
https://blog.csdn.net/h952520296/article/details/84880094

一次Redis问题排查
https://blog.csdn.net/KimmKing/article/details/79815979

redis cluster
https://www.iteye.com/blog/shift-alt-ctrl-2285470

sentinel
https://blog.csdn.net/sanwenyublog/article/details/53385616

redis订阅
https://www.cnblogs.com/kellynic/p/9952386.html

##Redis Cluster实现原理 
redis raft
https://www.iteye.com/blog/shift-alt-ctrl-2285470

一、Redis Cluster主要特性和设计
    集群目标
    1）高性能和线性扩展，最大可以支撑到1000个节点；Cluster架构中无Proxy层，Master与slave之间使用异步replication，且不存在操作的merge。（即操作不能跨多个nodes，不存在merge层）
    2）一定程度上保证writes的安全性，需要客户端容忍一定程度的数据丢失：集群将会尽可能（best-effort）保存客户端write操作的数据；通常在failover期间，会有短暂时间内的数据丢失（因为异步replication引起）；当客户端与少数派的节点处于网络分区时（network partition），丢失数据的可能性会更高。（因为节点有效性检测、failover需要更长的时间）
    3）可用性：只要集群中大多数master可达、且失效的master至少有一个slave可达时，集群都可以继续提供服务；同时“replicas migration”可以将那些拥有多个slaves的master的某个slave，迁移到没有slave的master下，即将slaves的分布在整个集群相对平衡，尽力确保每个master都有一定数量的slave备份。

  Redis Cluster集群有多个shard组成，每个shard可以有一个master和多个slaves构成，数据根据hash slots配额分布在多个shard节点上，节点之间建立双向TCP链接用于有效性检测、Failover等，Client直接与shard节点进行通讯；Cluster集群没有Proxy层，也没有中央式的Master用于协调集群状态或者state存储；集群暂不提供动态reblance策略）
    备注：下文中提到的query、查询等语义，泛指redis的读写操作。

 Mutli-key操作
    Redis单实例支持的命令，Cluster也都支持，但是对于“multi-key”操作（即一次RPC调用中需要进行多个key的操作）比如Set类型的交集、并集等，则要求这些key必须属于同一个node。Cluster不能进行跨Nodes操作，也没有nodes提供merge层代理。
    Cluster中实现了一个称为“hash tags”的概念，每个key都可以包含一个自定义的“tags”，那么在存储时将根据tags计算此key应该分布在哪个nodes上（而不是使用key计算，但是存储层面仍然是key）；此特性，可以强制某些keys被保存在同一个节点上，以便于进行“multikey”操作，比如“foo”和“{foo}.student”将会被保存在同一个node上。不过在人工对slots进行resharding期间，multikey操作可能不可用。
    我们在Redis单例中，偶尔会用到“SELECT”指令，即可以将key保存在特定的database中（默认database索引号为0）；但是在Cluster环境下，将不支持SELECT命令，所有的key都将保存在默认的database中。

   客户端与Server角色
   集群中nodes负责存储数据，保持集群的状态，包括keys与nodes的对应关系（内部其实为slots与nodes对应关系）。nodes也能够自动发现其他的nodes，检测失效的节点，当某个master失效时还应该能将合适的slave提升为master。
   为了达成这些行为，集群中的每个节点都通过TCP与其他所有nodes建立连接，它们之间的通信协议和方式称为“Redis Cluster Bus”。Nodes之间使用gossip协议（参见下文备注）向其他nodes传播集群信息，以达到自动发现的特性，通过发送ping来确认其他nodes工作正常，也会在合适的时机发送集群的信息。当然在Failover时（包括人为failover）也会使用Bus来传播消息。
  （gossip：最终一致性，分布式服务数据同步算法，node首选需要知道（可以读取配置）集群中至少一个seed node，此node向seed发送ping请求，此时seed节点pong返回自己已知的所有nodes列表，然后node解析nodes列表并与它们都建立tcp连接，同时也会向每个nodes发送ping，并从它们的pong结果中merge出全局nodes列表，并逐步与所有的nodes建立连接.......数据传输的方式也是类似，网络拓扑结构为full mesh）
    因为Node并不提供Proxy机制，当Client将请求发给错误的nodes时（此node上不存在此key所属的slot），node将会反馈“MOVED”或者“ASK”错误信息，以便Client重新定向到合适的node。理论上，Client可以将请求发送给任意一个nodes，然后根据在根据错误信息转发给合适的node，客户端可以不用保存集群的状态信息，当然这种情况下性能比较低效，因为Client可能需要2次TCP调用才能获取key的结果，通常客户端会缓存集群中nodes与slots的映射关系，并在遇到“Redirected”错误反馈时，才会更新本地的缓存。

   安全写入（write safety）
   在Master-slaves之间使用异步replication机制，在failover之后，新的Master将会最终替代其他的replicas（即slave）。在出现网络分区时（network partition），总会有一个窗口期（node timeout）可能会导致数据丢失；不过，Client与多数派的Master、少数派Master处于一个分区（网络分区，因为网络阻断问题，导致Clients与Nodes被隔离成2部分）时，这两种情况下影响并不相同。
   1）write提交到master，master执行完毕后向Client反馈“OK”，不过此时可能数据还没有传播给slaves（异步replication）；如果此时master不可达的时间超过阀值（node timeout，参见配置参数），那么将触发slave被选举为新的Master（即Failover），这意味着那些没有replication到slaves的writes将永远丢失了！
   2）还有一种情况导致数据丢失：
        A）因为网络分区，此时master不可达，且Master与Client处于一个分区，且是少数派分区。
        B）Failover机制，将其中一个slave提升为新Master。
        C）此后网络分区消除，旧的Master再次可达，此时它将被切换成slave。
        D）那么在网络分区期间，处于少数派分区的Client仍然将write提交到旧的Master，因为它们觉得Master仍然有效；当旧的Master再次加入集群，切换成slave之后，这些数据将永远丢失。
   在第二种情况下，如果Master无法与其他大多数Masters通讯的时间超过阀值后，此Master也将不再接收Writes，自动切换为readonly状态。当网络分区消除后，仍然会有一小段时间，客户端的write请求被拒绝，因为此时旧的Master需要更新本地的集群状态、与其他节点建立连接、角色切换为slave等等，同时Client端的路由信息也需要更新。
    只有当此master被大多数其他master不可达的时间达到阀值时，才会触发Failover，这个时间称为NODE_TIMEOUT，可以通过配置设定。所以当网络分区在此时间被消除的话，writes不会有任何丢失。反之，如果网络分区持续时间超过此值，处于“小分区”（minority）端的Master将会切换为readonly状态，拒绝客户端继续提交writes请求，那么“大分区”端将会进行failover，这意味着NODE_TIMEOUT期间发生在“小分区”端的writes操作将丢失（因为新的Master上没有同步到那些数据）。 
    
  可用性
    处于“小分区”的集群节点是不可用的；“大分区”端必须持有大多数Masters，同时每个不可达的Master至少有一个slave也在“大分区”端，当NODE_TIMEOUT时，触发failover，此后集群才是可用的。Redis Cluster在小部分nodes失效后仍然可以恢复有效性，如果application希望大面积节点失效仍然有效，那么Cluster不适合这种情况。
  比如集群有N个Master，且每个Master都有一个slave，那么集群的有效性只能容忍一个节点（master）被分区隔离（即一个master处于小分区端，其他处于大分区端），当第二个节点被分区隔离之前仍保持可用性的概率为1 - (1/(N * 2 - 1))（解释：当第一个节点失效后，剩余N * 2 -1个节点，此时没有slave的Master失效的概率为1/(N * 2 -1)）。比如有5个Master，每个Master有一个slave，当2个nodes被隔离出去（或者失效）后，集群可用性的概率只有1/(5 * 2 - 1) = 11.11%，因此集群不再可用。
    幸好Redis Cluster提供了“replicas migration”机制，在实际应用方面，可以有效的提高集群的可用性，当每次failover发生后，集群都会重新配置、平衡slaves的分布，以更好的抵御下一次失效情况的发生。（具体参见下文）

性能
    Redis Cluster并没有提供Proxy层，而是告知客户端将key的请求转发给合适的nodes。Client保存集群中nodes与keys的映射关系（slots），并保持此数据的更新，所以通常Client总能够将请求直接发送到正确的nodes上。因为采用异步replication，所以master不会等待slaves也保存成功后才向客户端反馈结果，除非显式的指定了WAIT指令。multi-key指令仅限于单个节点内，除了resharding操作外，节点的数据不会在节点间迁移。每个操作只会在特定的一个节点上执行，所以集群的性能为master节点的线性扩展。同时，Clients与每个nodes保持链接，所以请求的延迟等同于单个节点，即请求的延迟并不会因为Cluster的规模增大而受到影响。高性能和扩展性，同时保持合理的数据安全性，是Redis Cluster的设计目标。
 
   Redis Cluster没有Proxy层，Client请求的数据也无法在nodes间merge；因为Redis核心就是K-V数据存储，没有scan类型（sort，limit，group by）的操作，因此merge操作并不被Redis Cluster所接受，而且这种特性会极大增加了Cluster的设计复杂度。（类比于mongodb）

二、Cluster主要组件
    keys分布模型
    集群将key分成16384个slots（hash 槽），slot是数据映射的单位，言外之意，Redis Cluster最多支持16384个nodes（每个nodes持有一个slot）。集群中的每个master持有16384个slots中的一部分，处于“stable”状态时，集群中没有任何slots在节点间迁移，即任意一个hash slot只会被单个node所服务（master，当然可以有多个slave用于replicas，slave也可以用来扩展read请求）。keys与slot的映射关系，是按照如下算法计算的：HASH_SLOT = CRC16(key) mod 16384。其中CRC16是一种冗余码校验和，可以将字符串转换成16位的数字。

  hash tags
    在计算hash slots时有一个意外的情况，用于支持“hash tags”；hash tags用于确保多个keys能够被分配在同一个hash slot中，用于支持multi-key操作。hash tags的实现比较简单，key中“{}”之间的字符串就是当前key的hash tags，如果存在多个“{}”，首个符合规则的字符串作为hash tags，如果“{}”存在多级嵌套，那么最内层首个完整的字符串作为hash tags，比如“{foo}.student”，那么“foo”是hash tags。如果key中存在合法的hash tags，那么在计算hash slots时，将使用hash tags，而不再使用原始的key。即“foo”与“{foo}.student”将得到相同的slot值，不过“{foo}.student”仍作为key来保存数据，即redis中数据的key仍为“{foo}.student”。

   集群节点的属性
    集群中每个节点都有唯一的名字，称之为node ID，一个160位随机数字的16进制表示，在每个节点首次启动时创建。每个节点都将各自的ID保存在实例的配置文件中，此后将一直使用此ID，或者说只要配置文件不被删除，或者没有使用“CLUSTER RESET”指令重置集群，那么此ID将永不会修改。
    集群通过node ID来标识节点，而不是使用IP + port，因为node可以修改它的IP和port，不过如果ID不变，我们仍然认定它是集群中合法一员。集群可以在cluster Bus中通过gossip协议来探测IP、port的变更，并重新配置。
    node ID并不是与node相关的唯一信息，不过是唯一一个全局一致的。每个node还持有如下相关的信息，有些信息是关系集群配置的，其他的信息比如最后ping时间等。每个node也保存其他节点的IP、Port、flags（比如flags表示它是master还是slave）、最近ping的时间、最近pong接收时间、当前配置的epoch、链接的状态，最重要的是还包含此node上持有的hash slots。这些信息均可通过“CLUSTER NODES”指令开查看。
    
   Cluster Bus
    每个Node都有一个特定的TCP端口，用来接收其他nodes的链接；此端口号为面向Client的端口号 + 10000，比如果客户端端口号为6379，那么次node的BUS端口号为16379，客户端端口号可以在配置文件中声明。由此可见，nodes之间的交互通讯是通过Bus端口进行，使用了特定的二进制协议，此端口通常应该只对nodes可用，可以借助防火墙技术来屏蔽其他非法访问。
 
集群拓扑
    Redis Cluster中每个node都与其他nodes的Bus端口建立TCP链接（full mesh，全网）。比如在由N各节点的集群中，每个node有N-1个向外发出的TCP链接，以及N-1个其他nodes发过来的TCP链接；这些TCP链接总是keepalive，不是按需创建的。如果ping发出之后，node在足够长的时间内仍然没有pong响应，那么次node将会被标记为“不可达”，那么与此node的链接将会被刷新或者重建。Nodes之间通过gossip协议和配置更新的机制，来避免每次都交互大量的消息，最终确保在nodes之间的信息传送量是可控的。
    
节点间handshake
    Nodes通过Bus端口发送ping、pong；如果一个节点不属于集群，那么它的消息将会被其他nodes全部丢弃。一个节点被认为是集群成员的方式有2种：
    1）如果此node在“Cluster meet”指令中引入，此命令的主要意义就是将指定node加入集群。那么对于当前节点，将认为指定的node为“可信任的”。（此后将会通过gossip协议传播给其他nodes）
    2）当其他nodes通过gossip引入了新的nodes，这些nodes也是被认为是“可信任的”。
 
   只要我们将一个节点加入集群，最终此节点将会与其他节点建立链接，即cluster可以通过信息交换来自动发现新的节点，链接拓扑仍然是full mesh。
    
三、重定向与resharding
    MOVED重定向
    理论上，Client可以将请求随意发给任何一个node，包括slaves，此node解析query，如果可以执行（比如语法正确，multiple keys都应该在一个node slots上），它会查看key应该属于哪个slot、以及此slot所在的nodes，如果当前node持有此slot，那么query直接执行即可，否则当前node将会向Client反馈“MOVED”错误：
Java代码  
GET X  
-MOVED 3999 127.0.0.1:6381  
    错误信息中包括此key对应的slot（3999），以及此slot所在node的ip和port，对于Client 而言，收到MOVED信息后，它需要将请求重新发给指定的node。不过，当node向Client返回MOVED之前，集群的配置也在变更（节点调整、resharding、failover等，可能会导致slot的位置发生变更），此时Client可能需要等待更长的时间，不过最终node会反馈MOVED信息，且信息中包含指定的新的node位置。虽然Cluster使用ID标识node，但是在MOVED信息中尽可能的暴露给客户端便于使用的ip + port。
    当Client遇到“MOVED”错误时，将会使用“CLUSTER NODES”或者“CLUSTER SLOTS”指令获取集群的最新信息，主要是nodes与slots的映射关系；因为遇到MOVED，一般也不会仅仅一个slot发生的变更，通常是一个或者多个节点的slots发生了变化，所以进行一次全局刷新是有必要的；我们还应该明白，Client将会把集群的这些信息在被缓存，以便提高query的性能。
    还有一个错误信息：“ASK”，它与“MOVED”都属于重定向错误，客户端的处理机制基本相同，只是ASK不会触发Client刷新本地的集群信息。
    
##源码原理
https://www.jianshu.com/p/0232236688c1

##redis集群核心原理:gossip通信、jedis Smart定位、主备切换
gossip
https://blog.csdn.net/qq_33814629/article/details/79904158
1、节点间的内部通信机制
1.1 基础通信原理
redis cluster节点间采取gossip协议进行通信 
维护集群的元数据有两种方式：集中式和gossip 
集中式： 
优点在于元数据的更新和读取，时效性非常好，一旦元数据出现变更立即就会更新到集中式的存储中，其他节点读取的时候立即就可以立即感知到； 
不足在于所有的元数据的更新压力全部集中在一个地方，可能导致元数据的存储压力。 
gossip： 
优点在于元数据的更新比较分散，不是集中在一个地方，更新请求会陆陆续续，打到所有节点上去更新，有一定的延时，降低了压力； 
缺点在于元数据更新有延时可能导致集群的一些操作会有一些滞后。
10000端口 
每个节点都有一个专门用于节点间通信的端口，就是自己提供服务的端口号+10000，比如7001，那么用于节点间通信的就是17001端口。 
每个节点每隔一段时间都会往另外几个节点发送ping消息，同时其他几点接收到ping之后返回pong。
1.2 gossip协议
gossip协议包含多种消息，包括ping，pong，meet，fail等等。 
meet：某个节点发送meet给新加入的节点，让新节点加入集群中，然后新节点就会开始与其他节点进行通信，不需要发送形成网络的所需的所有CLUSTER MEET命令。发送CLUSTER MEET消息以便每个节点能够达到其他每个节点只需通过一条已知的节点链就够了。由于在心跳包中会交换gossip信息，将会创建节点间缺失的链接。
ping：每个节点都会频繁给其他节点发送ping，其中包含自己的状态还有自己维护的集群元数据，互相通过ping交换元数据； 
pong: 返回ping和meet，包含自己的状态和其他信息，也可以用于信息广播和更新； 
fail: 某个节点判断另一个节点fail之后，就发送fail给其他节点，通知其他节点，指定的节点宕机了。

2、面向集群的jedis内部实现原理
2.1 基于重定向的客户端
请求重定向 
客户端会挑选任意一个redis实例去发送命令，每个redis实例接收到命令，都会计算key对应的hash slot；

查看key对应的hash slot
cluster keyslot key

连接客户端时加-c参数可以自动重定向
redis-cli -c

计算hash slot 
计算hash slot的算法，就是根据key计算CRC16值，然后对16384取模，拿到对应的hash slot； 
用hash tag可以手动指定key对应的slot，同一个hash tag下的key，都会在一个hash slot中，比如set mykey1:{100}和set mykey2:{100}；
hash slot查找 
节点间通过gossip协议进行数据交换，就知道每个hash slot在哪个节点上；
2.2 smart jedis
(1) 什么是smart jedis？ 
基于重定向的客户端，很消耗网络io，因为大部分情况下，可能都会出现一次请求重定向，才能找到正确的节点； 
所以大部分的客户端比如java redis客户端，都是jedis，都是smart的， 
本地维护一份hashslot -> node的映射表在缓存里，大部分情况下直接走本地缓存就可以找到hashslot -> node，不需要通过节点进行moved重定向；
3. 高可用性与主备切换原理
3.1 判断节点宕机
如果一个节点认为另外一个节点宕机，name就是pfail，主观宕机； 
如果多个节点都认为另外一个节点宕机了，那么就是fail，客观宕机，跟哨兵的原理几乎一样，sdown，odown； 
在cluster-node-timeout内，某个节点一直没有返回pong，那么就被认为pfail； 
如果一个节点认为某个节点pfail了，那么会在gossip ping消息中，ping给其他节点，如果超过半数的节点都认为pfail了，那么就会变成fail；
3.2 从节点过滤
对宕机的master node，从其所有的slave node中，选择一个切换成master node； 
检查每个slave node与master node断开连接的时间，如果超过了cluster-node-timeout * cluster-slave-validity-factor，那么就没有资格切换成master；
3.3 从节点选举
哨兵：对所有从节点进行排序，slave priority，offset，run id； 
每个从节点，都根据自己对master复制数据的offset，来设置一个选举时间，offset越大（复制数据越多）的从节点，选举时间越靠前，优先进行选举； 
所有的master node开始slave选举投票，给要进行选举的slave进行投票，如果大部分master node（N/2 + 1）都投票给了某个从节点，那么选举通过，那个从节点可以切换成master； 
从节点执行主备切换，从节点切换为主节点；
3.4 与哨兵比较
整个流程跟哨兵相比，非常类似，所以说，redis cluster功能强大，直接集成了replication和sentinal的功能；


##Redis源码阅读（三）集群-连接建立 
http://www.imooc.com/article/73798
对于并发请求很高的生产环境，单个Redis满足不了性能要求，通常都会配置Redis集群来提高服务性能。3.0之后的Redis支持了集群模式。
　　Redis官方提供的集群功能是无中心的，命令请求可以发送到任意一个Redis节点，如果该请求的key不是由该节点负责处理，则会返回给客户端MOVED错误，提示客户端需要转向到该key对应的处理节点上。支持集群模式的redis客户端会自动进行转向，普通模式客户端则只返回MOVED错误。

　　节点两两之间都有连接，只有主节点可以处理客户端的命令请求；从节点复制主节点数据，并在主节点下线后，升级为主节点。每个主节点可以挂多个从节点，在主节点下线后从节点需要竞争，只有一个从节点会被选举为主节点。
 
考虑以下几个关键点：
节点是如何互发现的，请求又是如何分配到各个节点的？
其中部分节点出现故障，其他节点是如何发现的又是怎样恢复的？
主节点下线后从节点是如何竞争的？
是否可以不中断Redis服务进行动态的扩容？
　　接下来几篇会从这几个关键问题入手来分析Redis集群源码；首先先看集群的基本数据结构，以及节点之间是如何建立连接的

1. 数据结构
 　　Redis集群是无中心的，每个节点会存储整个集群各个节点的信息。我们看下Redis源码中存储集群节点信息的数据结构：


clusterNode结构体存储了一个节点的基本信息，包括节点的IP，port，连接信息等；Redis节点每次和其他节点建立连接都会创建一个clusterNode用来记录其他节点的信息, 这些clusterNode都会存储到clusterState结构中，每个节点自身只拥有一个clusterState，用来存储整个集群系统的状态和信息。



2 连接建立
　　集群节点在初始化前都是孤立的Redis服务节点，还没有连成一个整体。其他节点的信息是如何被该节点获取的，整个集群是如何连接起来的呢？
　　这里有两种途径：
　　1）人为干预指定让节点和其他节点连接，也就是通过cluster meet命令来指定要连入的其他节点；
　　2）集群自发传播，靠集群内部的gossip协议自发扩散其他节点的信息。想象下如果没有集群内部的自发传播，任意两个节点间的连接都需要人为输入命令来建立；节点数如果为n, 整个集群建立的总连接数量会达到n*(n-1)；要想建立起整个集群，让每个节点都知道完整的集群信息，需要的cluster meet指令数量是O(n2)，节点多起来的话初始化的成本会很高。所以说内部自发的传播是很有必要的。

　　A节点收到cluster meet B指令后，A进入处理函数clusterCommand，并在该函数中调用clusterStartHandshake连接B服务器。这个函数实质上也只是创建一个记录了B节点信息的clusterNode(B)，并将clusterNode(B)的link置为空。真正发起连接的是集群的时间事件处理函数clusterCron。clusterCron会遍历A节点上所有的nodes，并向link为空的节点发起连接。这里的连接又用到前面介绍的文件事件机制，不再赘述。


Gossip消息扩散
　　Gossip消息的扩散是利用节点之间的ping消息，在通过meet建立连接之后为了对节点在线状态进行检测，每个节点都要对自己已知集群节点发送ping消息，如果在超时时间内返回了pong则认为节点正常在线。

　　假定对于A、B、C三个节点，初始只向A节点发送了如下两条meet指令：
　　　　Cluster meet B
　　　　Cluster meet C
　　对于A来讲，B和C都是已知的节点信息；A会向B、C分别发送ping消息；在A发送ping消息给B时，发送方A会在gossip消息体中随机带上已知的节点信息(假设包含C节点)；接收到ping消息的B节点会解析这gossip消息体中的节点信息，发现C节点是未知节点，那么就会向C节点进行握手，并建立连接。那么对B来讲，C也成为了已知节点。

　Gossip协议的原理通俗来讲就是一传十，十传百；互相之间传递集群节点信息，最终可以达到系统中所有节点都能获取到完整的集群节点。在ping消息中附加集群节点信息，带来的额外负担就是每次接收到ping消息都要预先遍历下gossip消息中所有节点信息，并判断是否有包含自身未知的节点，还要建立连接。为了减轻接收方的负担，gossip消息可以不附带所有节点信息，附带随机节点也可以最终达到所有节点都取到完整集群信息的目的。


##redis原理及实现
https://www.cnblogs.com/xiufengchen/p/10455288.html

6 redis主挂了怎么操作
　　redis提供了哨兵模式，当主挂了，可以选举其他的进行代替，哨兵模式的实现原理，就是三个定时任务监控，
　　6.1 每隔10s，每个S节点（哨兵节点）会向主节点和从节点发送info命令获取最新的拓扑结构

　　6.2 每隔2s，每个S节点会向某频道上发送该S节点对于主节点的判断以及当前Sl节点的信息，

　　　　同时每个Sentinel节点也会订阅该频道，来了解其他S节点以及它们对主节点的判断（做客观下线依据）

　　6.3 每隔1s，每个S节点会向主节点、从节点、其余S节点发送一条ping命令做一次心跳检测(心跳检测机制)，来确认这些节点当前是否可达

　　当三次心跳检测之后，就会进行投票，当超过半数以上的时候就会将该节点当做主

7 redis集群
　　redis集群在3.0以后提供了ruby脚本进行搭建，引入了糙的概念，
　　Redis集群内节点通过ping/pong消息实现节点通信，消息不但可以传播节点槽信息，还可以传播其他状态如：主从状态、节点故障等。因此故障发现也是通过消息传播机制实现的，主要环节包括：主观下线（pfail）和客观下线（fail）

　　主客观下线：

　　主观下线：集群中每个节点都会定期向其他节点发送ping消息，接收节点回复pong消息作为响应。如果通信一直失败，则发送节点会把接收节点标记为主观下线（pfail）状态。
　　
　　客观下线：超过半数，对该主节点做客观下线

　　主节点选举出某一主节点作为领导者，来进行故障转移。

　　故障转移（选举从节点作为新主节点）

8 内存淘汰策略
Redis的内存淘汰策略是指在Redis的用于缓存的内存不足时，怎么处理需要新写入且需要申请额外空间的数据。

noeviction：当内存不足以容纳新写入数据时，新写入操作会报错。

allkeys-lru：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的key。

allkeys-random：当内存不足以容纳新写入数据时，在键空间中，随机移除某个key。

volatile-lru：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，移除最近最少使用的key。

volatile-random：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，随机移除某个key。

volatile-ttl：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，有更早过期时间的key优先移除。

9 缓存击穿的解决方案：
　　原因：就是别人请求数据的时候，很多数据在缓存中无法查询到，直接进入数据查询，
　　解决方法，对相关数据进行查询的数据只查询缓存，如果是一些特殊的可以进行数据库查询，
　　也可以采用布隆过滤器进行查询

10缓存雪崩的解决方案：
　　缓存雪崩的原因：一次性加入缓存的数据过多，导致内存过高，从而影响内存的使用导致服务宕机
　　解决方法：
　　1 redis集群，通过集群方式将数据放置
　　2 后端服务降级和限流：当一个接口请求次数过多，那么就会添加过多数据，可以对服务进行限流，限制访问的数量，这样就可以减少问题的出现

##Redis自己的事件模型 ae
https://www.cnblogs.com/shijingxiang/articles/5369224.html

##raft
http://ifeve.com/raft/
http://ifeve.com/解读raft协议（一-算法基础）/
http://ifeve.com/解读raft（二-选举和日志复制）/
https://www.jianshu.com/p/a5d37ca84f5e
https://www.cnblogs.com/lushilin/p/9268969.html

RAFT算法随想
https://blog.csdn.net/hfty290/article/details/75331948#comments

解读Raft（三 安全性）
https://blog.csdn.net/weixin_33778778/article/details/89588200

##Redis的缓存淘汰策略LRU与LFU
https://www.jianshu.com/p/c8aeb3eee6bc

##Redis基础 常用类型 时间复杂度
https://blog.csdn.net/andy86869/article/details/88366513

RDB
采用RDB 的持久化方式，redis 会定期保存数据快照到一个 rdb 文件中，并在启动时自动加载 rdb文件，恢复之前保存的数据，可以在 redis.conf 中配置 redis 进行快照保存的时机
save [ seconds ] [changes]
意思是在 seconds 秒内如果发生了 changes 次数据修改，则进行一次 RDB 快照保存 
eg.  save 60 100

在 conf 文件中 可以配置多条 save 策略
save 900 1 
save 300 10 
save 60 10000

还可以通过手工 bgsave 命令触发 rdb 快照保存
下面说说 rdb 的优点
对性能影响小 子进程 操作 rdb
每次生成一个完整的 rdb 文件，作为非常可靠的灾难恢复手段
rdb文件恢复数据比 aof 快很多

RDB 缺点是
快照是定期生成的，所以可能干好在 两次 rdb 之间 redis 挂了，那么这个自从上一次到这一次之间的数据，或多或少地会丢失部分数据
如果数据集非常的大 而且 CPU 不够强，Redis 是 fork 子进程时可能会消耗相对较长的时间，因此影响这期间客户端的请求。

AOF
采用 aof 持久化方式，redis 会把每一次的写请求都放在一个日志文件里，redis 重启的时候，会把 aof 文件中的记录的所有的写操作顺序的执行一遍，确保数据恢复到最新
aof 默认是关闭的，需要开启进行如下配置
appendonly yes

aof 提供了 3 中 fsync 配置 always 、everysec 、no
appendfsync no 不进行 fsync 将 flush 文件的时机交给 os 决定，速度最快
appendfsync always 每写入一条日志就进行一次的 fsync 操作，数据安全性最高，单数速度最慢
appendfsync 折中的做法，交给后台线程每秒 fsync 一次

随着 aof 不断的记录写操作日子，必定会出现一些无用的日志，例如某个时间点执行了 set key1 “abc” ，在此之后又执行了 set key1 “bcd”，那么第一条命令显然是没有用的，大量的无用日志会使得 aof 文件过大，也会让数据恢复的时间加长。基于这种情况，redis 提供了 aof rewrite 功能，只保留能够把书恢复到最新状态的最小写操作集。

aof rewrite 命令 可以通过 bgrewriteaof 命令触发，也可以通过 redis 定期自动执行。
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min--size 64mb

两行的含义是  reids 在每次 aof  rewrite 时，会记录rewrite 后 aof 的大小，当 aof 日志
	在该基础上增长了 100% 后，自动进行 aof rewrite 操作，同时如果增长的大小没有达到 64mb
	 则不胡 rewrite

aof 的优点
安全性很高， appendfsync always 时候，任何写入的数据都不会丢失，但是这个策略一般不会使用，太慢了
aof 在发生断电的时候也不会损坏，即使出现某条日志写入一半的情况，也可以使用 redis-check-aof 工具轻松修复
aof 文件易读
aof 的缺点
aof 的文件通常都比 rdb 文件大
性能消耗比 rdb 高
数据恢复比 rdb 慢

##Redis EXISTS命令耗时过长case排查
https://blog.csdn.net/chosen0ne/article/details/50543335

##redis慢查询分析
https://blog.csdn.net/kwy15732621629/article/details/79131137

##使用 Redis 的 slowlog get [n] 慢查询日志彻底解决生产问题！
https://blog.csdn.net/weixin_44018338/article/details/99460667

##Redis 性能问题排查：slowlog 和排队延时
https://blog.csdn.net/weixin_33815613/article/details/92370024

##第三十五章：Redis持久化方案
https://zhuanlan.zhihu.com/p/44196874

2）RDB 持久化条件
格式：save <seconds> <changes>
示例：
save 900 1 ： 表示15分钟（900秒钟）内至少1个键被更改则进行快照。
save 300 10 ： 表示5分钟（300秒）内至少10个键被更改则进行快照。
save 60 10000 ：表示1分钟内至少10000个键被更改则进行快照。
特别说明：
Redi s启动后会读取 RDB 快照文件，将数据从硬盘载入到内存。
根据数据量大小与结构和服务器性能不同，这个时间也不同。通常将记录一千万个字符串类型键、大小为 1GB 的快照文件载入到内存中需要花费 20～30 秒钟。
2）快照的实现原理
（1）Redis 使用 fork 函数复制一份当前进程的副本(子进程)
（2）父进程继续接收并处理客户端发来的命令，而子进程开始将内存中的数据写入硬盘中的临时文件。
（3）当子进程写入完所有数据后会用该临时文件替换旧的RDB文件，至此，一次快照操作完成。

2、AOF 方式
1）介绍
（1）默认情况下 Redis 没有开启 AOF（append only file）方式的持久化
（2）开启 AOF 持久化后每执行一条会更改 Redis 中的数据的命令，Redis 就会将该命令写入硬盘中的 AOF 文件，这一过程显然会降低 Redis 的性能，但大部分情况下这个影响是能够接受的，另外使用较快的硬盘可以提高 AOF 的性能（固态硬盘）。

2）AOF 重写原理（优化 AOF 文件）
（1）Redis 可以在 AOF 文件体积变得过大时，自动地在后台对 AOF进行重写，重写后的新 AOF 文件包含了恢复当前数据集所需的最小命令集合。 
（2）整个重写操作是绝对安全的，因为 Redis 在创建新 AOF 文件的过程中，会继续将命令追加到现有的 AOF 文件里面，即使重写过程中发生停机，现有的 AOF 文件也不会丢失。 而一旦新 AOF 文件创建完毕，Redis 就会从旧 AOF 文件切换到新 AOF 文件，并开始对新 AOF 文件进行追加操作。
（3）AOF 文件有序地保存了对数据库执行的所有写入操作， 这些写入操作以 Redis 协议的格式保存， 因此 AOF 文件的内容非常容易被人读懂， 对文件进行分析（parse）也很轻松。

3）参数说明
（1）auto-aof-rewrite-percentage 100 表示当前 aof 文件大小超过上一次 aof 文件大小的百分之多少的时候会进行重写。如果之前没有重写过，以启动时 aof 文件大小为准。
（2）auto-aof-rewrite-min-size 64mb 限制允许重写最小 aof 文件大小，也就是文件大小小于 64mb 的时候，不需要进行优化。

4）同步磁盘数据
Redis 每次更改数据的时候， aof 机制都会将命令记录到 aof 文件，但是实际上由于操作系统的缓存机制，数据并没有实时写入到硬盘，而是进入硬盘缓存。再通过硬盘缓存机制去刷新到保存到文件。

（1）appendfsync always 每次执行写入都会进行同步 ， 这个是最安全但是是效率比较低的方式
（2）appendfsync everysec 每一秒执行
（3）appendfsync no 不主动进行同步操作，由操作系统去执行，这个是最快但是最不安全的方式

5）AOF 文件损坏以后如何修复
服务器可能在程序正在对 AOF 文件进行写入时停机， 如果停机造成了 AOF 文件出错（corrupt）， 那么 Redis 在重启时会拒绝载入这个 AOF 文件， 从而确保数据的一致性不会被破坏。
当发生这种情况时， 可以用以下方法来修复出错的 AOF 文件：
（1）为现有的 AOF 文件创建一个备份。
（2）使用 Redis 附带的 redis-check-aof 程序，对原来的 AOF 文件进行修复。
redis-check-aof --fix

3. 重启 Redis 服务器，等待服务器载入修复后的 AOF 文件，并进行数据恢复。
6）如何选择 RDB 和 AOF
（1）一般来说,如果对数据的安全性要求非常高的话，应该同时使用两种持久化功能。
如果可以承受数分钟以内的数据丢失，那么可以只使用 RDB 持久化。
（2）有很多用户都只使用 AOF 持久化， 但并不推荐这种方式： 因为定时生成 RDB 快照（snapshot）非常便于进行数据库备份， 并且 RDB 恢复数据集的速度也要比 AOF 恢复的速度要快 。
两种持久化策略可以同时使用，也可以使用其中一种。如果同时使用的话， 那么 Redis 重启时，会优先使用AOF文件来还原数据。

##REDIS持久化之RDB和AOF的区别
https://www.jianshu.com/p/1d9ab6bc0835

##redis的持久化方式RDB和AOF的区别
https://www.jianshu.com/p/425b8530ae76

#codis
##[codis]https://github.com/CodisLabs/codis/blob/release3.2/doc/tutorial_zh.md

##Codis与Redis Cluster集群方案对比
https://blog.csdn.net/tiandao321/article/details/88353128

##Codis与RedisCluster的原理详解 
https://www.cnblogs.com/pingyeaa/p/11294773.html

##redis4.0、codis、阿里云redis 3种redis集群对比分析
https://blog.csdn.net/weixin_33701617/article/details/90585083

##高效运维最佳实践（03）：Redis 集群技术及 Codis 实践
https://www.infoq.cn/article/effective-ops-part-03/

##最详细的Codis集群扩容方法
https://blog.csdn.net/wyc_cs/article/details/51023338

##基于Codis的Redis分布式缓存实现
https://blog.csdn.net/fuyuwei2015/article/details/71131780

##用Codis实现Redis分布式集群
https://blog.csdn.net/openbox2008/article/details/80033133

三、角色分批

zookeeper集群:
10.10.0.47
10.10.0.48
10.10.1.76

codis-config、codis-ha：
10.10.32.10:18087

codis-proxy：
10.10.32.10:19000
10.10.32.49:19000

codis-server：
10.10.32.42：6379、10.10.32.43：6380（主、从）
10.10.32.43：6379、10.10.32.44：6380（主、从）
10.10.32.44：6379、10.10.32.42：6380（主、从）

七、关于监控
关于整个codis集群的监控，我们这边用的是zabbix，监控的指标比较简单，所以这块大家有什么好的建议多给我提提哈
zookeeper：监控各个节点的端口联通性（以后想着把进程也监控上）
codis-proxy：监控了端口的联通性，这个监控远远不够呀
codis-server：监控了内存使用率、连接数、联通性
codis-ha：监控进程
dashboard：监控端口联通性

八、使用过程中遇到的问题
1、codis-proxy的日志切割，codis-proxy的默认日志级别是info，日志量很大，我们这边每天产生50多G日志，目前codis-proxy还不支持热重启，想修改启动参数还是比较麻烦的，日志切割推荐用logrotate
2、codis-proxy的监听地址默认没有具体ipv4，也就是codis-proxy启动之后没有0.0.0.0:19000这样的监听，这样会导致的问题就是前端lvs没有办法负载均衡codis-proxy，不能转发请求过，这个问题已联系作者处理了，在codis-proxy启动的配置文件中加上proto=tcp4这个参数就支持监听ipv4了
3、添加 Redis Server Group的时候，非codis-server（原生的redis）竟然也能加入到codis集群里面，在redis和codis-server共存在一个物理机上的清楚，很容易加错，希望能有个验证，非codis-server不能加入到codis集群
4、codis集群内部通讯是通过主机名的，如果主机名没有做域名解析那dashboard是通过主机名访问不到proxy的http-addr地址的，这会导致从web界面上看不到 OP/s的数据，至于还有没有其他问题，目前我这边还没有发现，建议内部通讯直接用内网IP

test:
root      25248  25231  0 5月06 ?       04:22:30 codis-dashboard -l log/dashboard.log -c dashboard.toml --host-admin 192.168.11.71:28080
root      25398  25381  0 5月06 ?       02:27:53 codis-proxy -l log/proxy.log -c proxy.toml --host-admin 192.168.11.71:21080 --host-proxy 192.168.11.71:29000
root      25514  25497  0 5月06 ?       01:01:09 codis-server *:6379
root      25613  25596  0 5月06 ?       01:00:53 codis-server *:6379
root      25712  25695  0 5月06 ?       01:01:07 codis-server *:6379
root      25885  25866  0 5月06 ?       00:39:42 codis-server *:6379
root      25998  25981  0 5月06 ?       00:10:12 codis-fe -l log/fe.log --zookeeper 192.168.11.71:2181 --listen=0.0.0.0:8080 --assets=/gopath/src/github.com/CodisLabs/codis/bin/assets

##Codis AutoRebalance 流程学习
https://zhuanlan.zhihu.com/p/53044266

疑问：Dashboard 和 Proxy 到底是如何同步状态的呢，会不会出现状态不一致的情况？

如果 A 接收到了某个 key 的请求，A 会将数据从 From 同步到 Target，然后将 From 上的 key 删除，然后在 Target 上执行命令。如果 B 后来又接收到了同一个 key 的请求，由于没有同步 slot 信息，还是会在 From 上执行命令，但是此时这个 Key 已经不存在了，那么这个请求就岂不是会报错？
实际情况下是不会出现这个问题的，Codis 在做迁移时，分成了多个阶段，主要是 Preparing 阶段、Prepared 阶段、Migrating 阶段。Proxy 会先在 Preparing 阶段将对应的 slot 的请求阻塞起来，如果在这个阶段有 Proxy 服务同步信息失败，那么 Dashboard 会回滚状态，迁移也就不会往下进行了。

当确保所有的 Proxy 都将请求阻塞住的时候，才会开始进入 Prepared 阶段，这个阶段各个 Proxy 会获取 From 和 Target 信息，这个阶段在遇到 slot 的请求，Proxy 才会执行上面的 SLOTSMGRTAGONE 命令来迁移单个 key。所以不会出现上面所说的场景。

疑问：Codis 是如何保证避免单点故障的？
Proxy 是无状态的，可以轻松扩容，多实例保证可用性。
Dashboard 同一时间只能有一个实例，可以借助 zookeeper、etcd 来部署主备节点。
Redis Group 则可以使用 Sentinel 和 Redis 主从节点来保证可用性。

安装
安装codis的编译环境需要有 golang 、git 环境。运行环境需要依赖有可用的zookeeper
 
一、依赖环境安装
1、安装编译环境前的一些依赖安装
yum install mercurial
yum install git
yum install gcc
 
以上这些命令在正常的情况下执行都没有问题，但在安装git的过程中遇到了一些问题，导致使用这些命令安装不成功，在备用参考命令里有，没有使用yum安装git。
 
2、安装go环境(一定要用1.3.1的)
wget https://storage.googleapis.com/golang/go1.3.1.linux-amd64.tar.gz
curl -O -L https://storage.googleapis.com/golang/go1.3.1.linux-amd64.tar.gz
tar -C /usr/local -xzf go1.3.1.linux-amd64.tar.gz
 
修改配置文件
vi /etc/profile
 
profile文件修改如下：
export GOROOT=/usr/local/go
export PATH=$GOROOT/bin:$PATH
export GOPATH=/data/gopkg
 
刷新配置文件
source /etc/profile
 
3、安装zookeeper的方式
wget http://mirrors.cnnic.cn/apache/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz
tar -C /usr/local -xzf zookeeper-3.4.6.tar.gz
go get github.com/wandoulabs/codis
cd get /usr/local/zookeeper-3.4.6/conf
cp zoo_sample.cfg zoo.cfg
然后启动zookeeper
cp ../bin/
./zkServer.sh start
安装到这里，依赖环境已经全部完成，下面就可以惊心动魄的codis安装，其实搞起来也很简单，熟练之后十几秒就能搞定。
 
二、安装codis
go get github.com/wandoulabs/codis
cd $GOPATH/src/github.com/wandoulabs/codis #$gopath改成自己的目录即可.
./bootstrap.sh
make gotest

##Codis 3.2 部署配置汇总
服务端：codis-fe------codis-dashboard------codis-proxy------codis-group------codis-server
客户端：client------nginx-tcp------codis-proxy
cdis-fe可以管理多个codis-dashboard
每个codis-dashboard代表一个产品线，每个codis-dashboard可以管理多个codis-proxy
每个codis-proxy可以管理多个codis-server group
每个codis-server group至少由两个codis-server组成，最少1主1备
由上可知一个大的codis集群可以分多个产品线，客户端连接各个产品线的codis-proxy，业务线之间可以做到物理隔离，比如group1，group2，group3分给codis-product1业务线，group4，
group5，group6分给codis-product2业务线，codis-dashboard配置保存在zookeeper里。
特别注意
同一个codis-server加入多个codis-dashboard的codis-group里，但是在不同的codis-dashboard里面主备的角色要一致，这代表逻辑隔离。
同一个codis-server只加入唯一的codis-dashboard的codis-group里，这代表物理隔离。

一，Codis简介
Codis 是一个分布式 Redis 解决方案, 对于上层的应用来说, 连接到 Codis Proxy 和连接原生的 Redis Server 没有显著区别 (不支持的命令列表), 上层应用可以像使用单机的 Redis 一
样使用, Codis 底层会处理请求的转发, 不停机的数据迁移等工作, 所有后边的一切事情, 对于前面的客户端来说是透明的, 可以简单的认为后边连接的是一个内存无限大的 Redis 服务。
不支持命令列表
https://github.com/CodisLabs/codis/blob/release3.2/doc/unsupported_cmds.md
redis的修改
https://github.com/CodisLabs/codis/blob/release3.2/doc/redis_change_zh.md
go安装
https://golang.org/doc/install

二，Codis 3.x
最新 release 版本为 codis-3.2，codis-server 基于 redis-3.2.8
支持 slot 同步迁移、异步迁移和并发迁移，对 key 大小无任何限制，迁移性能大幅度提升
相比 2.0：重构了整个集群组件通信方式，codis-proxy 与 zookeeper 实现了解耦，废弃了codis-config 等
元数据存储支持 etcd/zookeeper/filesystem 等，可自行扩展支持新的存储，集群正常运行期间，即便元存储故障也不再影响 codis 集群，大大提升 codis-proxy 稳定性
对 codis-proxy 进行了大量性能优化,通过控制GC频率、减少对象创建、内存预分配、引入 cgo、jemalloc 等，使其吞吐还是延迟，都已达到 codis 项目中最佳
proxy 实现 select 命令，支持多 DB
proxy 支持读写分离、优先读同 IP/同 DC 下副本功能
基于 redis-sentinel 实现主备自动切换
实现动态 pipeline 缓存区（减少内存分配以及所引起的 GC 问题）
proxy 支持通过 HTTP 请求实时获取 runtime metrics，便于监控、运维
支持通过 influxdb 和 statsd 采集 proxy metrics
slot auto rebalance 算法从 2.0 的基于 max memory policy 变更成基于 group 下 slot 数量
提供了更加友好的 dashboard 和 fe 界面，新增了很多按钮、跳转链接、错误状态等，有利于快速发现、处理集群故障
新增 SLOTSSCAN 指令，便于获取集群各个 slot 下的所有 key
codis-proxy 与 codis-dashbaord 支持 docker 部署

三，Codis 3.x 由以下组件组成：
Codis Server：基于 redis-3.2.8 分支开发。增加了额外的数据结构，以支持 slot 有关的操作以及数据迁移指令。具体的修改可以参考文档 redis 的修改。

Codis Proxy：客户端连接的 Redis 代理服务, 实现了 Redis 协议。 除部分命令不支持以外(不支持的命令列表)，表现的和原生的 Redis 没有区别（就像 Twemproxy）。
对于同一个业务集群而言，可以同时部署多个 codis-proxy 实例；

不同 codis-proxy 之间由 codis-dashboard 保证状态同步。

Codis Dashboard：集群管理工具，支持 codis-proxy、codis-server 的添加、删除，以及据迁移等操作。在集群状态发生改变时，codis-dashboard 维护集群下所有 codis-proxy 的状态的一致性。

对于同一个业务集群而言，同一个时刻 codis-dashboard 只能有 0个或者1个；
所有对集群的修改都必须通过 codis-dashboard 完成。

Codis Admin：集群管理的命令行工具。
可用于控制 codis-proxy、codis-dashboard 状态以及访问外部存储。

Codis FE：集群管理界面。
多个集群实例共享可以共享同一个前端展示页面；
通过配置文件管理后端 codis-dashboard 列表，配置文件可自动更新。

Storage：为集群状态提供外部存储。
提供 Namespace 概念，不同集群的会按照不同 product name 进行组织；
目前仅提供了 Zookeeper、Etcd、Fs 三种实现，但是提供了抽象的 interface 可自行扩展。

##Codis 3.2 集群搭建与测试
https://blog.51cto.com/brucewang/2159131

二、搭建的步骤

一）、物理硬件需求
二）、初始化服务环境
三）、部署zookeeper集群
四）、部署codis-dashboard
五）、部署codis-fe管理后台
六）、部署codis-server加入集群
七）、部署codis-proxy代理服务
八）、部署redis-sentinel实现集群HA
九）、利用redis-benchmark进行集群压测
十）、codis的一些常见问题以及理解


一）、物理硬件需求
    对于测试的环境来说，其实一台就可以2c4g的机器就可以，没什么需求。但我想用于生产环境，所以需求又四台4c8g的机器来搭建，具体需求的算法根据业务的数据存储来算，当然也跟业务的繁忙度有关系。我的机器配置如下：
10.21.1.206
4C8G
10.21.1.207
4C8G
10.21.1.208
4C8G
10.21.1.209
4C8G

### 二）、初始化服务环境
   需要的环境就一个java环境，这个是zookeeper的依赖，也不是必须的，如果搭建使用的是默认的filesystem，那java环境也不需要了。codis的建议用我提供的实例，省得自己去整合脚本，配置文件，各个实例下载后安装的目录如下:
zookeeper
/usr/local/zookeeper/
codis
/usr/local/codis/
java
/usr/local/java/

### 三）、部署zookeeper集群

10.21.1.206
10.21.1.207
10.21.1.209

1、设置配置文件 /usr/local/zookeeper/conf/zoo.cfg
[root@localhost zookeeper]# cat conf/zoo.cfg
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/data/zookeeper/data
dataLogDir=/data/zookeeper/log
clientPort=2181
maxClientCnxns=300
server.1=10.21.1.206:2888:3888
server.2=10.21.1.207:2888:3888
server.3=10.21.1.208:2888:3888

2、设置zookeeper存储目录
mkdir -p /data/zookeeper/log
mkdir -p /data/zookeeper/data


3、设置集群ID

IP
myid位置
内容
10.21.1.206
/data/zookeeper/data/myid
1
10.21.1.207
/data/zookeeper/data/myid
2
10.21.1.208
/data/zookeeper/data/myid
3

4、启动zookeeper集群

在三个机器分别启动zookeeper
 /usr/local/zookeeper/bin/zkServer.sh start


5、测试启动的zookeeper是否成功
 查看zookeeper状态
[root@localhost bin]# ./zkServer.sh status
/usr/local/java/bin/java
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper/bin/../conf/zoo.cfg
Client port found: 2181. Client address: localhost.
Mode: leader

 进入zookeeper
[root@localhost bin]# ./zkCli.sh 
........
[zk: localhost:2181(CONNECTED) 0]  ls /

###四）、部署codis-dashboard

10.21.1.206
   
1、修改配置文件
 vi /usr/local/codis/conf/dashboard.toml
coordinator_name = "zookeeper"
coordinator_addr = "10.21.1.206:2181,10.21.1.207:2181,10.21.1.208:2181"

    
   默认为filesystem的Storage，修改成使用zookeeper。

2、启动dashboard
 cd /usr/local/codis
  ./admin/codis-dashboard-admin.sh start
 ss -tpnl |grep  18080
LISTEN     0      128         :::18080                   :::*                   users:(("codis-dashboard",pid=1017,fd=6))


3、查看日志排查错误
 tail -n 200 /usr/local/codis/log/codis-dashboard.out
 tail -n 200  /usr/local/codis/log/codis-dashboard.log.2018-08-09


### 五）、10.21.206上部署codis-fe管理后台

0、部署机器
10.21.1.206

1、修改启动文件
 修改存储文件成zookeeper方式
 vi /usr/local/codis/codis-fe-admin.sh
COORDINATOR_NAME="zookeeper"
COORDINATOR_ADDR="10.21.1.206:2181,10.21.1.207:2181,10.21.1.208:2181"


2、启动fe
 /usr/local/codis/admin/codis-fe-admin.sh start
 ss -tpnl |grep 9090
LISTEN     0      128         :::9090                    :::*                   users:(("codis-fe",pid=1072,fd=6))


3、打开后台查看

### 六）、部署codis-server加入集群

0、部署机器
10.21.1.206
10.21.1.207
10.21.1.208
10.21.1.209

1、在四台机器都启动codis-server
/usr/local/codis/admin/codis-server-admin.sh start


2、在fe后台添加两个Group，每个group分配两个机器

### 八）、部署redis-sentinel实现集群HA

0、部署机器

10.21.1.206
10.21.1.207
10.21.1.208

1、修改配置文件
port 26379
dir "/tmp"
protected-mode no


2、启动sentinel
sentinel部署官方脚本，是根据codis-server启动脚本进行修改
/usr/local/codis/admin/codis-sentinel-admin.sh start

 ss -tpnl |grep 26379


3、fe界面添加Sentinels

4、点下SYNC,这样配置文件会自动添加如下内容
 Generated by CONFIG REWRITE
sentinel myid ea5a5c3613f12dbddf469851ecacef8dba60dc23
sentinel monitor codis-demo-2 10.21.1.208 6379 2
sentinel failover-timeout codis-demo-2 300000
sentinel config-epoch codis-demo-2 0
sentinel leader-epoch codis-demo-2 0
sentinel known-slave codis-demo-2 10.21.1.209 6379
sentinel known-sentinel codis-demo-2 10.21.1.207 26379 0757848854fba1295387b9d5b5ca49178e73ebd6
sentinel known-sentinel codis-demo-2 10.21.1.208 26379 7c8ee4b48e5dbbe49f3f999b8746caf0c18e4565
sentinel monitor codis-demo-1 10.21.1.206 6379 2
sentinel failover-timeout codis-demo-1 300000
sentinel config-epoch codis-demo-1 2
sentinel leader-epoch codis-demo-1 2
sentinel known-slave codis-demo-1 10.21.1.207 6379
sentinel known-sentinel codis-demo-1 10.21.1.207 26379 0757848854fba1295387b9d5b5ca49178e73ebd6
sentinel known-sentinel codis-demo-1 10.21.1.208 26379 7c8ee4b48e5dbbe49f3f999b8746caf0c18e4565
sentinel current-epoch 2


九）、利用redis-benchmark进行集群压测

##Codis3.2集群HA高可用方案

https://www.jianshu.com/p/68552828ef8a

