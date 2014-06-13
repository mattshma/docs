Memcached Tutorial
===

Some memcached options
---
以下介绍memcached的一些可选参数：

- -M  
内存不够时禁止LRU，并返回一个错误

- -I  
指定能存储的最大对象的大小，默认是1M，最小是1K，最大是128M

- -n  
初始（最小）chunk的大小（byte），默认是48byte

- -D  
设置 key prefixs 和 IDs 之间的分隔符。默认分隔符是 `:`，如果指定该参数，数据统计功能自动开启。如果没有使用，可以使用 `stats detail on` 来启动

- -c  
设置连接数，默认值为1024。当连接数较多时，此值需要相应调整，否则会出现部分连接timeout的情况

Slab Allocate
---
memcached使用预分配方式分配内存，这里有几个概念：

- slab  
- page  
- chunk  

默认情况下，一个 page 大小为1M，根据初始 chunk 的大小（48b）和增长因子的大小，可以得出会有多少个slab class. 每个slab可以由多个 page 构成。当一个 slab 没存储对象时，memcached不会向这个 slab 分配 page，当 slab 不够时，memcached会向这个 slab 分配 page。

commands
---
memcached的命令行语句形式如下：  

    <command name> <key> <flags> <exptime> <bytes> [noreply]\r\n
    cas <key> <flags> <exptime> <bytes> <cas unique> [noreply]\r\n

有几点需要说明的：  
1. `flags` 在memcached1.2.0及以下版本是一个任意的无符号的16bit整数， 在memcached1.2.1及以上版本是32bit整数。  
2. `exptime`的单位是s，如果该值小于30天（30\*24\*60\*60s），该值可以是相对值，如果该值大于30天，则需为时间的绝对值。若值为0，表示永不过期，除非被LRU删除。  
3. memcached新增的CAS协议(Check and Set)是为了解决多个Memcached client **并发** 修改 或者新增item的问题。其类似于“版本号”，只不过只有在新增和修改item时都会增加，删除item版本值不会减小。

in-depth  
---
- lazy expiration  
memcached内部不会监视item是否过期，而是在get时查看记录的时间戳，检查其是否过期。

- lazy remove  
删除item对象时，memcached不会释放已分配的内存，而是做删除标记，然后将标记该内存的指针放入slot回收插槽，待下次分配时可以直接使用。

Using Namespaces
---
memcached是一个很大的 `key/value` 缓存系统。有时候需要手动删除部分有规则的数据，但memcached只提供了一个`flush_all`，这些数据该怎么删除呢？可以参过构建“伪命名空间”来删除一些有规则的数。原理其实很简——使用prefix:key代替原来的key(memcached默认使用 “:” 作为prefix和key之间分隔符，可通过`-D`设置)：

    newkey = namespace_name:namespace_version:key

如namespace为"foo"，原本是`set("ten", 10)`，现在将这条命令替换成`set("foo:0001:ten", 10)`。如果需要清除这个namespace下的所有数据，只需要在下一请求过来时`increment(namespace:version)`即可，（若此处变为0002）。这时请求相应会被处理成“foo:0002:ten”，而这条key以前是不存在的，memcached返回`miss`，重新请求数据库并缓存结果。之前数据（foo:0001:ten）的内存会因为LRU或者expired time到期而被重新分配。

Data Expiry
---
除了等待LRU删除cache的数据外，设置expiration time也是让数据过期的方法。明确指定过期时间在某些场景也是一种非常有效的措施：如处理session的时候。

Memcached Log
---
Memcached有三种详细模式，参数如下：  
- -v  
最低级别的详细级别，输出各种错误消息

- -vv  
输出具体的slab、chunk信息及所有的命令

- -vvv  
输出详细的读写过程，常用来诊断与客户端之间的通信问题

需要指出的是，如果memcached要使用Log，可以通过上述参数之一再重定向至日志文件即可。但Memcached中所有的详细模式都是给调试和测试用的，大量产生这些信息会对繁忙的服务器产生显著的负面影响。同时需注意到，将错误信息输出，尤其是重定向至磁盘，可能会抵消memcached带来的性能上的增长。因此如非必需，实际生产环境和发布环境不推荐使用<sup>[0]</sup>。

Memcached Statistics
---
这里说下关于Memcached数据统计的知识，关于获取这些数据信息，除了使用标准的Memcache协议，还可以使用各个API的接口和memcached-tool之类的工具。

- stats  
在使用 `stats` 获取的信息中，根据 `get_misses` 和 `get_hits` 的比较，可以得出该memcached实例的大致情况，如果在该实例启动很长一段时间后`get_hits`仍比较小，这可能暗示分配给该实例的内存过小，需要增大内存或增多memcached实例。

- stats slabs  
在 `stats slabs` 获取的信息中，根据 `total_chunks` (或 `total_pages` ) 与 `used_chunks` 可大致估计使用最多的几个chunks，并依此调优 **增长因子**。如果不同的 slab classes 很多，而相对而言很多slab使用了较少的chunks，这一般是因为设置了不够好的增长因子。  
  `active_slabs` 表示分配出去的slab classes 总数。`total_malloceed` 表示分配给所有slab pages的总的内存大小。

- stats items  
在 `stats items` 得到的信息中，其 `items：id` 对应于 `stats slabs`中的slabID。 `number` 对应于 `used_chunks`。 而 `evicted` 代表被移除的items数目，结合 slab classes 具体信息，如果一个slab的 `evicted`数值明显大于其他slab中的 `evicted`，刚代表该memcached实例需要进行优化。  

- stats sizes  
此命令会将该实例使用的cache锁定起来，直到计算完每个item的大小后，才允许get/set操作。一般在调试或判断时可能会用到这些信息。

- stats detail on|off|dump  
对于使用了prefix的key，`stats detail dump`提供了get、set和del方面的统计。该命令需要先`stats detail on`。通过该命令可以判断数据使用的热度。因为返回的信息是以key作为记录，所以还可以根据这些信息来判断错误或者操作是否以指定的keys为核心。

slab automove
---
一般的命令是`-o slab_reassign,slab_automove`，如果一个slab class在10s内eviction的次数达到3次或以上，将会从一个最近30s内eviction次数为0的slab class中获取一个page来存储数据<sup>[1]</sup>。 slab\_automove需要启用slab\_reassign。

slab_automove有3个值：
- 0  
关闭slab automove  
- 1  
默认值，将会移动src page和dest page,每10s检查一次。
- 2  
采取极其积极的page再分配算法，在每一次eviction时，将会移入一个page至这个slab class<sup>[2]</sup>。


memcached-tool
---
memcached提供了 memcached-tool 这个工具，通过这个工具我们可以看到 memcached 的一些使用情况。如有时候某些 slab 会出现较多的 evicted，通过 memcached-tool 得到 slab 的分配和使用情况后，我们可以优化下增长因子和初始 chunk 大小，一般来说增长因子过大，会浪费一些内存；而增长因子过小，意味 slab 多，eviction queue也会增加，如果未来存储对象大小发生变化，就有更多的内存碎片，当然，重启memcached会解决这个问题。总之，根据情况设定增长因子。

some questions
---

### timeout
参考[memcached timeout](http://code.google.com/p/memcached/wiki/Timeouts)的处理过程。


annotation
---
- [0]: [memcached Logs](http://docs.oracle.com/cd/E17952_01/refman-5.6-en/ha-memcached-using-logs.html)  
- [1]: [Memcached 1.4.11 Release Notes](http://code.google.com/p/memcached/wiki/ReleaseNotes1411)  
- [2]: [Memcached 1.4.14 Release Notes](http://code.google.com/p/memcached/wiki/ReleaseNotes1414)  

List of Reference
-----
- [memcached's protocol.txt](https://github.com/memcached/memcached/blob/master/doc/protocol.txt)
- [NewPerformance](http://code.google.com/p/memcached/wiki/NewPerformance)
- [Using MySQL with memcached](http://docs.oracle.com/cd/E17952_01/refman-5.0-en/ha-memcached-using.html)
- [NewConfiguringClient](http://code.google.com/p/memcached/wiki/NewConfiguringClient)
