
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


