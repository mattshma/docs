Redis sentinel 测试
===

测试
---
开两个 redis 实例，一个做 master，一个做 slave，另找三台机器开3个 sentinel 实例。其中 redis 的 master 和 slave 网络一直是通的，各个 sentinel 间网络一直是通的。

### sentinel instance number = 3, quorum = 1

- 切断 master 与其中一个 sentinel 节点的网络

测试结果：master 变为 slave，原来的 slave 成为新的 master。

### instance = 3, quorum = 2

- 切断 master 与其中一个 sentinel 节点的网络

测试结果：整体结构不变

- 切断 master 与两个 sentinel 节点的网络

测试结果: master 成为 slave，原来的 slave 成为 master。

### instance = 2, quorum = 1

- 切断 master 与一个 slave 节点的网络

测试结果： master, slave 互相切换。

最后又测试了一种情况，master 和 slave 上各一个sentinel，设 quorum = 1，当master 和 slave 之间的网络挂掉后，并不会出现 master, slave 切换的情况。

综上得出：

若 sentinel 间网络一直是通的，当 quorum 个sentinel 认为 master 为 sdown 状态，则会发生 failover，根据 slave 权重选出新的 master。若某个 sentinel 间与其它 sentinel 实例网络不通，且与master不通，该 sentinel 不会触发 failover。

过程
---

Sentinel 中有两种状态：`sdown`(Subjectively Down) 和 `odown`(Objectively Down)。sentinel 通过发送 `PING` 来判断其他实例是否存活。当一个 sentinel 在 `down-after-milliseconds` 时间没有接收到对方的有效回复，则认为对方是 sdown 状态。当 quorum 个 sentinel 都认为 master 是 sdown 状态时(sentinel 间通过发送 `is-master-down-after-milliseconds` 来判断)，将 master 标记为 odown 状态，此时将发生 failover。有效回复有如下几种：

- PING replied with +PONG.
- PING replied with -LOADING error.
- PING replied with -MASTERDOWN error.

其他情况（包括没收到回复）都被认为是无效回复。

sentinel auto discovery
---

sentinel 不需要知道 slave 和其他 sentinel 的情况，当连上 master 后，可通过 pub/sub 机制得到这些信息，channel 名是 `__sentinel__:hello`。每两秒每个sentinel 都会将自己的ip, port, runid 发送到这个 channel。若发现新的 sentinel ，则将该新的 sentinel 加到配置文件中。

疑问
---

从官方文档的描述看

>
> Quorum: the number of Sentinel processes that need to detect an error condition in order for a master to be flagged as ODOWN.   
> The failover is triggered by the ODOWN state.   
> Once the failover is triggered, the Sentinel trying to failover is required to ask for authorization to a majority of Sentinels (or more than the majority if the quorum is set to a number greater than the majority).

当 quorum 个 sentinel 认为 master 是 sdown 状态时，将 master 标志为 odown，此时触发 failover，但 sentinel 真的发生 failover 的条件是 n/2 + 1 个 sentinel（当quorum 大于 n/2 + 1 时，则为 quorum） 都认为master为 sdown 状态。与测试结果不一样。


源码分析
---

[entinelStartFailoverIfNeeded](https://github.com/beitian/redis/blob/unstable/src/sentinel.c#L3386) 是判断 failover 条件是否具备的函数，可以看到 failover 的条件是：

- master 处于 odown 状态
- 没有正在运行的 failover 进程
- 当前时间与上次 failover 时间差值大于 `failover_time * 2`。（用于判断上次 failover 的时间是否发生在不久前）

当同时满足这三个条件时，会发生 failover。

再来看下 odown 的判断条件，见 [sentinelCheckObjectivelyDown](https://github.com/beitian/redis/blob/unstable/src/sentinel.c#L3044)

```
if (master->flags & SRI_S_DOWN) {
        /* Is down for enough sentinels? */
        quorum = 1; /* the current sentinel. */
        /* Count all the other sentinels. */
        di = dictGetIterator(master->sentinels);
        while((de = dictNext(di)) != NULL) {
            sentinelRedisInstance *ri = dictGetVal(de);

            if (ri->flags & SRI_MASTER_DOWN) quorum++;
        }
        dictReleaseIterator(di);
        if (quorum >= master->quorum) odown = 1;
}

if (odown) {
       if ((master->flags & SRI_O_DOWN) == 0) {
           sentinelEvent(REDIS_WARNING,"+odown",master,"%@ #quorum %d/%d",
               quorum, master->quorum);
           master->flags |= SRI_O_DOWN;
           master->o_down_since_time = mstime();
       }
}
```
当 master 被标记为 sdown 后，接着遍历 master 的 sentinel 节点，若认为 master 处于 sdown 状态下的节点数大于 sentinel.conf 中设置的 quorum 数时，刚标记为odown 

所以现在整个过程清楚了，根据 quorum 数来判断是否 failover。


参考
---

- [sentinel](http://redis.io/topics/sentinel)

