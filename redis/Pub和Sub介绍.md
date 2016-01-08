pub/sub 字面意思是发布（Publish）/订阅（Subscribe）。在 Redis 中，若对某一个 Key 进行消息发布，那么订阅该 key 的客户端都会收到相应消息。这一功能最明显的用处是用作实时消息系统，而在做消息系统时，随着订阅者增加到一定程度，Redis 可能会出现 block，这时可能需要将订阅者放到队列中，或者放在 zset（订阅者有优先级）中。

pub/sub 模式也叫观察者模式，其将发布者和订阅者解耦。对于一些需要使用观察者模式的地方，可以使用 pub/sub。

`PSUBSCRIBE` 还支持模式匹配。见 [psubscribe](http://redis.io/commands/psubscribe)。

pub/sub 的命令见 [pubsub](http://redis.io/commands#pubsub)。

