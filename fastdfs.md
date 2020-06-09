#fastdfs
##3分钟快速了解FastDFS
https://www.cnblogs.com/liusijun113/p/11061298.html

1、介绍
FastDFS是一个C语言写的阿里开源的分布式文件存储服务器
主要由两部分组成：
1、Tracker server ——————主要负责调度和追踪Storage状态（调度服务器），默认监听端口：22122
2、Storage server ——————文件存储服务器

客户端请求 Tracker server 进行文 件上传、下载，通过 Tracker server 调度最终由 Storage server 完成文件上传和下载。

2、文件上传流程
① 客户端发出请求上传文件，发送给Tracker server
② Tracker server 调度告诉客户端上传到哪个Storage
③ 客户端向指定的Storage请求存储
④ Storage存储后将加密成的文件id返回给客户端存到数据库

4、FastDFS的优势
解决了海量存储的问题
可同步方便扩展
同样内容的文件在FastDFS里只存放一个（a，b用户上传了内容相同的文件（不管文件名相不相同）最后只会存一个文件，而后一个人的上传速度几乎可以秒速上传，以为验证存在就直接指向。类比百度云上传）

5、拓展：如何加速静态文件的加载
　　通过nginx加速文件上传下载，本质还是通过nginx实现动静分离（借助fast_nginx_module_master.zip）# nginx 配置server {

 nginx 配置
server {
            listen       8888;
            server_name  localhost;     # 网站域名
            location ~/group[0-9]/ {    # 正则匹配静态文件路径
                ngx_fastdfs_module;
            }
            error_page   500 502 503 504  /50x.html;
            location = /50x.html {
            root   html;
            }
        }


https://www.cnblogs.com/huangxincheng/p/7928345.html
http://bbs.chinaunix.net/thread-4133506-1-1.html

https://blog.csdn.net/fan75258/article/details/75045392
https://www.cnblogs.com/chiangchou/p/fastdfs.html

##用FastDFS一步步搭建文件管理系统
https://www.cnblogs.com/chiangchou/p/fastdfs.html#_label1_5

##关于fastDFS断点续传的问题
https://blog.csdn.net/weixin_42554373/article/details/94453609

##FastDFS合并存储原理分析
https://blog.csdn.net/hfty290/article/details/42026215

##FastDFS之合并存储缺陷导致数据丢失或错误
https://blog.csdn.net/hfty290/article/details/42030481

##api
https://blog.csdn.net/u010841177/article/details/84600150

##集群
https://www.jianshu.com/p/88ccae4cbd82
https://blog.csdn.net/wc1695040842/article/details/89766064


