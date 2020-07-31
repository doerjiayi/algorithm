##MySQL · 特性分析 · MyRocks简介
https://www.cnblogs.com/xueqiuqiu/articles/10993936.html

##你觉得MyRocks怎么样？
https://www.zhihu.com/question/271844793?sort=created

##MyRocks及其使用场景分析
https://zhuanlan.zhihu.com/p/45652076
总结
总的来说，相比InnoDB，MyRocks占用更少的存储空间，能够降低存储成本，提高热点缓存效率；具备更小的写放大比，能够更高效利用存储IO带宽；将随机写变为顺序写，提高了写入性能，延长SSD使用寿命；通过参数优化降低了主从复制延迟。因此，在数据量大、写密集型等业务场景下非常适用。此外，作为同样的MySQL写和空间优化方案，MyRocks具有更好的社区生态，适合用于替换TokuDB实例。MyRocks高效的缓存利用率，成熟的故障恢复和主从复制机制，使得其也可以作为Redis的持久化方案。

##MySQL · myrocks · myrocks之事务处理 
http://mysql.taobao.org/monthly/2016/11/02/

##网易 MyRocks 使用和优化实践
https://www.infoq.cn/article/z0agILsTCFuxa_sycTIK

##Innodb vs MyRocks vs Tokudb
https://zhuanlan.zhihu.com/p/57929571
