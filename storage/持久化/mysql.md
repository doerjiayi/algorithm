
##面试
###无语，我差点被面试官怼坏了，又给我问到MySQL索引
https://www.jianshu.com/p/c82148473235

###你说精通MySQL，竟然连bin log、redo log都不知道

###MySQL 万字精华总结 + 面试100 问，吊打面试官绰绰有余
https://www.jianshu.com/p/c189439fb32e

###去阿里面试被问：如果是MySQL引起的CPU消耗过大，你会如何优化？
https://www.jianshu.com/p/1990261fca3f

##同步
###MySQL 半同步复制模式说明及配置示例 - 运维小结 
https://www.cnblogs.com/kevingrace/p/10228694.html

##mvcc
###史上最详尽，一文讲透 MVCC 实现原理
https://blog.csdn.net/diligent203/article/details/100751755#comments

##MySQL 主从同步延迟的原因及解决办法
https://blog.csdn.net/hao_yunfeng/article/details/82392261
 

##隔离级别
事务的四种隔离级别
https://www.cnblogs.com/catmelo/p/8878961.html

在数据库操作中，为了有效保证并发读取数据的正确性，提出的事务隔离级别。我们的数据库锁，也是为了构建这些隔离级别存在的。
隔离级别
 
 
隔离级别 脏读（Dirty Read）不可重复读（NonRepeatable Read）幻读（Phantom Read）
未提交读（Read uncommitted）可能 可能 可能
已提交读（Read committed）不可能 可能 可能
可重复读（Repeatable read）不可能 不可能 可能
可串行化（Serializable ）不可能 不可能 不可能

未提交读(Read Uncommitted)：允许脏读，也就是可能读取到其他会话中未提交事务修改的数据
提交读(Read Committed)：只能读取到已经提交的数据。Oracle等多数数据库默认都是该级别 (不重复读)
可重复读(Repeated Read)：可重复读。在同一个事务内的查询都是事务开始时刻一致的，InnoDB默认级别。在SQL标准中，该隔离级别消除了不可重复读，但是还存在幻象读
串行读(Serializable)：完全串行化的读，每次读都需要获得表级共享锁，读写相互都会阻塞

可重复读和幻读
在可重复读中，该sql第一次读取到数据后，就将这些数据加锁（悲观锁），其它事务无法修改这些数据，就可以实现可重复读了。但这种方法却无法锁住insert的数据，所以当事务A先前读取了数据，或者修改了全部数据，事务B还是可以insert数据提交，这时事务A就会发现莫名其妙多了一条之前没有的数据，这就是幻读，不能通过行锁来避免。需要Serializable隔离级别 ，读用读锁，写用写锁，读锁和写锁互斥，这么做可以有效的避免幻读、不可重复读、脏读等问题，但会极大的降低数据库的并发能力。
但是MySQL、ORACLE、PostgreSQL等成熟的数据库，出于性能考虑，都是使用了以乐观锁为理论基础的MVCC（多版本并发控制）来实现。

##MySQL日志系统：redo log、binlog、undo log 区别与作用
https://blog.csdn.net/u010002184/article/details/88526708

一、重做日志（redo log）
作用：
确保事务的持久性。防止在发生故障的时间点，尚有脏页未写入磁盘，在重启mysql服务的时候，根据redo log进行重做，从而达到事务的持久性这一特性。

二、回滚日志（undo log）
作用：
保存了事务发生之前的数据的一个版本，可以用于回滚，同时可以提供多版本并发控制下的读（MVCC），也即非锁定读

三、二进制日志（binlog）：
作用：
用于复制，在主从复制中，从库利用主库上的binlog进行重播，实现主从同步。 
用于数据库的基于时间点的还原。

梳理下事务执行的各个阶段：
（1）写undo日志到log buffer；
（2）执行事务，并写redo日志到log buffer；
（3）如果innodb_flush_log_at_trx_commit=1，则将redo日志写到log file，并刷新落盘。
（4）提交事务。

可能有同学会问，为什么没有写data file，事务就提交了？
在数据库的世界里，数据从来都不重要，日志才是最重要的，有了日志就有了一切。

因为data buffer中的数据会在合适的时间 由存储引擎写入到data file，如果在写入之前，数据库宕机了，根据落盘的redo日志，完全可以将事务更改的数据恢复。好了，看出日志的重要性了吧。先持久化日志的策略叫做Write Ahead Log，即预写日志。

分析几种异常情况：
innodb_flush_log_at_trx_commit=2（innodb_flush_log_at_trx_commit和sync_binlog参数详解）时，将redo日志写入logfile后，为提升事务执行的性能，存储引擎并没有调用文件系统的sync操作，将日志落盘。如果此时宕机了，那么未落盘redo日志事务的数据是无法保证一致性的。

undo日志同样存在未落盘的情况，可能出现无法回滚的情况。

checkpoint：
checkpoint是为了定期将db buffer的内容刷新到data file。当遇到内存不足、db buffer已满等情况时，需要将db buffer中的内容/部分内容（特别是脏数据）转储到data file中。在转储时，会记录checkpoint发生的”时刻“。在故障回复时候，只需要redo/undo最近的一次checkpoint之后的操作。

##Mysql乐观锁实现
https://blog.csdn.net/lp2388163/article/details/80683383
Mysql锁机制--乐观锁 & 悲观锁
https://www.cnblogs.com/cyhbyw/p/8869855.html

##mysql索引
https://www.cnblogs.com/wlwl/p/9465583.html
https://www.cnblogs.com/boothsun/p/8970952.html#autoid-7-1-0
https://www.cnblogs.com/boothsun/p/8970952.html#autoid-6-1-0
https://blog.csdn.net/hixiaoxiaoniao/article/details/80983709

##高性能mysql
《高性能Mysql》重点总结（一）——数据库基础
https://blog.csdn.net/honhong1024/article/details/81268757
MySQL逻辑架构与相关知识 ———>《高性能MySQL》
https://blog.csdn.net/weixin_42123737/article/details/91547875


##Mysql事物与二阶段提交

### 1.事务的四种特性(ACID)
事务可以是一个非常简单的SQL构成，也可以是一组复杂的SQL语句构成。事务是访问并且更新数据库中数据的一个单元，在事务中的操作，要么都修改，要么都不做修改，这就是事务的目的，也是事务模型区别于其他模型的重要特征之一。
事务的原子性:原子是不可分割的,事务不可分割(没有commit数据不能被读到).

事务的持久性:在commit之后,不能丢数据.(就是在提交后,数据必须落盘redo落盘).

事务的隔离性:在数据库里面,各个事务之间不能互相影响.

事务的一致性:事务前后,不能违反mysql的约束.

#### 1.原子性(Atomicity)

原子性是指是事务是不可分割的一部分，一个事务内的任务要么全部执行成功，要么全部不执行，不存在执行一部分的情况。
可以将整个取款流程当做原子操作，要么取款成功，要么取款失败。

1. 我们可以使用取钱的例子，来讲解这一特性

2. 登录ATM机器

3. 从远程银行的数据库中，获取账户的信息

4. 用户在ATM上输入欲取出的金额

5. 从远程银行的数据库中，更新账户信息

6. ATM机器出款

7. 用户取钱

整个取钱操作，应该视为原子操作，要么都做，要么都不做，不能用户钱还没出来，但是银行卡的钱已经被扣除了。使用事务模型可以保证该操作的一致性

####  2.一致性(Consistency)
是指事务的完整性约束没有被破坏，如果遇到了违反约束的情况，数据库会被自动回滚。一致性是指数据库从一种状态转变为另一种状态。在开始事务和结束事务后，数据库的约束性没有被破坏。例如，数据库表中的用户ID列为唯一性约束，即在表中姓名不能重复.
 

####  3.隔离性(isolation)
事物的隔离性要求每个事务的操作，不会受到另一个事务的影响。隔离状态执行事务,使他们好像是系统在给定时间内执行的唯一操作.这种属性有时称为串行化.

####  4.持久性(Durability)
 事务一旦提交，那么结果就是持久性的,不会被回滚。即使发生宕机，数据库也可以做数据恢复。
说明：隔离性通过锁实现，原子性、一致性、持久性通过数据库的redo和undo来完成

支持事务的数据库：InnoDB、NDBCluster、TokuDB

不支持事务的数据库：MyISAM、MEMORY

###  5.事物的实现

重做日志（redo）用来实现事务的持久性，有两部分组成，一是内存中的重做日志，一个是硬盘上的重做日志文件。innodb是支持事务的存储引擎，通过日志先行WAL，来实现数据库的事务的特性，在一个事务提交的时候，并不是直接对内存的脏数据进行落盘，而是先把重做日志缓冲中的日志写入文件，然后再提交成功。这样保证了即使在断电的情况下，依然可以依靠redo log进行数据的恢复与重做。只要是提交的事务，在redo中就会有记录，数据库断电后重启，数据库会对提交的事务，但是没有写入硬盘的脏数据，利用redo来进行重做。
还要一个保证事务的是undo，undo有两个作用：
1.实现事务的回滚

2.实现mvcc的快照读取

redo是物理逻辑日志，计算页的物理修改操作.

undo是逻辑记录，记录了行的操作内容.

两阶段提交:先写redo -buffer再写binlog 并落盘最后落盘redo.


###   6.lsn号

 LSN（log sequence number）日志序列号
查看LSN信息:  show engine innodb status\G;

LSN实际上对应日志文件的偏移量，新的LSN＝旧的LSN + 写入的日志大小.

日志文件刷新后，LSN不会进行重置

Log sequence number:当前系统LSN最大值,新的事务日志LSN将在此基础上生成(LSN1+新日志的大小）

Log flushed up to:当前已经写入日志文件的LSN

Pages flushed up to:当前最旧的脏页数据对应的LSN,写Checkpoint的时候直接将此LSN写入到日志文件

Last checkpoint at:当前已经写入Checkpoint的LSN


###  7.redo作用总结
 redo作用
Redo介绍:

1. DML操作导致的页面变化，均需要记录Redo日志（物理日志）

2. 在页面修改完成之后，在脏页刷出磁盘之前，写入Redo日志;

3. 日志先行（WAL），日志一定比数据页先写回磁盘;

4. 聚簇索引/二级索引/Undo页面修改，均需要记录Redo日志;

为了管理脏页，在 Buffer Pool 的每个instance上都维持了一个flush list，flush list 上的 page 按照修改这些page 的LSN号进行排序。因此定期做redo checkpoint点时，选择的 LSN 总是所有 bp instance 的 flush list 上最老的那个page（拥有最小的LSN）。由于采用WAL的策略，每次事务提交时需要持久化 redo log 才能保证事务不丢。而延迟刷脏页则起到了合并多次修改的效果，避免频繁写数据文件造成的性能问题。

REDO的作用:提高性能和做crash recovery

提高性能:

1. 日志用来记录buffer pool中页的page修改的，每次数据提交只要写redo日志就可以，不需要每次都写脏页。

2. 通常一个数据页是16KB，如果不写日志，每次的写入还是16kb，即使修改很少数据，仍然要全部落盘，性能影响非常严重。

3. 如果没有日志，每次都会刷脏页，脏页的位置导致的IO是随机IO，而redo的数据页的大小是512字节，这样非常契合硬盘的块大小，可以进行顺序IO，这样可以保证顺序IO，同时可以大大提高IOPS

做crash recovery:

1. 数据库重启后，利用redo log进行数据库恢复工作，比对redolog LSN和数据页的LSN，如果数据页LSN低于REDO LOG LSN就会进行数据页实例恢复

 
### 8.undo作用
1. 用于回滚事务
2. 保证mvcc多版本高并发控制

innodb把undo分为两类

1. 新增undo

2. 修改undo

分类依据就是是否需要做purge操作.

insert在事务执行完成后，回滚记录就可以丢掉了。但是对于更新和删除操作而言，在完成事务后，还需要为MVCC提供服务，这些日志就被放到一个history list，用于MVCC以及等待purge。

undo日志的正确性是通过redo来保证的，所以在数据库恢复的时候，需要先恢复redo，在所有数据块都保证一致性的情况下，在进行undo的逻辑操作。

### 9.检查点（checkpoint）

检查点就是落脏页的点,本质是一个特殊的lsn.
检查点解决的问题:

1. 缩短数据库恢复时间

2. 缓冲池不够用的时候，刷新脏页到磁盘

3. 重做日志不够用的时候，刷新脏页

当数据库发生宕机的时候，数据库不需要恢复所有的页面，因为检查点之前的页面都已经刷新回磁盘了。故数据库只需要对检查点以后的日志进行恢复，这就大大减少了恢复时间。

检查点的类型:

检查点分为两种类型，一种是sharp检查点，一种是fuzzy检查点

sharp checkpoint落盘条件:

1. 关闭数据库的时候设置 innodb_fast_shutdown=1，在关闭数据库的时候，会刷新所有脏页到数据库内。

fuzzy checkpoint在数据库运行的时候，进行页面的落盘操作，不过这种模式下，不是全部落盘，而是落盘一部分数据。

Fuzzy落盘的条件:

1. master thread checkpoint： master每一秒或者十秒落盘
2. sync check point： redo 不可用的时候，这时候刷新到磁盘是从脏页链表中刷新的。
3. Flush_lru_list check point : 刷新flush list的时候

落盘的操作是异步的，因此不会阻塞其他事务执行。

检查点的作用:

缩短数据库的恢复时间
缓冲池不够用的时候，将脏页刷新到磁盘
重做日志不可用的时候，刷新脏页（循环使用redo文件，当旧的redo要被覆盖的时候，需要刷新脏页，造成检查点）

### 10.两阶段提交

MySQL二阶段提交流程：
事务的提交主要分三个主要步骤：

####1.Storage Engine（InnoDB） transaction prepare阶段：

存储引擎的准备阶段,写redo-buffer
此时SQL已经成功执行，并生成xid信息及redo和undo的内存日志。

####2.Binary log日志提交：

写binlog并落盘.
write()将binary log内存日志数据写入文件系统缓存。
fsync()将binary log文件系统缓存日志数据永久写入磁盘。

####3.Storage Engine（InnoDB）内部提交：落盘redo日志.

修改内存中事务对应的信息，并且将日志写入重做日志缓冲。

调用fsync将确保日志都从重做日志缓冲写入磁盘。

一旦步骤2中的操作完成，就确保了事务的提交，即使在执行步骤3时数据库发送了宕机。

即binlog落盘成功,就算redo未落盘成功,那么事务也算是提交成功了.

binlog落盘条件：参数sync_binlog： 0每秒落盘，1每次commit落盘  n 每n个事物落盘

此外需要注意的是，每个步骤都需要进行一次fsync操作才能保证上下两层数据的一致性。
步骤2的fsync参数由sync_binlog控制，步骤2的fsync由参数innodb_flush_log_at_trx_commit控制。(双1配置)

两阶段提交:先写redo -buffer 再写binlog 并落盘最后落盘redo-buffer.

最终:mysql在落盘日志的时候,先落盘binlog,再落盘redo.

 
###11.Rollback过程

当事务在binlog阶段crash，此时日志还没有成功写入到磁盘中，启动时会rollback此事务。
当事务在binlog日志已经fsync()到磁盘后crash，但是InnoDB没有来得及commit，此时MySQL数据库recovery的时候将会从二进制日志的Xid（MySQL数据库内部分布式事务XA）中获取提交的信息重新将该事务重做并commit使存储引擎和二进制日志始终保持一致。

总结起来说就是如果一个事物在prepare log阶段中落盘成功，并在MySQL Server层中的binlog也写入成功，那这个事务必定commit成功。

###12.事物隔离级别
  隔离级别                        事物                问题         缩写
READ UNCOMMITED        未提交读           脏读          RU
READ COMMITED              提交读             幻读           RC
REPEATEABLE READ          可重复读                            RR
SERIALIZABLE                    序列化
使用RR级别的话，可以保证数据库没有幻读，但是在该模式下，会增加锁竞争，造成数据库并发能力的下降。在RC模式下，没有next_lock的存在，即使在没有索引的情况下，也很难造成大规模的锁表而导致的死锁问题。

###13.自动提交参数设置

Mysql会自动提交 (mysql在mysql客户端是自动提交,但是java python里面不自动提交)
取消自动提交: set autocommit=off;   

###14.查看mysql隔离级别

  show variables like '%iso%';
 

###15.修改事物隔离级别

设置全局参数:
set global transaction isolation level read uncommitted;

Set global transaction isolation level read committed;

Set global transaction isolation level repeatable read;

设置会话级别参数:

set session transaction isolation level read uncommitted;

Set session transaction isolation level read committed;

Set session transaction isolation level repeatable read;

查看全局的MySQL GLOBAL的配置的隔离级别。

global参数设置后，需要新开启session才能生效.

### 4.事务隔离级别对应问题:(脏读,幻读,不可重复读)
RU                             ---------     产生脏读问题                
RC         ----------  解决脏读问题,但产生幻读和不可重复读问题.

RR           ---------  避免幻读和不可重复读问题.,待容易锁等待.

SERIALIZABLE     -----------序列化,串行读写.

在 REPEATEABLE READ下，其他事务对于数据的修改（update，delete）不会影响本事务对于数据的读取，会避免幻读的产生，幻读就是在一个事务内，读取到了不同的数据行数结果。

数据越安全，相对来说，数据库的并发能力越弱（并不代表总体性能越弱）。

脏读(Drity Read)：事务T1修改了一行数据，事务T2在事务T1提交之前读到了该行数据。

不可重复读(Non-repeatable read): 事务T1读取了一行数据。 事务T2接着修改或者删除了该行数据，当T1再次读取同一行数据的时候，读到的数据时修改之后的或者发现已经被删除。(针对update/delete操作).

在READ COMMITED下，未被提交的事务不会被读到，只有被提交的事务的数据，才会被读取到。（执行两次相同的SQL得到了不同的结果。),这就造成了幻读和不可重复读问题.

幻读(Phantom Read): 事务T1读取了满足某条件的一个数据集，事务T2插入了一行或者多行数据满足了T1的选择条件，导致事务T1再次使用同样的选择条件读取的时候，得到了比第一次读取更多的数据集。(针对insert操作).

幻读和可重复读的区别:

幻读更多的是针对于insert来说，即在一个事务之中，先后的两次select查询到了新的数据，新的数据来自于另一个事务的insert，一般称之为幻读，通过gap_lock(间隙锁)来防止产生幻读（虽然record可以避免数据行被修改，但是却无法阻止insert，gap_lock锁定索引间隙，防止了在事务查询的范围内的insert情况）

在ru隔离级别下造成脏读,在rc隔离级别下造成幻读和不可重复读.

可重复读:一般是针对于update和delete来说，可重复读采用了mvcc多版本控制来实现数据查询结果本身的不变。

###  5.事物隔离级别示例
#### 1.ru(READ-UNCOMMITED 未提交读)
#####Ru级别造成了脏读:  
session2可以读取到session1的没有提交的事务的数据(内存中没有提交的脏页).

脏读的发生至少在RU下，而目前几乎所有的数据库几乎都在RC级以上的隔离界别上。

脏读实例:

两个session修改隔离级别:  
set session transaction isolation level read uncommitted;

session1                         
session2

begin;                           
begin;

insert into t1 values(null,'testru')
select * from test1.t1;#查询到了testru数据,造成了脏读
commit;                                          
commit;


#### 2.rc(read committed 已提交读)
Rc级别解决了脏读.但会造成不可重复读和幻读.两者发生方式一样,但由于解决方法不同而区分.

解决了脏读实例:(针对select.)

修改隔离级别:   
Set session transaction isolation level read committed;

session1                                     
session2

begin;                                        
begin;
insert into test1.t1 values(null,'testrc');   
  select * from test1.t1;#没有查到
commit;
 select * from test1.t1; 查询到了
   commit;


##### 不可重复读:针对于update/delete来说.

##### 幻读:针对于insert来说的.

 幻读实例:  
 Set session transaction isolation level read committed;                              

session1                         
session2

begin;                           
begin;
select * from test1.t1;--7条
insert into t1 values()

commit;        
select * from test1.t1 --8
commit;


不可重复读实例:  
Set session transaction isolation level read committed;  

session1                         
session2

begin;                           
begin;
select * from test1.t1 where name=’aa’;--找到1条

Update t1 set name=’bb’ where name=’aa’;

commit;
select * from test1.t1 where name=’aa’ --没有找到.
commit;


加锁实例1:(加锁只针对指定的行,不影响update/insert操作)

Set session transaction isolation level read committed;

session1                               
session2

begin;                                 
begin;

update  test1.t1 set name='bbb' ;

insert into test1.t1 values(null,'test_rr') 加了锁,但插入成功.

commit;        
select * from test1.t1  1000w+1   
Commit;                                            


#### 3.rr(repeatable read 可重复读)
RR隔离级别:可重复读 .repeatable read

在RR模式下GAP_LOCK是默认开启的.

避免幻读实例:(insert)

set session transaction isolation level repeatable read;

session1                           
session2

begin;                             
begin;

select * from test1.t1;

insert into t1 values(null,'testrr');

commit;
select * from test1.t1;查询不到testrr
commit;
select * from test1.t1;查询到testrr


避免不可重复读实例:(update /delete)

set session transaction isolation level repeatable read;

session1                          
session2

begin;                             
begin;

select * from test1.t1;

update test1.t1 set name='testrr_upd' where name='testrr'

commit;
select * from test1.t1;查询不到testrr_upd

commit;
select * from test1.t1;查询到testrr_upd


锁等待实例:(一定会锁,update后,除了select,其他操作如insert/update都会被锁.)

set session transaction isolation level repeatable read;

session1                                   
session2

begin;                                     
begin;

update  test1.t1 set name='ccc';

insert into test1.t1 values(null,'test_rr')插入被阻塞,进入锁等待状态

commit;        

commit;  

select * from test1.t1  1000w+1                                          

 

###  5.MVCC的简单实现
MVCC在MySQL的InnoDB中的实现原理: 基于UNDO的多版本快照日志
MySQL InnoDB存储引擎，实现的是基于多版本的并发控制协议——MVCC (Multi-Version Concurrency Control)

(注：与MVCC相对的，是基于锁的并发控制，Lock-Based Concurrency Control)。

#### 1.MVCC的好处:读不加锁,读写不冲突.
读不加锁，读写不冲突。在读多写少的OLTP应用中，读写不冲突是非常重要的，极大的增加了系统的并发性能，这也是为什么现阶段，几乎所有的RDBMS，都支持了MVCC。

#### 2.读的类型:快照读和当前读.
在MVCC并发控制中，读操作可以分成两类：快照读 (snapshot read)与当前读 (current read)。

快照读: 读取的是记录的可见版本 (有可能是历史版本)，不用加锁。例如 select 就为快照读.

当前读:读取的是记录的最新版本，并且，当前读返回的记录，都会加上锁，保证其他事务不会再并发修改这条记录。例如删除/更新/删除操作。

过程: 在MVCC读取数据的过程中，会先对目前的事务和数据行的事务号进行对比，如果发现事务行的事务版本号，已经增长，则说明该行数据已经被其他事务修改，那么就需要根据undo，读取undo内的历史版本的相关逻辑操作信息，然后根据逻辑操作信息构建符合当前查询的数据版本，然后返回结果集（在RC和RR模式下，对于UNDO构建数据块的版本选择不同）

通过MVCC，虽然每行记录都需要额外的存储空间，更多的行检查工作以及一些额外的维护工作，但可以减少锁的使用，大多数读操作都不用加锁，读数据操作很简单，性能很好，并且也能保证只会读取到符合标准的行，也只锁住必要行。

#### 3.一致性非锁定读
一致性的非锁定读就是INNODB存储引擎通过行多版本控制的方式来读取当前执行时间数据库中的数据。如果读取的行正在执行DELETE或者UPDATE，这个时候读取操作不会等待所在行的锁的释放。INNODB这个时候会读取一个快照数据。（若读的数据加锁了，则读取其前一个版本的undo日志。）

一致性非锁定读取的原理是这样的：

Innodb通过隐藏的回滚指针保存前一个版本的undo日志，通过当前记录加上undo日志可以构造出记录的前一个版本，从而实现同一记录的多个版本。

快照数据就是当前数据行的历史版本，每个行记录可能有多个版本

在RC模式下，MVCC会一直读取最新的快照数据。

在RR模式下，MVCC会读取本事务开始时候的快照数据。

对于一致性非锁定读取，即使被读取的行已经SELECT ⋯ FOR UPDATE，也是可以进行读取的。

 

####  4.一致性锁定读取
有些用户需要采用锁定读取的方式来进行读取保证数据的一致性。

手动在查询中添加锁

查询中使用S锁:

select * from test1.t1 where id=1 lock in share mode;

查询中添加X锁:

select * from test1.t1 where id=1 for update;

关于锁等待的参数：参数支持范围为Session和Global,并且支持动态修改

事务等待获取资源等待的最长时间，超过这个时间还未分配到资源则会返回应用失败。

设置锁等待时间参数: innodb_lock_wait_timeout 30  --单位秒.

####  锁
Mysql锁加在索引上.
锁的作用:将并行的事务变成串行的.

从锁的颗粒度来说,锁分:表锁, 页锁,  行锁。

MySQL中锁的概念可以等同于：并发控制，序列化，隔离性.

这种用隔离性来描述锁，就是因为是事务ACID特性中的I，而锁就是用来实现事务一致性和隔离性的一种常用技术。

当数据库事务并发各自运行的时候，每个事务的运行不受到其他事务的影响。

简单的加锁技术就是对对象加上一个锁，若访问该事务的时候，发现已经有锁，则等待该事务锁的释放。

通过多粒度锁定，保证了数据库中事务的并发性。

#### 1.意向锁
意向锁:打算向这个表里的数据加锁,会提前在表级别加一个意向锁,加在聚簇索引的根节点.

1. 揭示下一层级请求的锁的类型

2. IS:事物想要获得一张表中某几行的共享锁

3. IX:事物想要获得一张表中某几行的排他锁

4. InnoDB存储引擎中意向锁都是表锁

假如此时有 事物tx1 需要在 记录A 上进行加 X锁 :

1. 在该记录所在的数据库上加一把意向锁IX 2. 在该记录所在的表上加一把意向锁IX

3. 在该记录所在的页上加一把意向锁IX

4. 最后在该记录A上加上一把X锁

假如此时有 事物tx2 需要对 记录B (假设和记录A在同一个页中)加 S锁 :

1. 在该记录所在的数据库上加一把意向锁IS 
2. 在该记录所在的表上加一把意向锁IS

3. 在该记录所在的页上加一把意向锁IS

4. 最后在该记录B上加上一把S锁

加锁是从上往下,一层一层 进行加的.

意向锁存在意义:

· 意向锁是为了实现多粒度的锁，表示在数据库中不但能实现行级别的锁，还可以实现页级别的锁，表级别的锁以及数据库级别的锁

· 如果没有意向锁，当你去锁一张表的时候，你就需要对表下的所有记录都进行加锁操作，且对其他事物刚刚插入的记录(游标已经扫过的范围)就没法在上面加锁了，此时就没有实现锁表的功能。

IX，IS是表级锁，不会和行级的X，S锁发生冲突。只会和表级的X，S发生冲突，行级别的X和S按照普通的共享、排他规则即可。所以只要写操作不是同一行，就不会发生冲突。

InnoDB 没有数据库级别的锁，也没有页级别的锁(InnoDB只能在表和记录上加锁)，所以InnoDB的意向锁只能加在表上，即InnoDB存储引擎中意向锁都是表锁.

#### 2.行锁(X和S锁)
对于行锁，根据其作用类型，可以分为两类：

共享锁(S lock) ,允许读取一个数据，同时允许其他事务对该事务进行更改。

排他锁(X lock),允许删除或者更新一条数据，同时不允许其他事务对该事务进行操作。

锁兼容:当一行获取S锁的时候，也可以获取另一个事务的S锁，这称之为锁兼容.

##mysql--主主复制
https://www.jianshu.com/p/469279c1ad39

##MySQL索引
https://www.jianshu.com/p/c82148473235
三、索引的分类
常见的索引类型有：主键索引、唯一索引、普通索引、全文索引、组合索引

1、主键索引：即主索引，根据主键pk_clolum（length）建立索引，不允许重复，不允许空值；

ALTER TABLE 'table_name' ADD PRIMARY KEY pk_index('col')；
2、唯一索引：用来建立索引的列的值必须是唯一的，允许空值

ALTER TABLE 'table_name' ADD UNIQUE index_name('col')；
3、普通索引：用表中的普通列构建的索引，没有任何限制

ALTER TABLE 'table_name' ADD INDEX index_name('col')；
4、全文索引：用大文本对象的列构建的索引（下一部分会讲解）

ALTER TABLE 'table_name' ADD FULLTEXT INDEX ft_index('col')；
5、组合索引：用多个列组合构建的索引，这多个列中的值不允许有空值

ALTER TABLE 'table_name' ADD INDEX index_name('col1','col2','col3')；
*遵循“最左前缀”原则，把最常用作为检索或排序的列放在最左，依次递减，组合索引相当于建立了col1,col1col2,col1col2col3三个索引，而col2或者col3是不能使用索引的。

*在使用组合索引的时候可能因为列名长度过长而导致索引的key太大，导致效率降低，在允许的情况下，可以只取col1和col2的前几个字符作为索引

ALTER TABLE 'table_name' ADD INDEX index_name(col1(4),col2（3))；

表示使用col1的前4个字符和col2的前3个字符作为索引

四、索引的实现原理
MySQL支持诸多存储引擎，而各种存储引擎对索引的支持也各不相同，因此MySQL数据库支持多种索引类型，如BTree索引，B+Tree索引，哈希索引，全文索引等等，

1、哈希索引：

只有memory（内存）存储引擎支持哈希索引，哈希索引用索引列的值计算该值的hashCode，然后在hashCode相应的位置存执该值所在行数据的物理位置，因为使用散列算法，因此访问速度非常快，但是一个值只能对应一个hashCode，而且是散列的分布方式，因此哈希索引不支持范围查找和排序的功能。

2、全文索引：

FULLTEXT（全文）索引，仅可用于MyISAM和InnoDB，针对较大的数据，生成全文索引非常的消耗时间和空间。对于文本的大对象，或者较大的CHAR类型的数据，如果使用普通索引，那么匹配文本前几个字符还是可行的，但是想要匹配文本中间的几个单词，那么就要使用LIKE %word%来匹配，这样需要很长的时间来处理，响应时间会大大增加，这种情况，就可使用时FULLTEXT索引了，在生成FULLTEXT索引时，会为文本生成一份单词的清单，在索引时及根据这个单词的清单来索引。FULLTEXT可以在创建表的时候创建，也可以在需要的时候用ALTER或者CREATE INDEX来添加：

//创建表的时候添加FULLTEXT索引
CTREATE TABLE my_table(
    id INT(10) PRIMARY KEY,
    name VARCHAR(10) NOT NULL,
    my_text TEXT,
    FULLTEXT(my_text)
)ENGINE=MyISAM DEFAULT CHARSET=utf8;
//创建表以后，在需要的时候添加FULLTEXT索引
ALTER TABLE my_table ADD FULLTEXT INDEX ft_index(column_name);
全文索引的查询也有自己特殊的语法，而不能使用LIKE %查询字符串%的模糊查询语法

SELECT * FROM table_name MATCH(ft_index) AGAINST('查询字符串');
注意：

*对于较大的数据集，把数据添加到一个没有FULLTEXT索引的表，然后添加FULLTEXT索引的速度比把数据添加到一个已经有FULLTEXT索引的表快。

*5.6版本前的MySQL自带的全文索引只能用于MyISAM存储引擎，如果是其它数据引擎，那么全文索引不会生效。5.6版本之后InnoDB存储引擎开始支持全文索引

*在MySQL中，全文索引支队英文有用，目前对中文还不支持。5.7版本之后通过使用ngram插件开始支持中文。

*在MySQL中，如果检索的字符串太短则无法检索得到预期的结果，检索的字符串长度至少为4字节，此外，如果检索的字符包括停止词，那么停止词会被忽略。

3、BTree索引和B+Tree索引

BTree索引
BTree是平衡搜索多叉树，设树的度为2d（d>1），高度为h，那么BTree要满足以一下条件：

每个叶子结点的高度一样，等于h；
每个非叶子结点由n-1个key和n个指针point组成，其中d<=n<=2d,key和point相互间隔，结点两端一定是key；
叶子结点指针都为null；
非叶子结点的key都是[key,data]二元组，其中key表示作为索引的键，data为键值所在行的数据；


B+Tree索引
B+Tree是BTree的一个变种，设d为树的度数，h为树的高度，B+Tree和BTree的不同主要在于：

B+Tree中的非叶子结点不存储数据，只存储键值；
B+Tree的叶子结点没有指针，所有键值都会出现在叶子结点上，且key存储的键值对应data数据的物理地址；
B+Tree的每个非叶子节点由n个键值key和n个指针point组成；

B+Tree对比BTree的优点：

1、磁盘读写代价更低

一般来说B+Tree比BTree更适合实现外存的索引结构，因为存储引擎的设计专家巧妙的利用了外存（磁盘）的存储结构，即磁盘的最小存储单位是扇区（sector），而操作系统的块（block）通常是整数倍的sector，操作系统以页（page）为单位管理内存，一页（page）通常默认为4K，数据库的页通常设置为操作系统页的整数倍，因此索引结构的节点被设计为一个页的大小，然后利用外存的“预读取”原则，每次读取的时候，把整个节点的数据读取到内存中，然后在内存中查找，已知内存的读取速度是外存读取I/O速度的几百倍，那么提升查找速度的关键就在于尽可能少的磁盘I/O，那么可以知道，每个节点中的key个数越多，那么树的高度越小，需要I/O的次数越少，因此一般来说B+Tree比BTree更快，因为B+Tree的非叶节点中不存储data，就可以存储更多的key。

2、查询速度更稳定

由于B+Tree非叶子节点不存储数据（data），因此所有的数据都要查询至叶子节点，而叶子节点的高度都是相同的，因此所有数据的查询速度都是一样的。

更多操作系统内容参考：

硬盘结构

扇区、块、簇、页的区别

操作系统层优化（进阶，初学不用看）

带顺序索引的B+TREE
很多存储引擎在B+Tree的基础上进行了优化，添加了指向相邻叶节点的指针，形成了带有顺序访问指针的B+Tree，这样做是为了提高区间查找的效率，只要找到第一个值那么就可以顺序的查找后面的值。

总结如下：

InnoDB 支持事务，支持行级别锁定，支持 B-tree、Full-text 等索引，不支持 Hash 索引；
MyISAM 不支持事务，支持表级别锁定，支持 B-tree、Full-text 等索引，不支持 Hash 索引；
此外，Memory 不支持事务，支持表级别锁定，支持 B-tree、Hash 等索引，不支持 Full-text 索引；



##原理剖析：InnoDB与MyISAM 聚集索引与非聚集索引
https://blog.csdn.net/qq_28584889/article/details/88778741

 InnoDB使用的是聚簇索引，将主键组织到一棵B+树中，而行数据就储存在叶子节点上，若使用"where id = 14"这样的条件查找主键，则按照B+树的检索算法即可查找到对应的叶节点，之后获得行数据。
 
 若对Name列进行条件搜索，则需要两个步骤：
 
 第一步在辅助索引B+树中检索Name，到达其叶子节点获取对应的主键。
 
 第二步使用主键在主索引B+树种再执行一次B+树检索操作，最终到达叶子节点即可获取整行数据。



MyISM使用的是非聚簇索引，非聚簇索引的两棵B+树看上去没什么不同，节点的结构完全一致只是存储的内容不同而已，主键索引B+树的节点存储了主键，辅助键索引B+树存储了辅助键。

表数据存储在独立的地方，这两颗B+树的叶子节点都使用一个地址指向真正的表数据，对于表数据来说，这两个键没有任何差别。

由于索引树是独立的，通过辅助键检索无需访问主键的索引树。

因为InnoDB的数据文件本身要按主键聚集，所以InnoDB要求表必须有主键（MyISAM可以没有），如果没有显式指定，则MySQL系统会自动选择一个可以唯一标识数据记录的列作为主键，如果不存在这种列，则MySQL自动为InnoDB表生成一个隐含字段作为主键，这个字段长度为6个字节，类型为长整形。

并且和MyISAM不同，InnoDB的辅助索引数据域存储的也是相应记录主键的值而不是地址，所以当以辅助索引查找时，会先根据辅助索引找到主键，再根据主键索引找到实际的数据。

所以Innodb不建议使用过长的主键，否则会使辅助索引变得过大。

建议使用自增的字段作为主键，这样B+Tree的每一个结点都会被顺序的填满，而不会频繁的分裂调整，会有效的提升插入数据的效率。

##MySQL 8.0 InnoDB全文索引可用于生产环境吗（续）
https://blog.csdn.net/n88Lpo/article/details/106045113

##MySQL每秒57万的写入，带你飞~
https://blog.csdn.net/n88Lpo/article/details/78718321


