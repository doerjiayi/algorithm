
#Nginx源码分析(25篇)
https://blog.csdn.net/yangyin007/article/details/82777086

Nginx源码分析 - 初探Nginx的架构
https://blog.csdn.net/initphp/article/details/50582568

Nginx进程结构
Nginx是一款多进程的软件。Nginx启动后，会产生一个master进程和N个工作进程。其中nginx.conf中可以配置工作进程的个数：

worker_processes  1;
多进程模块有一个非常大的好处，就是不需要太多考虑并发锁的问题。

我们常见的软件Memcached就和Nginx相反，就是典型的多线程模型的c语言软件。

Nginx模块设计
高度模块化的设计是Nginx的架构基础。Nginx服务器被分解为多个模块，每个模块就是一个功能模块，只负责自身的功能，模块之间严格遵循“高内聚，低耦合”的原则。


Nginx源码分析 - 基础数据结构篇 - 内存池 ngx_palloc.c
Nginx源码分析 - 基础数据结构篇 - 数组结构 ngx_array.c
Nginx源码分析 - 基础数据结构篇 - 缓冲区结构 ngx_buf.c
Nginx源码分析 - 基础数据结构篇 - 双向链表结构 ngx_queue.c
Nginx源码分析 - 基础数据结构篇 - 单向链表结构 ngx_list.c
Nginx源码分析 - 基础数据结构篇 - hash表结构 ngx_hash.c
Nginx源码分析 - 基础数据结构篇 - 字符串结构 ngx_string.c
Nginx源码分析 - 主流程篇 - Nginx的启动流程

##Nginx源码分析 - 主流程篇 - 平滑重启和信号控制
https://blog.csdn.net/initphp/article/details/51841597


Nginx源码分析 - 主流程篇 - 全局变量cycle初始化
Nginx源码分析 - 主流程篇 - 模块的初始化
Nginx源码分析 - 主流程篇 - 解析配置文件
Nginx源码分析 - 主流程篇 - 多进程实现

##Nginx源码分析 - 主流程篇 - 多进程的惊群和进程负载均衡处理
https://blog.csdn.net/initphp/article/details/52266844
Linux2.6版本之前还存在对于socket的accept的惊群现象。之后的版本已经解决掉了这个问题。

惊群是指多个进程/线程在等待同一资源时，每当资源可用，所有的进程/线程都来竞争资源的现象。

Nginx采用的是多进程的模式。假设Linux系统是2.6版本以前，当有一个客户端要连到Nginx服务器上，Nginx的N个进程都会去监听socket的accept的，如果全部的N个进程都对这个客户端的socket连接进行了监听，就会造成资源的竞争甚至数据的错乱。我们要保证的是，一个链接在Nginx的一个进程上处理，包括accept和read/write事件。

 

Nginx解决惊群和进程负载均衡处理的要点
Nginx的N个进程会争抢文件锁，当只有拿到文件锁的进程，才能处理accept的事件。
没有拿到文件锁的进程，只能处理当前连接对象的read事件
当单个进程总的connection连接数达到总数的7/8的时候，就不会再接收新的accpet事件。
如果拿到锁的进程能很快处理完accpet，而没拿到锁的一直在等待（等待时延：ngx_accept_mutex_delay），容易造成进程忙的很忙，空的很空


Nginx源码分析 - Event事件篇 - Nginx的Event事件模块概览
Nginx源码分析 - Event事件篇 - Event模块和配置的初始化
Nginx源码分析 - Event事件篇 - Event模块的进程初始化ngx_event_process_init
Nginx源码分析 - Event事件篇 - epoll事件模块
Nginx源码分析 - HTTP模块篇 - ngx_http_block函数和HTTP模块的初始化
Nginx源码分析 - HTTP模块篇 - ngx_http_optimize_servers函数和TCP连接建立过程
Nginx源码分析 - HTTP模块篇 - ngx_http_wait_request_handler函数和HTTP Request解析过程
Nginx源码分析 - HTTP模块篇 - ngx_http_core_run_phases函数和HTTP模块的阶段处理PHASE handler
Nginx源码分析 - 实战篇 - 编写一个自定义的模块
Nginx源码分析 - 实战篇 - 编写一个挂载到阶段处理的模块