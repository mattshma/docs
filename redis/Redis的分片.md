[partitioning](http://www.redis.io/topics/partitioning)

优点：
- 利用更多 cpu 和内存

缺点：
- 集合 交，差，并 等操作可能无法完成
- 事务涉及多个字段
- 数据管理更复杂
- 添加删除节点更复杂

store or cache
----
warning:

- If Redis is used as a cache scaling up and down using consistent hashing is easy.
- If Redis is used as a store, **we need to take the map between keys and nodes fixed, and a fixed number of nodes**. Otherwise we need a system that is able to rebalance keys between nodes when we add or remove nodes, and currently only Redis Cluster is able to do this, but Redis Cluster is currently in beta, and not yet considered production ready.

:+1: 

Presharding
---
由上知 Redis 做 store 时，在 Redis 规模变化时，需要做数据迁移。而随着数据的急剧增长，Redis 节点增加是很常见的操作。但数据怎么迁移呢？怎么知道哪些数据是需要迁移的？一个干净的 Redis 实例占用的内存不到 1M. 因此可以在最开始设计时，就指定多个实例，即使当时只使用一个 Redis Server。当以后节点增加时，只需要将节点从老的 Redis Server 迁到新 Redis Server 即可。过程如下：

- Start empty instances in your new server.
- Move data configuring these new instances as slaves for your source instances.
- Stop your clients.
- Update the configuration of the moved instances with the new server IP address.
- Send the SLAVEOF NO ONE command to the slaves in the new server.
- Restart your clients with the new updated configuration.
- Finally shut down the no longer used instances in the old server.

虽然不得不说 @antirez 很机智，但是，切换过去之后，新 redis 实例 hash 出来的位置应该和原实例 hash 出来的位置不同啊！

分片的实现
---

### 应用端
如 php.ini 中设置一致性 Hash。

### 中间代理层 Twemproxy
这是 twitter 开源出来做 redis 和 memcached 代理的。支持 pipeline, 分片，自动摘除故障节点等诸多优点，但在cache 和 app 之间再加代理，无疑对性能还是有影响的，虽然 twemproxy 的影响不是很大，所说在最差的情况下有 20% 的影响。在 redis sentinel 出现之前， twemproxy 也是 antirez 推荐的做法。但 twemproxy 在使用时，还需要多测试。且 twitter 自己都没有用 twemproxy。

### redis sentinel

略。 


参考
---
- [partitioning](http://redis.io/topics/partitioning)



