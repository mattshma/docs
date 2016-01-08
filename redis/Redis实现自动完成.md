在平时做 auto complete 时，可能使用 Jquery 的 [autocomplete](http://jqueryui.com/autocomplete/)，其结果是字典排序。这里我们使用 Redis 来实现自动完成这个功能，并实现更复杂的效果。

基本实现过程
---
Redis 的 sorted set 中有如下两个命令：
- `ZRANGE`  
  用法：`ZRANGE key start stop [WITHSCORES]`。返回 key 中位置在 start 和 stop 之间的值。
- `ZRANK`   
  用法： `ZRANK key member`。返回 key 中元素的位置。

根据以上两个命令，可以得出 redis 实现自动完成的大致思路：将目标元素的所有前缀都存入 Redis 中，当在输入框中输入字母时，由 `ZRANK` 得出这些字母所有的位置，再使用 `ZRANGE` 列出所有值，这里再将目标元素做标志，返回 `ZRANGE` 结果中做标志的值即可。如 Redis 中存有 lucky, lunch 两个单词，若在输入框中输入 lu 时，需要返回 lucky 和 lunch。整个过程如下：

```
# lucky 和 lunch 的前缀都存入 sorted set 中，且完整单词以 "*" 结尾做标志。 
127.0.0.1:6379> zrange zset 0 -1
 1) "l"
 2) "lu"
 3) "luc"
 4) "luck"
 5) "lucky"
 6) "lucky*"
 7) "lun"
 8) "lunc"
 9) "lunch"
10) "lunch*"

# 根据输入的 lu，返回其在 zset 中的位置。
127.0.0.1:6379> zrank zset lu
(integer) 1

# 查出 zset 中从位置 1 开始所有含 lu 的元素
127.0.0.1:6379> zrange zset 1 -1
1) "lu"
2) "luc"
3) "luck"
4) "lucky"
5) "lucky*"
6) "lun"
7) "lunc"
8) "lunch"
9) "lunch*"

# 在代码中实现只返回 "*" 结尾的元素。
...
```
当目标元素较靠前时（如输入框中输入的单词是 courage 时， c 之后的所有元素都将被返回，这无疑是没必要的），上述方法就不太好了，这里可以换用`ZRANGEBYLEX`来实现这个功能。先看如下例子：

```
127.0.0.1:6379> zrange zset 0 -1
 1) "l"
 2) "lu"
 3) "luc"
 4) "luck"
 5) "lucky"
 6) "lucky*"
 7) "lun"
 8) "lunc"
 9) "lunch"
10) "lunch*"
11) "s"
12) "su"
13) "sup"
14) "supe"
15) "super"
16) "super*"
127.0.0.1:6379> zrangebylex zset [lu (lv
1) "lu"
2) "luc"
3) "luck"
4) "lucky"
5) "lucky*"
6) "lun"
7) "lunc"
8) "lunch"
9) "lunch*"
```
当输入 lu 时，设置 max 值为比 lu 值大点的值，这样返回的结果是很干净的。`ZRANGEBYLEX` 和 `ZRANGE` 的时间复杂度均为 `O(log(N)+M)`。

除此之外，还可以使用元素前缀作为 key, member 为目标元素。如 lucky 和 lunch: 
```
127.0.0.1:6379> zrange l 0 -1
1) "lucky"
2) "lunch"
127.0.0.1:6379> zrange lu 0 -1
1) "lucky"
2) "lunch"
127.0.0.1:6379> zrange luc 0 -1
1) "lucky"
127.0.0.1:6379> zrange lun 0 -1
1) "lunch"
...
127.0.0.1:6379> zrange lucky 0 -1
1) "lucky"
127.0.0.1:6379> zrange lunch 0 -1
1) "lunch"
```

这种方法不需要再判断 "*" 结尾的情况，因此使用越来较方便。下面还会看到这种方法的另一用处。

按 member 频率顺序返回值的实现
---

有时候可能需要的是 top n 个记录，如输入 lu 时，需要返回是使用次数最多的 lucky 和 lunch，而非使用次数较少的 luggage 。显然上面的方法不太适用于这种情况。这里的思路是给每一个前缀都建立 sorted set，然后更新目标元素所有前缀的 sorted set 中该元素的 score。如输入 lucky 后，将 key 为 l, lu, luc, luck, lucky 的所有 sorted sort 中值为 lucky 的 score 都加1。如下:

建立以目标元素前缀为 key，目标元素为 member 的 sorted set。 
```
#! /usr/bin/lua
-- top n auto complete
local word = ARGV[1]
for i=1, #word do
    redis.call("ZADD", string.sub(word,1,i), 0, word)
end
```
在 shell 中执行上述脚本
```
$ redis-cli --eval my.lua nil , lucky
$ redis-cli
127.0.0.1:6379> zrange l 0 -1
1) "lucky"
127.0.0.1:6379> zrange lu 0 -1
1) "lucky"
127.0.0.1:6379> zrange luc 0 -1
1) "lucky"
127.0.0.1:6379> zrange luck 0 -1
1) "lucky"
127.0.0.1:6379> zrange lucky 0 -1
1) "lucky"
```

以上得到 lucky 的所有前缀为 key 的 sorted set。若以后搜索了 lucky， 将这些前缀中 member 为 lucky 的sorted set 的 score 都加1。 

```
# 将所有 lucky 的前缀 key （这里以 <prefix> 代替）加1
> ZINCRBY <prefix> 1 lucky

# 每次输入前缀后 <prefix> ，返回该前缀中排位最前的几个元素即可
> ZRANGE <prefix> 0 4
```

多单词的自动完成
---
按照最基本的思路，建多个 zset。如下：
```
zadd zset:l 0 "life is wonderful"
zadd zset:l 0 "leaf and sunshine"
zadd zset:li 0 "life is wonderful"
zadd zset:le 0 "leaf and sunshine"
zadd zset:w 0 "life is wonderful"
zadd zset:s 0 "leaf and sunshine"
zadd zset:wo 0 "life is wonderful"
zadd zset:wi 0 "leaf and sunshine"
```
当输入 l 或者以上前缀时，会返回相应的值。进一步思考，当输入 l w 时，能不能自动补全呢？答案是肯定的。当输入 l w 后，若想得到 l w 共同补全的元素，很显然需要求 l 和 w 的交集。Redis 提供了 `ZINTERSTORE` 用来求 sorted set 的交集。如下：

```
127.0.0.1:6379> zinterstore zset:l_w 2 zset:l zset:w
(integer) 2
127.0.0.1:6379> zrange zset:l_w 0 -1
1) "leaf and sunshine"
2) "life is wonderful"
127.0.0.1:6379> zinterstore zset:l_wo 2 zset:l zset:wo
(integer) 1
127.0.0.1:6379> zrange zset:l_wo 0 -1
1) "life is wonderful"
```
以上实现了多字段的自动补全功能。想想看这个功能能用 set 实现吗？

设置过期时间
---
当 redis 中数据越来越多时，查找性能也会下降，所以需要清除部分使用较少的 member。这只需以后每次更新数据时，用 `expire` 重新设置下过期时间即可。 对于 `zinterstore` 命令，每次使用之后，给返回值设置一个过期时间。

Reference
---

- [Auto Complete with Redis ](http://oldblog.antirez.com/post/autocomplete-with-redis.html)
- [Two ways of using Redis to build a NoSQL autocomplete search index](http://patshaughnessy.net/2011/11/29/two-ways-of-using-redis-to-build-a-nosql-autocomplete-search-index)
