#codis
##[codis]CodisLabs tutorial_zh
https://github.com/CodisLabs/codis/blob/release3.2/doc/tutorial_zh.md

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
