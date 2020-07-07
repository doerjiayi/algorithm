
##单实例支撑每天上亿个请求的SSDB
http://www.ideawu.net/blog/archives/736.html

SSDB 是一个 C++ 开发的 NoSQL 存储服务器, 支持 zset, map 数据结构, 可替代 Redis, 特别适合存储集合数据. SSDB 被开发和开源出来后, 已经在生产环境经受了3个季度的考验, 一直稳定运行.

在一个支撑数千万用户的列表数据(例如用户的订单历史, 用户的好友列表, 用户的消息列表等)的实例上, SSDB 每天处理上亿个读写请求, 仍然能保持 CPU 占用在3%左右, 内存占用为 1G. 这种数据规模是我们原来使用的 Redis 所无法满足的, 因为 Redis 无法保存如此大量的数据, 物理内存的容量限制了 Redis 的能力. 根据我们的经验, Redis在10G数据规模时比较适用, 数据规模再扩大时, Redis 就非常吃力, 而且几乎无法扩展. 这时, 必须改用 SSDB.

SSDB 具有和 Redis 高度重合的 API, 而且对于 hash(map) 还是可分段遍历的, 相比较, Redis 只能通过 hgetall 一次遍历 hash 中的所有元素, 在大的 hash 中, 这个操作非常低效.

如果要列出几条必须放弃 Redis, 改为使用 SSDB 的观点, 我相信这几条非常有吸引力:
单个实例的存储容量相当于 100 个 Redis 实例!

内存占用只有 Redis 的一千分之一(最大设计容量时).
所有的数据集合(包括 KV)都是可分段(分页)遍历的.

特别适合存储列表等集合数据.
SSDB 是一个开源的项目(https://github.com/ideawu/ssdb), 你可以免费获取它的源码, 并且不需要编程和修改配置文件就可以启动服务器.
