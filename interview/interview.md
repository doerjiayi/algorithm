#mysql
##mysql 常见面试题
https://www.cnblogs.com/williamjie/p/11081592.html

##MySQL数据库面试题（2020最新版）
https://blog.csdn.net/ThinkWon/article/details/104778621

##我以为我对Mysql事务很熟
https://blog.csdn.net/qq_43255017/article/details/106442887

##阿里的面试官教你Mysql索引！
https://blog.csdn.net/weixin_30355437/article/details/101074940

##写给所有开发人员的 MySQL 4 种事务隔离级别（图文并茂）
https://blog.csdn.net/huhigher/article/details/106347347

#c++
##C++面试常见题
https://www.cnblogs.com/inception6-lxc/p/8686156.html

##虚函数实现原理(转)
https://blog.csdn.net/wanghaobo920/article/details/7674631

##面试之C++11
https://blog.csdn.net/xp178171640/article/details/104366004

##C++面试常见题
https://www.cnblogs.com/inception6-lxc/p/8686156.html

##当面试官问我C++ 11新特性的时候，应该怎样回答？
https://www.zhihu.com/question/65209863/answer/230185590

##面试笔记——C++11新特性
https://www.cnblogs.com/SHOR/p/6641688.html

#zk
##zookeeper的原理和应用（非常详细透彻）
https://blog.csdn.net/csd850182221/article/details/100543025

##Zookeeper常见面试题总结
https://blog.csdn.net/Sunshine_2211468152/article/details/87938175

##ZooKeeper面试题（2020最新版）
https://blog.csdn.net/ThinkWon/article/details/104397719


#redis
##redis相关原理及面试官由浅到深必问的15大问题（高级）
https://www.jianshu.com/p/16b6db9ee410

##Redis面试题（2020最新版）
https://thinkwon.blog.csdn.net/article/details/103522351

缓存降级
当访问量剧增、服务出现问题（如响应时间慢或不响应）或非核心服务影响到核心流程的性能时，仍然需要保证服务还是可用的，即使是有损服务。系统可以根据一些关键数据进行自动降级，也可以配置开关实现人工降级。
缓存降级的最终目的是保证核心服务可用，即使是有损的。而且有些服务是无法降级的（如加入购物车、结算）。
在进行降级之前要对系统进行梳理，看看系统是不是可以丢卒保帅；从而梳理出哪些必须誓死保护，哪些可降级；比如可以参考日志级别设置预案：

一般：比如有些服务偶尔因为网络抖动或者服务正在上线而超时，可以自动降级；
警告：有些服务在一段时间内成功率有波动（如在95~100%之间），可以自动降级或人工降级，并发送告警；
错误：比如可用率低于90%，或者数据库连接池被打爆了，或者访问量突然猛增到系统能承受的最大阀值，此时可以根据情况自动降级或者人工降级；
严重错误：比如因为特殊原因数据错误了，此时需要紧急人工降级。

服务降级的目的，是为了防止Redis服务故障，导致数据库跟着一起发生雪崩问题。因此，对于不重要的缓存数据，可以采取服务降级策略，例如一个比较常见的做法就是，Redis出现问题，不去数据库查询，而是直接返回默认值给用户。


热点数据和冷数据
热点数据，缓存才有价值
对于冷数据而言，大部分数据可能还没有再次访问到就已经被挤出内存，不仅占用内存，而且价值不大。频繁修改的数据，看情况考虑使用缓存
对于热点数据，比如我们的某IM产品，生日祝福模块，当天的寿星列表，缓存以后可能读取数十万次。再举个例子，某导航产品，我们将导航信息，缓存以后可能读取数百万次。
数据更新前至少读取两次，缓存才有意义。这个是最基本的策略，如果缓存还没有起作用就失效了，那就没有太大价值了。
那存不存在，修改频率很高，但是又不得不考虑缓存的场景呢？有！比如，这个读取接口对数据库的压力很大，但是又是热点数据，这个时候就需要考虑通过缓存手段，减少数据库的压力，比如我们的某助手产品的，点赞数，收藏数，分享数等是非常典型的热点数据，但是又不断变化，此时就需要将数据同步保存到Redis缓存，减少数据库压力。
缓存热点key
缓存中的一个Key(比如一个促销商品)，在某个时间点过期的时候，恰好在这个时间点对这个Key有大量的并发请求过来，这些请求发现缓存过期一般都会从后端DB加载数据并回设到缓存，这个时候大并发的请求可能会瞬间把后端DB压垮。
解决方案
对缓存查询加锁，如果KEY不存在，就加锁，然后查DB入缓存，然后解锁；其他进程如果发现有锁就等待，然后等解锁后返回数据或者进入DB查询


使用Redis做过异步队列吗，是如何实现的
使用list类型保存数据信息，rpush生产消息，lpop消费消息，当lpop没有消息时，可以sleep一段时间，然后再检查有没有信息，如果不想sleep的话，可以使用blpop, 在没有信息的时候，会一直阻塞，直到信息的到来。redis可以通过pub/sub主题订阅模式实现一个生产者，多个消费者，当然也存在一定的缺点，当消费者下线时，生产的消息会丢失。
Redis如何实现延时队列
使用sortedset，使用时间戳做score, 消息内容作为key,调用zadd来生产消息，消费者使用zrangbyscore获取n秒之前的数据做轮询处理。
Redis回收进程如何工作的？

一个客户端运行了新的命令，添加了新的数据。
Redis检查内存使用情况，如果大于maxmemory的限制， 则根据设定好的策略进行回收。
一个新的命令被执行，等等。
所以我们不断地穿越内存限制的边界，通过不断达到边界然后不断地回收回到边界以下。

如果一个命令的结果导致大量内存被使用（例如很大的集合的交集保存到一个新的键），不用多久内存限制就会被这个内存使用量超越。


##面试官：说说你对ZooKeeper集群与Leader选举的理解？
https://blog.csdn.net/g6U8W7p06dCO99fQ3/article/details/89166894



##真实面试经历：十面阿里，七面头条，六个Offer
https://www.pianshen.com/article/2178153039/

##面试鹅厂
https://zhuanlan.zhihu.com/p/83936791

##RAFT算法详解
https://www.cnblogs.com/charlieroro/articles/12655975.html

#tcp
##TCP连接拥塞控制四种方法总结（详细简单，稳的一批）
https://blog.csdn.net/qq_26896213/article/details/84594060

##图解 TCP 重传、滑动窗口、流量控制、拥塞控制
https://blog.csdn.net/qq_31442743/article/details/105613124

##今日头条面试----TCP拥塞控制和流量控制 
https://www.cnblogs.com/aademeng/articles/11083448.html

##TCP Win=0,Len=0
https://blog.csdn.net/farmwang/article/details/73518806

##TCP滑动窗口移动规则
https://blog.csdn.net/farmwang/article/details/73521663

##【面经笔记】TCP流量控制、阻塞控制
https://blog.csdn.net/xiaxzhou/article/details/76400640

##HTTP、TCP、IP协议常见面试题
https://blog.csdn.net/aigan8070/article/details/101573386

##TCP常见面试题
https://blog.csdn.net/mulinsen77/article/details/88925672

##网络协议面试题汇总
https://blog.csdn.net/tencupofkaiwater/article/details/88416256

##面试遇到TCP，读完这篇就够了！
https://zhuanlan.zhihu.com/p/158183161

##TCP 协议面试灵魂10问，建议收藏~
https://zhuanlan.zhihu.com/p/161970400

#进程间通信

##面试题：进程间通信的方式
https://blog.csdn.net/wm12345645/article/details/82381407

##操作系统面试题：关于共享内存相关API
https://blog.csdn.net/weixin_41019383/article/details/99186586

##共享内存实现进程间通信
https://blog.csdn.net/qq_17525769/article/details/80877032

##共享内存和信号
https://blog.csdn.net/farsight2009/article/details/5396777
