为什么需要消息队列？
---
对于很多耗时很长的操作来说，不可能使用同步的方式来通知用户。或者对于一些处理能力有限的系统，当任务并发到一定程度时，系统不能正常完成任务。对于这些场景，可以使用队列将需要按序执行的任务先保存起来。

一些消息队列如 RabbitMQ, ActiveMQ, ZeroMQ 来实现这些功能是蛮简单的，但是这些消息队列可能对于平日编程而言有点重，或者不能满足优先级等需要。Redis 的 List 提供了阻塞功能，可以避免对队列的轮询。另外 pub/sub 也可以用来实现消息队列的功能。这里用 Redis 的 List 来实现消息队列。

简单的消息队列
---
如下是用 Python 写的一个简单的消费者代码。

```
import redis

def consumer(task):
        print task

def get_task():
        r = redis.Redis()
        while True:
                task = r.brpop('tasklist', 0)
                consumer(task[1])

if __name__ == '__main__':
        get_task()
```

因为 tasklist 队列为空，所以运行这个 python 脚本时 brpop 阻塞住了请求。现在往 tasklist 中写数据，直接在 Redis 命令窗口中 lpush 数据 。

```
# 生产者端
127.0.0.1:6379> lpush tasklist "task 1"
(integer) 1
```

可以看到消费者端已消费数据。

优先级队列
---
有时候可能需要优先级队列，如 VIP 用户与普通用户的区别。当 VIP 用户来时，需要优先处理。这里使用两个队列，一个用来保存 VIP 的请求， 一个用来保存普通用户的请求。假设保存 VIP 请求的队列为 high_queue，保存普通用户请求的队列为 low_queue，如下 Python 代码：

```
import redis

def consumer(task):
        print task

def get_task():
        r = redis.Redis()
        while True:
                task = r.brpop(['high_queue', 'low_queue'], 0)
                consumer(task[1])

if __name__ == '__main__':
        get_task()
```

仍然在 Redis 命令行中 lpush 数据。可以看到结果。

更复杂的需求
---

如果有很多不同优先级的需求呢？如优先级等级从 0 到 10000 呢？这时候使用一个队列，如队列 [1, 20, 25,...3999]，当 优先级为 23 的请求过来时，可先使用 lset 将 23 插入到队列中去。插入的过程是一个查找的过程，所以尽量选择一些高效的查找算法。结合多个队列处理不同请求，如 1--1000 放一个队列，2000-3000 放一个队列，可更快的提高速度。此不赘述。

