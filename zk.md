
#深入浅出Zookeeper（一） Zookeeper架构及FastLeaderElection机制  http://www.jasongj.com/zookeeper/fastleaderelection/
##Zookeeper是什么
Zookeeper是一个分布式协调服务，可用于服务发现，分布式锁，分布式领导选举，配置管理等。
这一切的基础，都是Zookeeper提供了一个类似于Linux文件系统的树形结构（可认为是轻量级的内存文件系统，但只适合存少量信息，完全不适合存储大量文件或者大文件），同时提供了对于每个节点的监控与通知机制。
既然是一个文件系统，就不得不提Zookeeper是如何保证数据的一致性的。本文将介绍Zookeeper如何保证数据一致性，如何进行领导选举，以及数据监控/通知机制的语义保证。
##Zookeeper架构 角色
Zookeeper集群是一个基于主从复制的高可用集群，每个服务器承担如下三种角色中的一种
Leader 一个Zookeeper集群同一时间只会有一个实际工作的Leader，它会发起并维护与各Follwer及Observer间的心跳。所有的写操作必须要通过Leader完成再由Leader将写操作广播给其它服务器。
Follower 一个Zookeeper集群可能同时存在多个Follower，它会响应Leader的心跳。Follower可直接处理并返回客户端的读请求，同时会将写请求转发给Leader处理，并且负责在Leader处理写请求时对请求进行投票。
Observer 角色与Follower类似，但是无投票权。

##FastLeaderElection原理
术语介绍
myid
每个Zookeeper服务器，都需要在数据文件夹下创建一个名为myid的文件，该文件包含整个Zookeeper集群唯一的ID（整数）。例如某Zookeeper集群包含三台服务器，hostname分别为zoo1、zoo2和zoo3，其myid分别为1、2和3，则在配置文件中其ID与hostname必须一一对应，如下所示。在该配置文件中，server.后面的数据即为myid
server.1=zoo1:2888:3888
server.2=zoo2:2888:3888
server.3=zoo3:2888:3888
zxid
类似于RDBMS中的事务ID，用于标识一次更新操作的Proposal ID。为了保证顺序性，该zkid必须单调递增。因此Zookeeper使用一个64位的数来表示，高32位是Leader的epoch，从1开始，每次选出新的Leader，epoch加一。低32位为该epoch内的序号，每次epoch变化，都将低32位的序号重置。这样保证了zkid的全局递增性。
###支持的领导选举算法
可通过electionAlg配置项设置Zookeeper用于领导选举的算法。
到3.4.10版本为止，可选项有 
0 基于UDP的LeaderElection
1 基于UDP的FastLeaderElection
2 基于UDP和认证的FastLeaderElection
3 基于TCP的FastLeaderElection
在3.4.10版本中，默认值为3，也即基于TCP的FastLeaderElection。另外三种算法已经被弃用，并且有计划在之后的版本中将它们彻底删除而不再支持。
###FastLeaderElection
FastLeaderElection选举算法是标准的Fast Paxos算法实现，可解决LeaderElection选举算法收敛速度慢的问题。
服务器状态
LOOKING 不确定Leader状态。该状态下的服务器认为当前集群中没有Leader，会发起Leader选举
FOLLOWING 跟随者状态。表明当前服务器角色是Follower，并且它知道Leader是谁
LEADING 领导者状态。表明当前服务器角色是Leader，它会维护与Follower间的心跳
OBSERVING 观察者状态。表明当前服务器角色是Observer，与Folower唯一的不同在于不参与选举，也不参与集群写操作时的投票
选票数据结构
每个服务器在进行领导选举时，会发送如下关键信息
logicClock 每个服务器会维护一个自增的整数，名为logicClock，它表示这是该服务器发起的第多少轮投票
state 当前服务器的状态
self_id 当前服务器的myid
self_zxid 当前服务器上所保存的数据的最大zxid
vote_id 被推举的服务器的myid
vote_zxid 被推举的服务器上所保存的数据的最大zxid
投票流程
自增选举轮次
Zookeeper规定所有有效的投票都必须在同一轮次中。每个服务器在开始新一轮投票时，会先对自己维护的logicClock进行自增操作。
初始化选票
每个服务器在广播自己的选票前，会将自己的投票箱清空。该投票箱记录了所收到的选票。例：服务器2投票给服务器3，服务器3投票给服务器1，则服务器1的投票箱为(2, 3), (3, 1), (1, 1)。票箱中只会记录每一投票者的最后一票，如投票者更新自己的选票，则其它服务器收到该新选票后会在自己票箱中更新该服务器的选票。
发送初始化选票
每个服务器最开始都是通过广播把票投给自己。
接收外部投票
服务器会尝试从其它服务器获取投票，并记入自己的投票箱内。如果无法获取任何外部投票，则会确认自己是否与集群中其它服务器保持着有效连接。如果是，则再次发送自己的投票；如果否，则马上与之建立连接。
判断选举轮次
收到外部投票后，首先会根据投票信息中所包含的logicClock来进行不同处理
外部投票的logicClock大于自己的logicClock。说明该服务器的选举轮次落后于其它服务器的选举轮次，立即清空自己的投票箱并将自己的logicClock更新为收到的logicClock，然后再对比自己之前的投票与收到的投票以确定是否需要变更自己的投票，最终再次将自己的投票广播出去。
外部投票的logicClock小于自己的logicClock。当前服务器直接忽略该投票，继续处理下一个投票。
外部投票的logickClock与自己的相等。当时进行选票PK。
选票PK
选票PK是基于(self_id, self_zxid)与(vote_id, vote_zxid)的对比
外部投票的logicClock大于自己的logicClock，则将自己的logicClock及自己的选票的logicClock变更为收到的logicClock
若logicClock一致，则对比二者的vote_zxid，若外部投票的vote_zxid比较大，则将自己的票中的vote_zxid与vote_myid更新为收到的票中的vote_zxid与vote_myid并广播出去，另外将收到的票及自己更新后的票放入自己的票箱。如果票箱内已存在(self_myid, self_zxid)相同的选票，则直接覆盖
若二者vote_zxid一致，则比较二者的vote_myid，若外部投票的vote_myid比较大，则将自己的票中的vote_myid更新为收到的票中的vote_myid并广播出去，另外将收到的票及自己更新后的票放入自己的票箱
统计选票
如果已经确定有过半服务器认可了自己的投票（可能是更新后的投票），则终止投票。否则继续接收其它服务器的投票。
更新服务器状态
投票终止后，服务器开始更新自身状态。若过半的票投给了自己，则将自己的服务器状态更新为LEADING，否则将自己的状态更新为FOLLOWING


#[Zookeeper一致性协议原理Zab](https://www.cnblogs.com/hongdada/p/8145075.html)
 ZooKeeper为高可用的一致性协调框架，自然的ZooKeeper也有着一致性算法的实现，ZooKeeper使用的是ZAB协议作为数据一致性的算法， ZAB（ZooKeeper Atomic Broadcast ） 全称为：原子消息广播协议；
ZAB可以说是在Paxos算法基础上进行了扩展改造而来的，ZAB协议设计了支持崩溃恢复，ZooKeeper使用单一主进程Leader用于处理客户端所有事务请求，采用ZAB协议将服务器数状态以事务形式广播到所有Follower上；
由于事务间可能存在着依赖关系，ZAB协议保证Leader广播的变更序列被顺序的处理，：一个状态被处理那么它所依赖的状态也已经提前被处理；
ZAB协议支持的崩溃恢复可以保证在Leader进程崩溃的时候可以重新选出Leader并且保证数据的完整性;
在ZooKeeper中所有的事务请求都由一个主服务器也就是Leader来处理，其他服务器为Follower，Leader将客户端的事务请求转换为事务Proposal，并且将Proposal分发给集群中其他所有的Follower，然后Leader等待Follwer反馈，当有 过半数（>=N/2+1） 的Follower反馈信息后，Leader将再次向集群内Follower广播Commit信息，Commit为将之前的Proposal提交;

##ZooKeeper从以下几点保证了数据的一致性
① 顺序一致性
来自任意特定客户端的更新都会按其发送顺序被提交。也就是说，如果一个客户端将Znode z的值更新为a，在之后的操作中，它又将z的值更新为b，则没有客户端能够在看到z的值是b之后再看到值a（如果没有其他对z的更新）。
② 原子性
每个更新要么成功，要么失败。这意味着如果一个更新失败，则不会有客户端会看到这个更新的结果。
③ 单一系统映像
一个客户端无论连接到哪一台服务器，它看到的都是同样的系统视图。这意味着，如果一个客户端在同一个会话中连接到一台新的服务器，它所看到的系统状态不会比在之前服务器上所看到的更老。当一台服务器出现故障，导致它的一个客户端需要尝试连接集合体中其他的服务器时，所有滞后于故障服务器的服务器都不会接受该连接请求，除非这些服务器赶上故障服务器。
④ 持久性
一个更新一旦成功，其结果就会持久存在并且不会被撤销。这表明更新不会受到服务器故障的影响。
==========================================================================

##ZAB协议的两个基本模式：恢复模式和广播模式
恢复模式:（选举）
当服务启动或者在领导者崩溃后，Zab就进入了恢复模式，当领导者被选举出来，且大多数server完成了和leader的状态同步以后，恢复模式就结束了。状态同步保证了leader和server具有相同的系统状态。
具体选举看下面文章
http://www.jasongj.com/zookeeper/fastleaderelection/
崩溃恢复过程中，为了保证数据一致性需要处理特殊情况：
1、已经被leader提交的proposal确保最终被所有的服务器follower提交
2、确保那些只在leader被提出的proposal被丢弃
针对这个要求,如果让leader选举算法能够保证新选举出来的Leader服务器拥有集群中所有机器最高的ZXID事务proposal，就可以保证这个新选举出来的Leader一定具有所有已经提交的提案，也可以省去Leader服务器检查proposal的提交与丢弃的工作。
 
广播模式：（数据同步）
一旦Leader已经和多数的Follower进行了状态同步后，他就可以开始广播消息了，即进入广播状态。
这时候当一个Server加入ZooKeeper服务中，它会在恢复模式下启动，发现Leader，并和Leader进行状态同步。待到同步结束，它也参与消息广播。
ZooKeeper服务一直维持在广播状态，直到Leader崩溃了或者Leader失去了大部分的Followers支持。
广播模式极其类似于分布式事务中的2pc（two-phrase commit 两阶段提交）：即Leader提起一个决议，由Followers进行投票，Leader对投票结果进行计算决定是否通过该决议，如果通过执行该决议（事务），否则什么也不做。

广播协议在所有的通讯过程中使用TCP的FIFO信道，通过使用该信道，使保持有序性变得非常的容易。通过FIFO信道，消息被有序的deliver。只要收到的消息一被处理，其顺序就会被保存下来。
Leader会广播已经被deliver的Proposal消息。在发出一个Proposal消息前，Leader会分配给Proposal一个单调递增的唯一id，称之为zxid。
广播是把Proposal封装到消息当中，并添加到指向Follower的输出队列中，通过FIFO信道发送到Follower。
当Follower收到一个Proposal时，会将其写入到磁盘，可以的话进行批量写入。一旦被写入到磁盘媒介当中，Follower就会发送一个ACK给Leader。
当Leader收到了指定数量的ACK时，Leader将广播commit消息并在本地递交该消息。当收到Leader发来commit消息时，Follower也会递交该消息。

 
ZAB协议简化了2PC事务提交：
1、去除中断逻辑移除，follower要么ack，要么抛弃Leader；
2、leader不需要所有的Follower都响应成功，只要一个多数派ACK即可。
 
丢弃的事务proposal处理过程：

ZAB协议中使用ZXID作为事务编号，ZXID为64位数字，低32位为一个递增的计数器，每一个客户端的一个事务请求时Leader产生新的事务后该计数器都会加1，
高32位为Leader周期epoch编号，当新选举出一个Leader节点时Leader会取出本地日志中最大事务Proposal的ZXID解析出对应的epoch把该值加1作为新的epoch，将低32位从0开始生成新的ZXID；
ZAB使用epoch来区分不同的Leader周期，能有效避免了不同的leader服务器错误的使用相同的ZXID编号提出不同的事务proposal的异常情况，大大简化了提升了数据恢复流程；
所以这个崩溃的机器启动时，也无法成为新一轮的Leader，因为当前集群中的机器一定包含了更高的epoch的事务proposal。

#注册中心篇（七）：Zookeeper 简介和使用入门 https://xueyuanjun.com/post/21225

#Zookeeper原理架构  https://www.cnblogs.com/ChrisMurphy/p/6683397.html
Zookeeper能干嘛？！
1. 配置管理
这个好理解。分布式系统都有好多机器，比如我在搭建hadoop的HDFS的时候，需要在一个主机器上（Master节点）配置好HDFS需要的各种配置文件，然后通过scp命令把这些配置文件拷贝到其他节点上，这样各个机器拿到的配置信息是一致的，才能成功运行起来HDFS服务。Zookeeper提供了这样的一种服务：一种集中管理配置的方法，我们在这个集中的地方修改了配置，所有对这个配置感兴趣的都可以获得变更。这样就省去手动拷贝配置了，还保证了可靠和一致性。 

2. 名字服务
这个可以简单理解为一个电话薄，电话号码不好记，但是人名好记，要打谁的电话，直接查人名就好了。 
分布式环境下，经常需要对应用/服务进行统一命名，便于识别不同服务； 
类似于域名与ip之间对应关系，域名容易记住； 
通过名称来获取资源或服务的地址，提供者等信息
3. 分布式锁
碰到分布二字貌似就难理解了，其实很简单。单机程序的各个进程需要对互斥资源进行访问时需要加锁，那分布式程序分布在各个主机上的进程对互斥资源进行访问时也需要加锁。很多分布式系统有多个可服务的窗口，但是在某个时刻只让一个服务去干活，当这台服务出问题的时候锁释放，立即fail over到另外的服务。这在很多分布式系统中都是这么做，这种设计有一个更好听的名字叫Leader Election(leader选举)。举个通俗点的例子，比如银行取钱，有多个窗口，但是呢对你来说，只能有一个窗口对你服务，如果正在对你服务的窗口的柜员突然有急事走了，那咋办？找大堂经理（zookeeper）!大堂经理指定另外的一个窗口继续为你服务！
4. 集群管理
在分布式的集群中，经常会由于各种原因，比如硬件故障，软件故障，网络问题，有些节点会进进出出。有新的节点加入进来，也有老的节点退出集群。这个时候，集群中有些机器（比如Master节点）需要感知到这种变化，然后根据这种变化做出对应的决策。我已经知道HDFS中namenode是通过datanode的心跳机制来实现上述感知的，那么我们可以先假设Zookeeper其实也是实现了类似心跳机制的功能吧！
Zookeeper的特点
1 最终一致性：为客户端展示同一视图，这是zookeeper最重要的功能。 
2 可靠性：如果消息被到一台服务器接受，那么它将被所有的服务器接受。 
3 实时性：Zookeeper不能保证两个客户端能同时得到刚更新的数据，如果需要最新数据，应该在读数据之前调用sync()接口。 
4 等待无关（wait-free）：慢的或者失效的client不干预快速的client的请求。 
5 原子性：更新只能成功或者失败，没有中间状态。 
6 顺序性：所有Server，同一消息发布顺序一致。
用到Zookeeper的系统
HDFS中的HA方案 
YARN的HA方案 
HBase：必须依赖Zookeeper，保存了Regionserver的心跳信息，和其他的一些关键信息。 
Flume：负载均衡，单点故障

#关于zookeeper第三方客户端zkclient的使用说明 https://blog.csdn.net/sun_wangdong/article/details/77461108 
ZkClient
       在使用ZooKeeper的Java客户端时，经常需要处理几个问题：重复注册watcher、session失效重连、异常处理。
       要解决上述的几个问题，可以自己解决，也可以采用第三方的java客户端来完成。这里就介绍一种常用的客户端zkclient，目前已经运用到了很多项目中，知名的有Dubbo、Kafka、Helix。
 
ZKClient的设计
 
ZkClient的组件说明
从上述结构上看，IZKConnection是一个ZkClient与ZooKeeper之间的一个适配器。在代码里直接使用的是ZKClient，其实质还是委托了zookeeper来处理了。
       前面有一篇文章中，已经说了，使用ZooKeeper客户端来注册watcher有几种方法：1、创建ZooKeeper对象时指定默认的Watcher，2、getData()，3、exists()，4、getchildren。其中getdata,exists注册的是某个节点的事件处理器（watcher），getchildren注册的是子节点的事件处理器（watcher）。而在ZKClient中，根据事件类型，分为了节点事件（数据事件）、子节点事件。对应的事件处理器则是IZKDataListener和IZKChildListener。另外加入了Session相关的事件和事件处理器。

ZkEventThread是专门用来处理事件的线程。
重要处理流程说明
 
启动ZKClient
在创建ZKClient对象时，就完成了到ZooKeeper服务器连接的建立。具体过程是这样的：
      
 
1、  启动时，指定好connection string，连接超时时间，序列化工具等。
2、  创建并启动eventThread，用于接收事件，并调度事件监听器Listener的执行。
3、  连接到zookeeper服务器，同时将ZKClient自身作为默认的Watcher。
 
为节点注册Watcher
       ZooKeeper的三个方法：getData、getChildren、exists，ZKClient都提供了相应的代理方法。就拿exists来看：
 
可以看到，是否注册watcher，由hasListeners(path)来决定的。
 
hasListeners就是看有没有与该数据节点绑定的listener。
 
所以呢，默认情况下，都会自动的为指定的path注册watcher，并且是默认的watcher（ZKClient）。怎么才能让hasListeners判定值为true呢，也就是怎么才能为path绑定Listener呢？
ZKClient提供了订阅功能：
 
 
一个新建的会话，只需要在取得响应的数据节点后，调用subscribteXxx就可以订阅上相应的事件了。
 
 
ZooKeeper的变更操作
Zookeeper中提供的变更操作有：节点的创建、删除，节点数据的修改。
 
创建操作，数据节点分为四种，ZKClient分别为他们提供了相应的代理：

 
删除节点的操作：
 
修改节点数据的操作：
 
writeDataReturnStat（）：写数据并返回数据的状态。
updateDataSerialized（）：修改已序列化的数据。执行过程是：先读取数据，然后使用DataUpdater对数据修改，最后调用writeData将修改后的数据发送给服务端。
 
客户端处理变更
       前面已经知道，ZKClient是默认的Watcher，并且在为各个数据节点注册的Watcher都是这个默认的Watcher。那么该是如何将各种事件通知给相应的Listener呢？
 
处理过程大致可以概括为下面的步骤：
1、判断变更类型：变更类型分为State变更、ChildNode变更（创建子节点、删除子节点、修改子节点数据）、NodeData变更（创建指定node，删除节点，节点数据变更）。
 
2、取出与path关联的Listeners，并为每一个Listener创建一个ZKEvent，将ZkEvent交给ZkEventThread处理。
3、ZkEventThread线程，拿到ZkEvent后，只需要调用ZkEvent的run方法进行处理。
 
从这里也可以知道，具体的怎么如何调用Listener，还要依赖于ZkEvent的run()实现了。
 
序列化处理
ZooKeeper中，会涉及到序列化、反序列化的操作有两种：getData、setData。在ZKClient中，分别用readData、writeData来替代了。
对于readData：先调用zookeeper的getData，然后进行使用ZKSerializer进行反序列化工作。
对于writeData：先使用ZKSerializer将对象序列化后，再调用zookeeper的setData。
 
ZkClient如何解决使用ZooKeeper客户端遇到的问题的呢？
 
Watcher自动重注册：这个要是依赖于hasListeners（）的判断，来决定是否再次注册。如果对此有不清晰的，可以看上面的流程处理的说明
       Session失效重连：如果发现会话过期，就先关闭已有连接，再重新建立连接。
       异常处理：对比ZooKeeper和ZKClient，就可以发现ZooKeeper的所有操作都是抛异常的，而ZKClient的所有操作，都不会抛异常的。在发生异常时，它或做日志，或返回空，或做相应的Listener调用。
 
 
相比于ZooKeeper官方客户端，使用ZKClient时，只需要关注实际的Listener实现即可。所以这个客户端，还是推荐大家使用的。
另外，是关于zkclient的一些接口，我们可以通过这些接口直接调用，使其完成一些相应的任务。
