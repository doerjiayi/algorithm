
##ZAB协议
https://cloud.tencent.com/developer/article/1469528
ZAB协议是为分布式协调服务Zookeeper专门设计的一种支持崩溃恢复的原子广播协议。在Zookeeper中，主要依赖ZAB协议来实现分布式数据一致性。

ZAB协议内容
ZAB协议主要有两种模式：崩溃恢复和消息广播。

当服务器启动、或者Leader宕机、或者Leader与绝大多数Follower无法正常通信时，ZAB协议就会进入崩溃恢复模式用来产生新的Leader。

当选举产生新的Leader并且完成数据同步以后，ZAB协议将由崩溃恢复模式转变为消息广播模式。

在ZAB协议的设计中，每一个进程都可能处于下面三种状态中的一种：

LOOKING：Leader选举阶段
FOLLOWING：Follower服务器和Leader保持同步状态
LEADING：Leader服务器作为主进程领导状态
ZAB协议需要保证以下两条原则：

确保那些已经在Leader服务器上提交的事务最终被所有的服务器进行提交(在Leader上提交，说明绝大多数Follower已经接收到了事务的Proposal请求并返回了ACK响应，只是还未收到commit请求)
确保丢弃那些只在Leader服务器上被提出的事务(只在Leader上被提出说明没有其他Follower并没有收到Proposal请求)
消息广播
消息广播模式是在整个集群稳定运行时的模式。它的操作类似于两阶段提交。

在消息的广播过程中，Leader会为每一个Follower准备一个事务队列，该队列符合FIFO原则。假设Follower接收到了客户端的写请求，写请求会被转发至Leader处理，当Leader收到客户端的写请求时，主要有以下步骤：

为写请求的事务Proposal分配一个全局唯一递增的ID(ZXID)
将这个事务放入每个Follower对应的事务队列，并按照FIFO顺序进行广播
Follower接收到事务请求后，将该事务请求写入本地事务日志文件，并在写成功后给Leader返回ACK响应
当Leader收到过半的Follower返回的ACK，便向所有的Follower发送commit请求通知Follower执行事务提交，同时Leader自身也完成事务提交
Follower收到commit请求后，完成事务提交
Leader为每个Follower使用队列做了异步解耦，大大降低同步阻塞，提高了系统的吞吐量。

崩溃恢复
崩溃恢复主要用来当Leader宕机或者Leader与大多数Follower因为网络原因无法通信时进行新Leader的选举或者集群启动时进行Leader的选举。崩溃恢复主要由两个过程组成：Leader选举、数据同步。

Leader选举
保证上述原则实现Leader很简单，只要保证新选出来的Leader服务器拥有最大的ZXID就可以，那么这个新Leader一定具有所有已提交的事务，还可以省去检查Proposal的提交和丢弃工作。

首先确认一个点，每个Zookeeper节点进入LOOKING状态时，都会发起选举流程，其他的Zookeeper机器收到该请求时，只有两种响应：

接收选票提议，同意Zookeeper节点成为Leader候选人
否决选票提议，并推荐自己上一次推荐的服务器作为Leader候选人
其次我们来讲述一下ZXID这个概念，ZXID其实就是一个全局单调递增的唯一的事务ID，由高32位的epoch表示选举周期和低32位的自增事务ID组成，每经历一次选举产生新的Leader，epoch的值将加1，而且低32位的的ID将被置为0，重新从0开始自增。

下面简单描述一下准Leader选举的过程，后面会出一篇源码分析来详解Leader的选举：

首先参与Leader选举的服务器必须是状态位LOOKING状态的节点
Zookeeper节点向其他的服务器节点发送自己要成为Leader候选人的请求(请求包含ZXID)
其他节点收到请求后，将本地事务日志的ZXID与请求中ZXID进行比较，如果发现比自己的大(如果ZXID一样大就比较myid(这个后面讲))，就同意该节点成为候选人并更新 该节点为推荐候选人而不是自己然后通知其他的节点，否则还是将自己作为候选人推荐
每次投票都会进行统计，判断是否有过半的服务器收到的推荐候选人是一致的，如果过半就认为已经选出了准Leader
一旦选举完成，就需要改变服务状态，新的Leader置为LEADING状态，其他机器转变为FOLLOWING状态
数据同步
只有当集群中的过半及其完成了数据同步，准Leader就可以真正的成为Leader。Follower只会接收ZXID比自己的最后一次事务的ZXID大的提议。

所有的Follower向准Leader发送自己的最后接收事务的epoch
准Leader选出最大的epoch，并在此基础上进行加1，然后将新的epoch发送给所有的Follower
Follower收到新的epoch之后，与自己的进行比较，小于就将自己的epoch更新成新的epoch，并向准Leader反馈ACK信息(epoch、历史事务集合)
准Leader收到ACK消息后，会在所有历史事务集合中选出其中一个历史事务集合作为初始化历史事务集合，该事务集合必须满足最大ZXID
准Leader将epoch和初始化历史事务集合发送给过半的Follower，每个Follower分配一个事务队列然后逐条将事务发送给Follower
Follower接收到事务请求后，如果已执行过则跳过，未执行则执行事务并反馈响应给准Leader
准Leader收到响应后则发起事务commit请求，提交事务
数据完成同步后，准Leader就是Leader，ZAB协议由崩溃恢复模式进入消息广播模式
ZAB和Paxos区别
本质区别在于设计的目的不一样，ZAB协议主要使用来构建一个高可用的分布式数据主备系统，Paxos算法主要是用来解决数据一致性。


##ZAB协议
　　介绍
　　1、zab协议是为分布式协调服务zookpeer专门设计的一种支持崩溃恢复的原子广播协议

　　2、在zookeeper中主要依赖ZAB协议来实现数据一致性，基于该协议zk实现了一种主备模式的系统架构来保证集群中各个副本之间数据的一致性。具体就是zk使用一个单一的主进程来接收并处理客户端的事务请求（就是写请求），并采用ZAB的原子广播协议，将服务器数据的状态变更以事务proposal的形式广播到所有的副本进程上去。

　　事务请求的处理方式
　　所有的事务请求必须由一个全局唯一的服务器来协调处理，这样的服务器被称为Leader服务器，而余下的其他服务器则是Follow服务器。Leader负责将一个事务请求转换成一个事务proposal，并将该proposal分发给集群中所有的Follow，之后Leader需要等待所有的Follow的反馈，一单超过半数Follow进行了正确的反馈后，那么Leader就会再次向所有的Follow发送commit消息，要求其将前一个proposal进行提交

　　注意：如果集群中非Leader服务器接收到事务请求，这些非Leader服务器会将事务请求转发给Leader服务器，只有Leader才能处理事务请求。

　　协议具体内容
　　ZAB协议包括两种基本模式：分别是崩溃恢复和消息广播

　　当整个集群启动过程中或者当Leader服务器出现网络中断，崩溃退出或重启等异常时，ZAB协议j就会进入恢复模式并选举产生新的Leader，当选举产生了新的Leader，同时集群中有过半的机器与该Leader服务器完成了状态同步（即数据同步）之后，ZAB协议就会退出崩溃恢复模式，进入消息广播模式。这时如果有一台遵守ZAB协议的服务器加入集群，因为此时集群中已经存在一个Leader服务器在广播消息，那么新加入的服务器自觉的进入恢复模式：找到Leader服务器并与之完成数据同步然后一起参与到消息广播流程中去。

　　当Leader出现崩溃退出或者机器重启，亦或是集群中已经不存在过半的服务器与Leader保持正常的通信，ZAB就会重新发一轮Leader选举并实现数据同步，最后又进入消息广播模式，接收事务请求

　　消息必须是有顺序的
　　在整个消息广播过程中，Leader会将每个事务请求转换成对应的proposal来进行广播，并且在广播事务proposal之前，Leader服务器会首先为这个事务proposal分配一个全局的单调递增的唯一ID，称之为事务ID（即ZXID），由于ZAB需要保证每一个消息严格的因果关系，因此必须将每一个proposal按照其ZXID的先后顺序来进行排序与处理。

　　消息广播
　　ZAB协议的消息广播使用的是一个原子广播协议，类似于二阶段提交，Leader接收事务请求，并转换成proposal广播给其他的Follow，然后过半的Follow ack消息，最后再给广播commit消息完成事务提交。

　　具体的，在消息广播过程中，Leader为每个Follow服务器分配一个单独的队列，然后将需要广播的proposal依次放到队列中去，并且根据FIFO策略进行消息发送。每一个Follow接收到proposal后，都会首先将其以事务日志的形式写入本地磁盘中，并且写入成功后反馈Leader一个ACK响应。当Leader接收到超过半数的ACK响应后，就会广播一个commit消息给Follow已通知他们完成事务提交，同时Leader自身也会完成事务的提交。

 

　　崩溃恢复
　　Leader挂了之后，ZAB协议就自动进入崩溃恢复模式，选举出新的Leader，并完成数据同步，然后退出崩溃恢复模式进入消息广播模式

 

　　ZAB协议如何保证数据一致性
　　异常情况：

　　1、假设一个事务在Leader上提交了，并且过半Follow都响应ACK了，但是Leader将commit消息发出后就挂了

　　2、假设一个事务在Leader提交了之后，Leader就挂掉了。

　　要保证如果发生上述2种情况，数据还能保持一致性，那么ZAB协议选举算法必须确保已经提交的proposal（发送过commit消息），在Follow上也必须完成提交；并且丢弃已经被跳过的事务proposal。

通过ZAB协议选举算法选举出来的Leader必须是拥有集群中最高编号（ZXID）proposal的机器，拥有最高编号说明新Leader一定具有所有已提交的提案。更为重要的是，如果让具有最高编号的事务proposal的机器成为Leader，就可以省去Leader服务器检查proposal的提交和丢弃工作的这一步操作了。

　　ZAB是如何数据同步？
　　完成Leader选举后（新Leader具有最高编号），在正式开始工作之前（接收事务请求，然后提出新的proposal）,Leader服务器会首先确认事务日志中的所有的Proposal是否都已经被集群中过半的提交了。

　　Leader服务器需要确保所有的Follow服务器能够接收到每一条事务proposal, 并且能将所有已经提交的事务proposal应用到内存数据库中去。等到Follow将所有尚未同步的事务proposal都从Leader服务器上同步过来并应用到内存数据库中去，Leader才会把该Follow加入到真正可用的Follow列表中。

　　ZAB是如何处理需要丢弃的Proposal？
　　在ZAB协议的事务编号ZXID设计中，ZXID是一个64位的数字，其中低32位可以看作成一个简单的单调递增计数器了，针对客户端每一个事务请求，Leader在产生新的事务Proposal时，都会对该计数器加1，而高32位则代表了Leader周期的epoch编号（可以理解为选举的届期），每当选举产生一个新的Leader，就会从这个Leader服务器上取出本地事务日志中最大的编号proposal的ZXID，并从ZXID中解析得到对应epoch编号，然后再对其进行加1，之后就以此编号作为新的epoch值，并将地32位置为0开始生成新的ZXID，ZAB协议通过epoch编号来区分Leader变化周期，能够有效的避免了不同的Leader错误的使用了相同的ZXID编号提出了不一样的proposal的异常情况。

　　基于这样的策略，当一个包含了上一个Leader周期中尚未提交过的事务proposal的服务器启动时，当这台机器加入集群中，以Follow角色连接上Leader服务器后，Leader服务器会根据自己服务器上最后提交的proposal来和Follow服务器的proposal进行比对，比对的结果肯定是Leader要求Follow进行一个回退操作，回退到一个确实已经被集群中过半机器提交的最新proposal。

　　ZAB协议特性：
　　1、ZAB协议需要确保那些已经在Leader服务器上提交（commit）的事务最终被所有的服务器提交

　　2、ZAB协议需要确保丢弃那些只在Leader上被提出的事务(只是被提出还没有被提交)

##阿里巴巴为什么不用 ZooKeeper 做服务发现？
https://www.jianshu.com/p/7fafac39bf84

有多少人想到过或者思考过一个问题：服务发现，ZooKeeper 真的是最佳选择么？而回望历史，我们也偶有迷思，在服务发现这个场景下，如果当年 ZooKeeper 的诞生之日比我们 HSF 的注册中心 ConfigServer 早一点会怎样？我们会不会走向先使用 ZooKeeper 然后疯狂改造与修补 ZooKeeper 以适应阿里巴巴的服务化场景与需求的弯路？但是，站在今天和前人的肩膀上，我们从未如今天这样坚定的认知到，在服务发现领域，ZooKeeper 根本就不能算是最佳的选择，一如这些年一直与我们同行的 Eureka 以及这篇文章 《Eureka! Why You Shouldn’t Use ZooKeeper for Service Discovery》那坚定的阐述一样，为什么你不应该用 ZooKeeper 做服务发现！吾道不孤矣。注册中心需求分析及关键设计考量接下来，让我们回归对服务发现的需求分析，结合阿里巴巴在关键场景上的实践，来一一分析，一起探讨为何说 ZooKeeper 并不是最合适的注册中心解决方案。注册中心是 CP 还是 AP 系统?CAP 和 BASE 理论相信读者都已经耳熟能详，其业已成了指导分布式系统及互联网应用构建的关键原则之一，在此不再赘述其理论，我们直接进入对注册中心的数据一致性和可用性需求的分析:

数据一致性需求分析
注册中心最本质的功能可以看成是一个 Query 函数 Si = F(service-name)，以 service-name 为查询参数，service-name 对应的服务的可用的 endpoints (ip:port)列表为返回值.

注: 后文将 service 简写为 svc。

先来看看关键数据 endpoints (ip:port) 不一致性带来的影响，即 CAP 中的 C 不满足带来的后果 :

如上图所示，如果一个 svcB 部署了 10 个节点 (副本 /Replica），如果对于同一个服务名 svcB, 调用者 svcA 的 2 个节点的 2 次查询返回了不一致的数据，例如: S1 = { ip1,ip2,ip3...,ip9 }, S2 = { ip2,ip3,....ip10 }, 那么这次不一致带来的影响是什么？相信你一定已经看出来了，svcB 的各个节点流量会有一点不均衡。ip1 和 ip10 相对其它 8 个节点{ip2...ip9}，请求流量小了一点，但很明显，在分布式系统中，即使是对等部署的服务，因为请求到达的时间，硬件的状态，操作系统的调度，虚拟机的 GC 等，任何一个时间点，这些对等部署的节点状态也不可能完全一致，而流量不一致的情况下，只要注册中心在 SLA 承诺的时间内（例如 1s 内）将数据收敛到一致状态（即满足最终一致），流量将很快趋于统计学意义上的一致，所以注册中心以最终一致的模型设计在生产实践中完全可以接受。

通过以上我们的阐述可以看到，在 CAP 的权衡中，注册中心的可用性比数据强一致性更宝贵，所以整体设计更应该偏向 AP，而非 CP，数据不一致在可接受范围，而 P 下舍弃 A 却完全违反了注册中心不能因为自身的任何原因破坏服务本身的可连通性的原则。

我们知道 ZooKeeper 的 ZAB 协议对每一个写请求，会在每个 ZooKeeper 节点上保持写一个事务日志，同时再加上定期的将内存数据镜像（Snapshot）到磁盘来保证数据的一致性和持久性，以及宕机之后的数据可恢复，这是非常好的特性，但是我们要问，在服务发现场景中，其最核心的数据 - 实时的健康的服务的地址列表真的需要数据持久化么？对于这份数据，答案是否定的。


通过事务日志，持久化连续记录这个变化过程其实意义不大，因为在服务发现中，服务调用发起方更关注的是其要调用的服务的实时的地址列表和实时健康状态，每次发起调用时，并不关心要调用的服务的历史服务地址列表、过去的健康状态。但是为什么又说需要呢，因为一个完整的生产可用的注册中心，除了服务的实时地址列表以及实时的健康状态之外，还会存储一些服务的元数据信息，例如服务的版本，分组，所在的数据中心，权重，鉴权策略信息，service label 等元信息，这些数据需要持久化存储，并且注册中心应该提供对这些元信息的检索的能力。Service Health Check使用 ZooKeeper 作为服务注册中心时，服务的健康检测常利用 ZooKeeper 的 Session 活性 Track 机制 以及结合 Ephemeral ZNode 的机制，简单而言，就是将服务的健康监测绑定在了 ZooKeeper 对于 Session 的健康监测上，或者说绑定在 TCP 长链接活性探测上了。这在很多时候也会造成致命的问题，ZK 与服务提供者机器之间的 TCP 长链接活性探测正常的时候，该服务就是健康的么？答案当然是否定的！注册中心应该提供更丰富的健康监测方案，服务的健康与否的逻辑应该开放给服务提供方自己定义，而不是一刀切搞成了 TCP 活性检测！健康检测的一大基本设计原则就是尽可能真实的反馈服务本身的真实健康状态，否则一个不敢被服务调用者相信的健康状态判定结果还不如没有健康检测。注册中心的容灾考虑前文提过，在实践中，注册中心不能因为自身的任何原因破坏服务之间本身的可连通性

在粗粒度分布式锁，分布式选主，主备高可用切换等不需要高 TPS 支持的场景下有不可替代的作用，而这些需求往往多集中在大数据、离线任务等相关的业务领域，因为大数据领域，讲究分割数据集，并且大部分时间分任务多进程 / 线程并行处理这些数据集，但是总是有一些点上需要将这些任务和进程统一协调，这时候就是 ZooKeeper 发挥巨大作用的用武之地。但是在交易场景交易链路上，在主业务数据存取，大规模服务发现、大规模健康监测等方面有天然的短板，应该竭力避免在这些场景下引入 ZooKeeper，在阿里巴巴的生产实践中，应用对 ZooKeeper 申请使用的时候要进行严格的场景、容量、SLA 需求的评估。所以可以使用 ZooKeeper，但是大数据请向左，而交易则向右，分布式协调向左，服务发现向右。

##Zookeeper ZAB协议中的zxid
zxid是事务编号，是64位的。可以把他拆分为两部分，分别都为32位。

低32位是事务id，是递增的。

高32位是leader周期epoch。 这里可能有点难理解，你可以把他理解为年代，每个leader都有一个年代。在leader所属的年代，该leader拥有最高权力。如果该leader挂了，那么就会换下一个leader，此时年代也应该+1.

会有这种情况，如果上一个年代的leader复活了，那就是说，之前的leader可能由于网络原因，现在好了。这可能会出现脑裂。（小声bb） 但是上一个leader所属的时代已经过去了，那他就会成为folower。

其实这样也是为了一致性，如果上一个leader还有权力，那么脑裂的时间会更长。还有，上一个leader，可能数据是不一致的，他会成为followe之后进行回退的操作，直到一个被集群中过半机器提交的最新事务。


致使ZooKeeper节点状态改变的每一个操作都将使节点接收到一个Zxid格式的时间戳，并且这个时间戳全局有序。也就是说，每个对节点的改变都将产生一个唯一的Zxid。如果Zxid1的值小于Zxid2的值，那么Zxid1所对应的事件发生在Zxid2所对应的事件之前。实际上，ZooKeeper的每个节点维护者两个Zxid值，为别为：cZxid、mZxid。

（1）cZxid： 是节点的创建时间所对应的Zxid格式时间戳。

（2）mZxid：是节点的修改时间所对应的Zxid格式时间戳。

实现中Zxid是一个64为的数字，它高32位是epoch用来标识Leader关系是否改变，每次一个Leader被选出来，它都会有一个新的epoch。低32位是个递增计数。

##Zookeeper：Zab协议、CAP定理详解
https://blog.csdn.net/weixin_41436619/article/details/83719270


##Zab算法详解 
https://www.cnblogs.com/aibabel/p/10973596.html

   Zookeeper使用了一种称为Zab（Zookeeper Atomic Broadcast）的协议作为其一致性复制的核心，据其作者说这是一种新发算法，其特点是充分考虑了Yahoo的具体情况：高吞吐量、低延迟、健壮、简单，但不过分要求其扩展性。下面将展示一些该协议的核心内容：
另，本文仅讨论Zookeeper使用的一致性协议而非讨论其源码实现
Zookeeper的实现是有Client、Server构成，Server端提供了一个一致性复制、存储服务，Client端会提供一些具体的语义，比如分布式锁、选举算法、分布式互斥等。从存储内容来说，Server端更多的是存储一些数据的状态，而非数据内容本身，因此Zookeeper可以作为一个小文件系统使用。数据状态的存储量相对不大，完全可以全部加载到内存中，从而极大地消除了通信延迟。
Server可以Crash后重启，考虑到容错性，Server必须“记住”之前的数据状态，因此数据需要持久化，但吞吐量很高时，磁盘的IO便成为系统瓶颈，其解决办法是使用缓存，把随机写变为连续写。
考虑到Zookeeper主要操作数据的状态，为了保证状态的一致性，Zookeeper提出了两个安全属性（Safety Property）
 
全序（Total order）：如果消息a在消息b之前发送，则所有Server应该看到相同的结果
因果顺序（Causal order）：如果消息a在消息b之前发生（a导致了b），并被一起发送，则a始终在b之前被执行。
为了保证上述两个安全属性，Zookeeper使用了TCP协议和Leader。通过使用TCP协议保证了消息的全序特性（先发先到），通过Leader解决了因果顺序问题：先到Leader的先执行。因为有了Leader，Zookeeper的架构就变为：Master-Slave模式，但在该模式中Master（Leader）会Crash，因此，Zookeeper引入了Leader选举算法，以保证系统的健壮性。归纳起来Zookeeper整个工作分两个阶段：


###Leader选举
####1. Atomic Broadcast
同一时刻存在一个Leader节点，其他节点称为“Follower”，如果是更新请求，如果客户端连接到Leader节点，则由Leader节点执行其请求；如果连接到Follower节点，则需转发请求到Leader节点执行。但对读请求，Client可以直接从Follower上读取数据，如果需要读到最新数据，则需要从Leader节点进行，Zookeeper设计的读写比例是2：1。
 
Leader通过一个简化版的二段提交模式向其他Follower发送请求，但与二段提交有两个明显的不同之处：
因为只有一个Leader，Leader提交到Follower的请求一定会被接受（没有其他Leader干扰）
不需要所有的Follower都响应成功，只要一个多数派即可
通俗地说，如果有2f+1个节点，允许f个节点失败。因为任何两个多数派必有一个交集，当Leader切换时，通过这些交集节点可以获得当前系统的最新状态。如果没有一个多数派存在（存活节点数小于f+1）则，算法过程结束。但有一个特例：
如果有A、B、C三个节点，A是Leader，如果B Crash，则A、C能正常工作，因为A是Leader，A、C还构成多数派；如果A Crash则无法继续工作，因为Leader选举的多数派无法构成。

####2. Leader Election
Leader选举主要是依赖Paxos算法，具体算法过程请参考其他博文，这里仅考虑Leader选举带来的一些问题。Leader选举遇到的最大问题是，”新老交互“的问题，新Leader是否要继续老Leader的状态。这里要按老Leader Crash的时机点分几种情况：

老Leader在COMMIT前Crash（已经提交到本地）

老Leader在COMMIT后Crash，但有部分Follower接收到了Commit请求

第一种情况，这些数据只有老Leader自己知道，当老Leader重启后，需要与新Leader同步并把这些数据从本地删除，以维持状态一致。

第二种情况，新Leader应该能通过一个多数派获得老Leader提交的最新数据

老Leader重启后，可能还会认为自己是Leader，可能会继续发送未完成的请求，从而因为两个Leader同时存在导致算法过程失败，解决办法是把Leader信息加入每条消息的id中，Zookeeper中称为zxid，zxid为一64位数字，高32位为leader信息又称为epoch，每次leader转换时递增；低32位为消息编号，Leader转换时应该从0重新开始编号。通过zxid，Follower能很容易发现请求是否来自老Leader，从而拒绝老Leader的请求。
 
因为在老Leader中存在着数据删除（情况1），因此Zookeeper的数据存储要支持补偿操作，这也就需要像数据库一样记录log。

####3. Zab与Paxos

Zab的作者认为Zab与paxos并不相同，只所以没有采用Paxos是因为Paxos保证不了全序顺序：

Because multiple leaders can propose a value for a given instance two problems arise.
First, proposals can conflict. Paxos uses ballots to detect and resolve conflicting proposals. 
Second, it is not enough to know that a given instance number has been committed, processes must also be able to fi
gure out which value has been committed.

Paxos算法的确是不关系请求之间的逻辑顺序，而只考虑数据之间的全序，但很少有人直接使用paxos算法，都会经过一定的简化、优化。
一般Paxos都会有几种简化形式，其中之一便是，在存在Leader的情况下，可以简化为1个阶段（Phase2）。仅有一个阶段的场景需要有一个健壮的Leader，因此工作重点就变为Leader选举，在考虑到Learner的过程，还需要一个”学习“的阶段，通过这种方式，Paxos可简化为两个阶段：
之前的Phase2

Learn
如果再考虑多数派要Learn成功，这其实就是Zab协议。Paxos算法着重是强调了选举过程的控制，对决议学习考虑的不多，Zab恰好对此进行了补充。
之前有人说，所有分布式算法都是Paxos的简化形式，虽然很绝对，但对很多情况的确如此，但不知Zab的作者是否认同这种说法？

####4.结束

本文只是想从协议、算法的角度分析Zookeeper，而非分析其源码实现，因为Zookeeper版本的变化，文中描述的场景或许已找不到对应的实现。另，本文还试图揭露一个事实：Zab就是Paxos的一种简化形式。

【参考资料】
A simple totally ordered broadcast protocol 
paxos
