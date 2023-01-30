### Redis Basics

#### redis性能
 - 10W/s QPS
 - 1W+ 长连接
 - 持久化 rdb 
 - 最复杂的Zset 6kw数据 写入1k/s 读取5k/s 平均耗时5ms

#### redis是否单线程
  通过unlink的异步删除原理，可知redis并非单线程，redis单线程是指redis处理网络请求和指令的执行时单个线程串行操作的。其他的数据持久化、异步删除、集群数据同步等，是由额外的线程执行的。redis6.0版本针对网络请求引入多线程机制
  
#### redis数据结构及底层数据结构
 - String -sds 字符串 (最大存储容量512M)
 - Hash -(ziplist, dict) 字典
 - List -(zpilist, quicklist) 列表
 - Set -(intset, dict) 集合
 - ZSet -(ziplist+skiptable) 有序集合
 - Pubsub 发布订阅
 - Bitmap 位图
 - GEO 地理位置
 - Stream -(radix-tree) 流5.0
 - Hyperloglog 基数统计

#### redis协议
 redis是文本协议
 - RESP 以CRLF结尾
 - RESP3 reids6启用，增加客户端缓存

#### Redis持久化方式
  生产环境一般采用RDB模式
 - RDB
 - AOF (类似Binglog row模式)
 - 混合模式: RDB+AOF

#### Redis O(n)指令
 - keys *
 - hgettall
 - smembers
 - sunion
 - ...
 建议在集合大小不确定的时候，使用`scan` `hscan` `zscan` 替代，另外`keys`建议`RENAME`

#### Redis性能优化
- unlink 删除key -> 异步避免阻塞
- pipeline 批量传输，减少网络RTT -> 减少频繁网络交互
- 多值指令(mset, hmset) -> 减少频繁网络交互
- 关掉aof -> 避免io_wait

#### Redis扩展方式
 - lua
 - redis-module (偏向底层开发)

#### Redis问题排查
 - monitor指令 回显所有执行的指令，可以使用`grep`配合
 - keyspace-events 订阅某些key的事件，顶层实现基于pubsub
 - slow log 满查询
 - --bigkeys 启动redis大key健康检查，使用`scan`的方式执行，不用担心阻塞
 - memory usage key、memory stats 指令
 - info指令，关注`instantaneous_ops_per_sec` `used_memory_human` `connected_clients`
 - redis-rdb-tools rdb线下分析

#### Redis淘汰策略
  被动删除 (只有被get时删除并返回nil，属于惰性删除)
  主动删除 (100ms运行一次，随机删除持续25ms， 类似Cron)
  内存使用超过maxmemory，触发主动清理策略
  - 默认 `volatile-lru` 从设置过期数据集里查找最近最少使用
  - `volatile-ttl` 从设置过期的数据集里面优先删除剩余时间短的key
  - `volatile-random` 从设置过期的数据集中任意选择数据淘汰
  - `allkeys-lru`
  - `allkeys-lfu`
  - `allkeys-random` 数据被使用频率最少的优先淘汰
  - `no-enviction`
  如果不设置`maxmemory`，redis将一直使用内存，直到触发操作系统的OOM-KILLER

#### Redis集群模式
 - 单机
 - 单机多实例
 - 主从 (1+n)
 - 主从 (1+n) & 哨兵 (3或奇数个)
 - Redis Cluster (推荐， 但使用有限制) [Redis Cluster参考](https://mp.weixin.qq.com/s?__biz=MzA4MTc4NTUxNQ==&mid=2650520049&idx=1&sn=17a260f877b3ac4ff2b393981213ff94&chksm=8780be35b0f737233d9039cf33c0141a31f337febab06dc68ae052b0ee2056fe841c35aa9e00&scene=21#wechat_redirect)
 [集群选择](https://mp.weixin.qq.com/s/F-R9rv0L1opkzjA-mt-Rog)
 [Docker搭建Redis Cluster](https://juejin.cn/post/6992872034065727525)
 大规模
 - twemproxy
 - codis
 - 基于Netty Rdis协议自研
 - 管理平台 CacheCloud

#### Redis使用场景
 - 缓存 (缓存一致性 缓存穿透 缓存击穿 缓存雪崩)
 - 分布式锁
 - 分布式限流
 - session
 ##### Api举例：
 - zset 排行榜，排序
 - bitmap 用户签到 在线状态
 - geo 地理位置 附近的人
 - stream 类似kafka消息流
 - hyperloglog 每日访问ip数统计
##### 缓存一致性问题
 - 写入：缓存和数据库是不同的组件，存在只有一个写成功的可能性，造成数据不一致
 - 更新：更新两个不同的组件
 - 读取：保证读取到的信息是最新的，和数据库中一致
 - 删除：删除数据库记录时，如何保证缓存中也被删除
 建议使用：Cache Aside Pattern
 读请求：
 - 先读cache，再读db
 变更操作：
 - 先操作数据库，在淘汰缓存
 涉及到复杂的事务和回滚操作，可以把淘汰放在finally里
 问题：缓存淘汰失败！(概率低，可定时补偿)
 [缓存一致性参考](https://juejin.cn/post/7168291215753314311)
 [缓存更多细节问题](https://www.cnblogs.com/hunna/p/11942688.html)

##### 缓存击穿
 影响轻微
 高流量下，大量请求读取一个`失效的key` -> redis miss -> 穿透到DB
 解决方式：采用分布式锁，只有拿到锁的第一个线程去请求数据库，然后插入缓存

##### 缓存穿透
 影响一般
 访问一个`不存在的key` -> redis miss -> 穿透到DB
 解决方式：
 - 给相应的key设置一个null值
 - BloomFilter预先判断

##### 缓存雪崩
 影响严重
 大量key同时失效|redis宕机 -> redis miss -> 压力到DB
 解决方式：
 - 给失效时间加上相对的随机数
 - 保证redis高可用

##### 缓存热点
 - 预热找到热点数据，可通过`Spark`实时分析
 - 将集中化流量打散，避免节点过载，由于只有一个key可在key后拼加`有序编号 key#01....`
 - 每次请求时，客户端随机访问

##### 缓存大Key
 当访问缓存时，如果key对应的value过大，读写加载容易超时，容易引发网络拥堵，缓存字段较多时，每个字段的变更都会引发缓存数据的变更，导致慢查询
 设计缓存时要注意`缓存的粒度`，既不能过大，也不能过小，会导致查询多次
 解决方式：
 - 设置一个阈值，当value的长度超过阈值时，对内容进行压缩，降低kv大小
 - 评估`大Key`的比例，采用`池化技术`，预先分配大对象空间
 - 颗粒划分，将大key拆分为多个小key，独立维护
 - 大key设置合理过期时间，尽量不淘汰大key

#### 分布式锁
 redis分布式锁建议使用redisson的redlock，基础命令是setnx
 ```
 setnx -> SET key value [EX seconds|PX milliseconds|KEEPTTL] [NX|XX] [GET]
 ```
 分布式锁关键点：
 - 原子性
 - 锁超时
 - 死锁
 - 读写锁
 - 故障转移
 
 分布式锁的坑
 - 线程1和线程2同一时刻访问业务，线程2获取锁成功，进行业务处理，线程1获取锁失败，但释放锁，线程3尝试获取，但线程2业务没有处理完成，导致线程2和线程3重复执行业务
 解决问题：再确认获取锁成功后才允许释放锁
 - 缓存失效问题
 redis存在内存清理机制，可能导致分布式锁失效: `allkeys-lru` `allkeys-random`策略

#### redis慢查询原因及解决
 - 背景分析：
 redis查询数据或获取锁并且context会设置超时，会产生报错
 keys指令如果同时打到redis，单线程会使其他业务指令等待n*x毫秒，业务导致key数量暴涨，keys执行指令时间变长
 - 解决方案
 将用户维度数据放到db，跟业务数据缓存db区分
 由于keys指令的O(n)特性，禁止keys指令操作
 大key的删除del，也可能导致redis主线程响应延迟，所以线上命令统一为unlink异步删除，unlink原理类似`空间换时间`，当执行unlink会使当前key失效，由子线程异步清除当前key所占用空间，新业务会分配新的内存空间，并非所有的unlink操作都是异步，对于string类型，或set、hash等数据结构(其元素个数不超过64)，主线程会同步删除，非阻塞删除
 - 关于context设置
 只用一个context，设置合理超时时间(1s太少)
 在redis查询失败后，查询数据库之前，基于context.Background()重新设置新超时时间的ctx查询数据库