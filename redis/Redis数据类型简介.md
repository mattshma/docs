Intro
---

Redis官网对Redis的定义如下：

> Redis is an open source, BSD licensed, advanced key-value cache and store. It is often referred to as a data structure server since keys can contain strings, hashes, lists, sets, sorted sets, bitmaps and hyperloglogs.

以下是对其中一些数据类型的介绍。

String
---
String 是 Redis 中最简单的类型，这也是 Memcached 中唯一的数据类型。String 类型中常用的操作见 [Strings commands](http://redis.io/commands#string)。对于设置过期时间，除了使用 `SET`, `SETEX` 等， 还可以使用`EXPIRE`, `TTL`等命令，见[Keys commands](http://redis.io/commands#generic)。 更多关于 String 的内容此不赘述。

使用场景：
- 一般key-value
- ` SET key value [EX seconds] [PX milliseconds] [NX|XX]` 实现分布式锁

List
---
使用场景: 
- 最新消息
- 最近历史记录
- 消息队列

Hash
---
对于关系型数据，如用户信息，若使用 string 去存储，如 `set id:name "foo"` 和 `set id:age 30`，会存在多个多余的id，显然这部分冗余的数据是不需要的。但若使用 `set id "name:foo age:30"，这样对于修改部分数据来说，又显得多余，加大了序列化和反序列化的开销。这里非常适合使用 hash 类型来存储该关系。 `hmset id name "foo" age 30`。另外，当 redis 是分布式时，使用hash还能避免多个相关的key分布在不同节点上。

Set
---
- 关系型数据库（Mysql）

以下两个表，book表表示每本书的明细，tag表表示每本书对应的标签。

```mysql
test> select * from book;  
+----+-----------------------------+----------------------+  
| id | name                        | author               |  
+----+-----------------------------+----------------------+  
|  1 | 嫌疑犯X的献身                 | 东野圭吾              |   
|  2 | 写给大家看的设计书             | Robin Williams       |   
|  3 | 1984                        | George Orwell        |   
|  4 | 追风筝的人                    | 卡勒德·胡赛尼          |   
|  5 | HBase权威指南                | Lars George          |   
+----+-----------------------------+----------------------+  
```
 
tag表如下
```
test> select * from tag;
+----+---------+---------+
| id | tagname | book_id |
+----+---------+---------+
|  1 | 小说    |       1 | 
|  2 | 日本    |       1 | 
|  3 | 欧美    |       2 | 
|  4 | 科普    |       2 | 
|  5 | 小说    |       3 | 
|  6 | 欧美    |       3 | 
+----+---------+---------+
```

当需要找出标签为”日本” ，“小说”的书箱ID时，可能使用查询语句`select t1.book_id from tag t1, tag t2 where t1.tagname='日本' and t2.tagname='小说' and t1.book_id=t2.book_id`。 此时需要`tag`表做自关联。 

- redis

使用redis来做以上操作。 `book` 表使用 `Strings` 类型。 使用 `Sets` 存取 `tag` 来做差集和交集，并集的操作。

```
# src/redis-cli --raw
> set book:1:name "嫌疑犯x的献身"
> set book:2:name "写给大家看的设计书"
> set book:3:name "1984"
> set book:4:name "追风筝的人"
> set book:5:name "HBase权威指南"

> set book:1:author "东野圭吾"
> set book:2:author "Robin Williams"
> set book:3:author "George Orwell"
> set book:4:author "卡勒德·胡赛尼"
> set book:5:author "Lars George"

> sadd tag:fiction 1
> sadd tag:japan 1
> sadd tag:western 2
> sadd tag:popular_science 2
> sadd tag:fiction 3
> sadd tag:western 3

> smembers tag:western
2
3
> smembers tag:fiction
1
3
```

标签为”欧美“且”小说“的书箱ID为： `> sinter tag:western tag:fiction`  
标签为”欧美“且非”小说“的书箱ID为： `> sdiff tag:western tag:fiction`  
标签为”欧美“或”小说“的书籍ID为： `> sunion tag:western tag:fiction`  

使用场景：  
- 独立IP
- 两个对象的相同/不同信息（set 的交集，差集）
- 好友推荐（如果两个对象的交集大于设定值，则互相推荐）


Sorted set
---
Sorted set 是非常昂贵的数据，如果不是非要使用 sorted set 的一些特性，最好使用其它类型替代之。

使用场景：
- 排行榜

bitmap
---
bitmap 源于《编程珠玑》第一章的算法。适用于非A即B的场景，即这种场景只有两种状态。bitmap 用来计算这种场景下不同对象某状态出现的次数，不包括同一对象该状态重复出现的情况。以用户登陆来说， bitmap 记录的是用户是否登陆，不会记录该用户登陆多少次。

bitmap 是对位的操作，因此非常高效且省内存。注意：对于 
1. 两端数据多中间少 
2. 两端数据跨度大 
3. 数据量小

同时满足这3个条件的数据，不太适合使用bitmap。

hyperloglog
---
hyperloglog 是基数估计的算法之一，常用在大数据中。这里先介绍下基数相关的一些知识。

### 什么是基数？
如数据集 {1, 3, 5, 7, 5, 3, 1}，这个数据集的基数集（数据集中不重复的部分）为 {1, 3, 5, 7} ,该基数集的基数(cardinality，不重复元素的个数)为4。

### 为什么要有基数统计？
对于每天有几十亿请求的网站，假设需要统计不同的来源IP，如果将所有内容都放在内存中做统计，需要使用几十G的内存大小。即使使用位图来做这个计算，仍需要使用100多M的内存大小。对于这样的情况，如果能容忍小量误差，可以使用基数估计来实现这个需求。

hyperloglog 是基数估计其中的一种方法，其使用少量内存而且能快速估算基数。

### hyperloglog 在 redis中的实现

见 antirez 所说：
> The standard error of HyperLogLog is 1.04/sqrt(m), where “m” is the number of registers used.
Redis uses 16384 registers, so the standard error is 0.81%.

linear Counting算法在基数比较小的情况下误差比较小。随着基数的增长，误差开始变大。而 hyperloglog Counting 算法在基数比较大的情况下误差比较小，基数比较小的时候，其误差比 linear Counting 大很多。

> Linear counting does not work well for large cardinalities compared to HyperLogLog, but works very well for small cardinalities. Since the HLL registers as a side effect also work as a linear counting bitmap, counting the number of zero registers it is possible to apply linear counting for the range where HLL does not perform well. Note that this is possible because when we update the registers, we don’t really use the longest run of zeroes, but the longest run of zeroes plus one. This means that if an element is added and it is addressing a register that was never addressed, the register will turn from 0 to a different value (at least 1).

所以 antirez 在实现 hyperloglog 时，当基数较小的时候，使用 linear counting，当基数达到 2.5*m (m变register数目)时，使用 hll 算法。在两种算法切换的边界处， hyperloglog 的误差会比较大，可以见 [HyperLogLog](http://antirez.com/news/75)。

适用场景：
- UV
- APP独立用户 （日活用户？）
- 商品或链接被独立IP访问的次数（已登陆用户使用用户id，未登陆用户使用cookie）


Reference
---

- [An introduction to Redis data types and abstractions](http://redis.io/topics/data-types-intro)
- [Redis new data structure: the HyperLogLog](http://antirez.com/news/75)
- [Big Data Counting: How to count a billion distinct objects using only 1.5KB of Memory](http://highscalability.com/blog/2012/4/5/big-data-counting-how-to-count-a-billion-distinct-objects-us.html)
- [解读Cardinality Estimation算法（第三部分：LogLog Counting）](http://blog.codinglabs.org/articles/algorithms-for-cardinality-estimation-part-iii.html)
- [sds](http://redis.io/topics/internals-sds)

