
##Raft 为什么是更易理解的分布式一致性算法
https://www.cnblogs.com/mindwind/p/5231986.html

Leader 节点对一致性的影响
Raft 协议强依赖 Leader 节点的可用性来确保集群数据的一致性。数据的流向只能从 Leader 节点向 Follower 节点转移。当 Client 向集群 Leader 节点提交数据后，Leader 节点接收到的数据处于未提交状态（Uncommitted），接着 Leader 节点会并发向所有 Follower 节点复制数据并等待接收响应，确保至少集群中超过半数节点已接收到数据后再向 Client 确认数据已接收。一旦向 Client 发出数据接收 Ack 响应后，表明此时数据状态进入已提交（Committed），Leader 节点再向 Follower 节点发通知告知该数据状态已提交。



3. 数据到达 Leader 节点，成功复制到 Follower 所有节点，但还未向 Leader 响应接收
这个阶段 Leader 挂掉，虽然数据在 Follower 节点处于未提交状态（Uncommitted）但保持一致，重新选出 Leader 后可完成数据提交，此时 Client 由于不知到底提交成功没有，可重试提交。针对这种情况 Raft 要求 RPC 请求实现幂等性，也就是要实现内部去重机制。


6. 数据到达 Leader 节点，成功复制到 Follower 所有或多数节点，数据在所有节点都处于已提交状态，但还未响应 Client
这个阶段 Leader 挂掉，Cluster 内部数据其实已经是一致的，Client 重复重试基于幂等策略对一致性无影响。


7. 网络分区导致的脑裂情况，出现双 Leader
网络分区将原先的 Leader 节点和 Follower 节点分隔开，Follower 收不到 Leader 的心跳将发起选举产生新的 Leader。这时就产生了双 Leader，原先的 Leader 独自在一个区，向它提交数据不可能复制到多数节点所以永远提交不成功。向新的 Leader 提交数据可以提交成功，网络恢复后旧的 Leader 发现集群中有更新任期（Term）的新 Leader 则自动降级为 Follower 并从新 Leader 处同步数据达成集群数据一致。


综上穷举分析了最小集群（3 节点）面临的所有情况，可以看出 Raft 协议都能很好的应对一致性问题，并且很容易理解。



##Raft系列4 任期和选举（原创）
https://www.jianshu.com/p/aa137c94ac0b

在raft算法中，比较谁的数据最新有2个参考指标，任期和logIndex，任期大的节点，数据一定最新，任期一样的话，就要比较该任期内谁的MaxLogIndex最大了。引入任期的概念可以简化数据比较的精度。
任期的作用：

不同的服务器节点观察到的任期转换的次数可能不同，在某些情况下，一个服务器节点可能没有看到 leader 选举过程或者甚至整个任期全程。

任期在 Raft 算法中充当逻辑时钟的作用，这使得服务器节点可以发现一些过期的信息比如过时的 leader 。

每一个服务器节点存储一个当前任期号，该编号随着时间单调递增。

服务器之间通信的时候会交换当前任期号；

如果一个服务器的当前任期号比其他的小，该服务器会将自己的任期号更新为较大的那个值。

如果一个 candidate 或者 leader 发现自己的任期号过期了，它会立即回到 follower 状态。（所以说老leader如果发生了网络分区，后来接收到新leader的心跳的时候，比拼完任期之后，会自动变成follower。

如果一个节点接收到一个包含过期的任期号的请求，它会直接拒绝这个请求。

##Raft算法的理解
https://www.jianshu.com/p/b6c201586044

Term:
在Raft中使用了一个可以理解为任期的概念，用Term作为一个周期，每个Term都是一个连续递增的编号，每一轮选举都是一个Term周期，在一个Term中只能产生一个Leader；
其中Term的变化流程：
Raft开始时所有Follower的Term为1，其中一个Follower逻辑时钟到期后转换为Candidate，Term加1这是Term为2，然后开始选举，这时候有几种情况会使Term发生改变：
　　1：如果当前Term为2的任期内没有选举出Leader或出现异常，则Term递增，开始新一任期选举
　　2：当这轮Term为2的周期选举出Leader后，过后Leader宕掉了，然后其他Follower转为Candidate，Term递增，开始新一任期选举
　　3：当Leader或Candidate发现自己的Term比别的Follower小时Leader或Candidate将转为Follower，Term递增
　　4：当Follower的Term比别的Term小时Follower也将更新Term保持与其他Follower一致；


##Raft 实现日志复制同步
https://github.com/AhaMessageQueue/paxos_raft_protocol/blob/master/raft/raft.md


##Raft在网络分区时leader选举的一个疑问？
https://www.zhihu.com/question/302761390

##RAFT算法随想
https://blog.csdn.net/hfty290/article/details/75331948

一、确定性状态机
    在RAFT论文与Paxos论文中都有提及。如Paxos make simple中提到的：服务器可以看成是一个以某种顺序执行客户端命令的确定性状态机。
    为什么是确定性状态机呢？其实这和主备同步有关。
    确定性状态机的意思是：对于一个输入，只有一个固定的状态变迁。而非确定状态机则是有多种状态变迁，选择其中一种。在主备同步过程中，若同步的命令不满足确定性的要求那么主备就无法保持一致。例如：
要同步一个命令 x=rand()，在主上执行的结果很可能与备是不同的，因此此时在同步命令时就需要对该命令进行转换。如：在Master上执行的结果为
 x=100; 那么Master可以将x=100这条命令同步给Slave就能够实现状态一致。
也就是说，Master在执行命令时，若一条命令不满足确定性的条件，那么Master要根据执行结果将其转换成确定性的命令同步给Slave以保持主备一致。

二、丢弃term<current term的请求
    看论文的时候当请求报文的term参数小于current term时，只是笼统的说返回false或者拒绝。这点参考了etcd的源码，在etcd之中的处理是直接丢弃该请求报文。在etcd之中，对于每个请求报文会做两个检查，首先请求者ID必须是认可的集群成员，其次是term要大于等于current
 term，若不满足任何一个都是丢弃请求报文。丢弃应该是比返回false更好的方法，比如：一个已经被删除的成员向集群中发消息，此时通过检查请求者ID是否属于集群成员的方式就可以避免受到干扰。返回false就会把current
 term带上，对方受到该回复报文之后，由于自己的term较小就会立即转换成Follower，那么原先的集群马上就没有Leader。

三、Leader自动降级
    Leader根据心跳超时时间向外发送心跳包，以获取Follower的授权。假若在一个选举超时时间内没有收到多数Follower的授权回复，此时Leader可以采取降级措施，以避免存在两个Master的请情况。
    在ETCD之中，这种自动降级操作是可配置的，默认不会自动降级。若没有自动降级，在收到新Leader心跳包时，由于请求的term>current
 term，则老Leader自动变成Follower。
但是若该Leader处于一个网络分区之中，可能收不到新Leader的心跳包，因此配置成自动降级是比较合理的。

四、日志的顺序复制与preLogTerm、preLogIndex匹配规则
    RAFT算法中的日志必须是顺序复制的。就是说，假如有一条旧的日志还未复制给FollowerA，那么更新的日志就不能复制。因为，如果Follower接收了该日志，那么就会造成日志空洞。其次，根据复制日志时必须匹配preLogIndex、preLogTerm的要求，实际上也无法满足该条件，因此当有旧日志未复制，而直接复制新日志，Follower应该返回false。
    日志的顺序复制，很大程度上简化了Raft算法。比如查看各成员日志的新旧，只要比较最后一条日志即可。
    preLogTerm、preLogIndex的匹配规则是用于实现顺序复制的手段。有了这个规则，根据归纳假设就很容易得到所有的Follower日志最终都会与Leader完全一致。

四、新的Leader为何必须具有所有已提交日志
    根据Raft的日志复制规则，所有的Follower的日志最终会与Leader的日志完全一致。另外若一个日志被设置成已提交，那么必须假设该条日志被应用于状态机。
    因此若一条日志被一个Leader提交了，即使该Leader提交日志，应用到状态机之后，提交状态还未同步给Follower就宕机了，也要保证所有其他机器在将来将该日志应用到状态机。又因为Follower的日志提交与日志内容都是完全与Leader一致，那么就需要保证后续的Leader必须具有原Leader提交的日志，并且会在何时的时候提交，然后将提交状态同步给Follower，然后Follower提交并应用到状态机。

五、新的Leader如何保证具有前面Leader提交的日志
    这是通过Leader选举规则来保证的。前面说过，日志是顺序复制的，日志的新旧可以通过查看最后一条日志来判断。
    在Leader选举中，请求报文中有Candidate最后一条日志的term和index。收到投票请求的Follower会检查本地最后一条日志的term、index。只有Candidate的最后一条日志大于等于本地日志时才能投票。
    而提交日志的条件是日志复制给集群中的多数成员，Candidate选举为Leader的条件也是需要多数成员的投票。那么这两个成员之间必须有一个交叉，即有一个成员具有该日志，并且投票给了新Leader，也就意味着新Leader的日志至少不比该成员旧，那么新Leader也具有该日志。这样就得到证明了，后续的Leader一定具有前面Leader提交的日志。

六、为何一个Leader不能提交前面Term下的日志
    假设新的term=10，要提交term=8的日志。那么在term=9的时候有一个Leader，这个Leader是在提交term=8这条日志之前就选出来的。因此term=9的这个Leader不能保证会有term=8的这条日志。若term=10的这个Leader提交term=8的日志并应用到状态机之后，马上宕机。而term=9的这个Leader重新选举为term=11的Leader，那么term=8的这条日志很可能就被新Leader覆盖掉，而再不会被提交与应用。
    其实这里的关键就是：新的Leader会有前面Leader提交的日志，而旧的Leader则不能保证。
