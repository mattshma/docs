redis Persistance
---

当仅要求高可用时，可以考虑 Redis Persistence 的 `AOF` 方案（当日志文件越来越大时，使用 `bgrewriteaof` 重写）。 注意数据较大时，在 AOF rewrite 的过程中可能会出现服务暂停（因为此时机器的I/O和load很高）的情况，此时可以考虑使用 `no-appendfsync-on-rewrite yes` 来关掉 rewrite 时 fsync 操作，直到 rewrite 完成，才将保存在内存中的数据写到磁盘中去。

Redis 在写 rdb 和 aof 文件的时候，都会先 folk 一个子进程来完成这个操作。Redis使用了Linux的 "Copy-on-write" 的思想（将旧数据copy至新的内存空间。因此对于频繁写的数据，folk出来的子进程占用的内存最坏和父进程占用的内存一样多），对机器的性能和内存影响较大，所以这里需要考虑用主从复制来保证数据的完整性。

Redis Replication 
---
使用两台机器A和B， A做master，不做持久化，B做 slave，做 AOF 的持久化。在 B中配置 `slaveof A'ip A'port` 后，启动服务即可。对Replication而言，可能使用 keepalived 或 sentinel 保证 failover 并发送报警。这里采用 sentinel。

sentinel
---
 sentinel的机制如下：

若有N个 sentinel 实例，且 sentinel 实例中 quorum 数设为M， 则当有M个 sentinel 实例都认为 master unreachable 时，这M个 sentinel 将master标记为 `SDOWN`，然后触发 failover, 此时所有 slave 开始选举， slave priority最低的当选（除0外），若priority无法选出，则根据`runid`选。当至少 `N/2 + 1` 个slave都认为某个slave可当选时其才当选。否则隔 `failover-timeout` 时间后再次选举。

sentinel 根据 sub/pub 构取slave的信息及其他sentinel结点的信息，因此sentinel的配置文件中只需配置 master 信息即可。
 
以下一份redis配置文件：

```
port 26379
dir "/tmp"
sentinel monitor mymaster 10.10.3.180 6379 1
sentinel down-after-milliseconds mymaster 2000
sentinel failover-timeout mymaster 10000
```

长连接？
---
将redis.conf中的`tcp-keepalive`设为60.

方案
---
做Master/Slave架构，目前想到有如下几种方法： twemproxy，Keepalived，Sentinel。当然，还有最古老的人工去操作 :relaxed: 

twemproxy在新的节点加入后，不会自动识别新节点？需要写脚本做这个操作；  
keepalived针对是整个服务器，粒度太大；  
sentinel在新节点加入后的表现？

Reference
---

- [Redis persistence](http://redis.io/topics/persistence)
