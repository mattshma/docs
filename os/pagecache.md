# Page Cache

_注：本文主要为[The Page Cache and Page Writeback](http://sylab-srv.cs.fiu.edu/lib/exe/fetch.php?media=paperclub:lkd3ch16.pdf)的笔记。_

首先看下维基百科对Page Cache的定义：
> In computing, a page cache, sometimes also called disk cache,[2] is a transparent cache for the pages originating from a secondary storage device such as a hard disk drive (HDD).

page cache是RAM中一系列存储磁盘内容的cache，目标是为了减少磁盘io。两个原因导致page cache在任何现代操作系统中都是一个非常重要的组件。其一是访问硬盘比访问内存慢太多；其二基于时间局部性原理--一个数据被访问，在不久的将来，很大可能它还会被再次访问。结合这2点，将磁盘上的数据存到cache将会带来很大的性能提升。

## Approaches to Caching

内核处理读请求时，会先在page cache中检查是否有被请求的数据，如果有的话，内核直接从RAM中读取数据并返回(cache hit)。若没有的话，内核先从磁盘中读取该数据，再将该数据写到RAM中(cache miss)，之后对于该数据的请求可直接从RAM读取。

内核处理写请求有3种不同的策略实现。第一种为`no-write`，即不缓存任何写操作，数据直接写到磁盘上，并使已缓存数据失效。由于访问磁盘比较费时，并且该方式缓存数据失效，导致读都要再从硬盘读，代价很大，所以一般内核不会使用这种策略。第二种为`write-through cache`，写操作时会同时更新RAM和磁盘中的数据。这种方式保证RAM和磁盘中数据的一致性。第三种方式为`write-back`，即内核直接在page cache上进行写操作，将page cache中被写入的page标记为dirty并加入dirty list中中。一个名为`writeback`的进程会周期性的将dirty list中的数据写回到磁盘，写回成功的page会被撤销其dirty标记。"dirty"这个词会带来误解，因为实际脏数据是在磁盘上（其上数据已过时）。由于延迟批量将数据写回到磁盘，所以`wirteback`是一种优于`write-through`的策略，不过其缺点是实际较复杂。

### Cache Eviction

一般而言，cache剔除算法为LRU算法，其能满足大部分应用，不过该算法也有缺点：许多文件只会访问一次，之后很长一段时间不会再被访问。Linux使用的是改良后的"Two-List"算法，即active链表和inactive链表，第一次访问数据时，将其放到inactive链表中，再次访问inactive链表上的数据时，将其移到active链表；这两种链表都使用伪LRU方式维护数据：数据项添加在list尾，数据项删除在list头，如同队列一样。当active链表越来越长时，将active链表头部的数据移动到inactive链表尾部。cache的剔除只发生在inactive链表中。这种方法解决了LRU不能处理只访问一次的数据的情况，"Two-List"算法也被称为LRU/2，依此类推，若有n个链表，可被称为LRU/n。

### Linux Page Cache
Linux中的page cache使用`address_space`结构体来表示pagechche，使用Radix tree快速定位目标page位置。关于内部实现，这里略过。

### Buffer Cache
在Linux 2.4之前的内核中，有两种独立的磁盘cache: page cache 和buffer cache。前者缓存page，后者缓存buffer，但实际上都是缓存磁盘block以减少磁盘访问次数，因此导致部分数据缓存2份，浪费内存，所以之后只有一种磁盘cache：page cache。内核仍需要使用buffer来表示磁盘block与page cache中page的映射关系。

### Flusher线程 

前面说过，内核会周期性的启动flusher线程将dirty数据写到磁盘。在如下几种情况会启动flusher线程：

- 剩余内存低于指定值（内存紧张），相关阈值为`/proc/sys/vm/dirty_background_ratio`。
- dirty数据缓存时间（`/proc/sys/vm/dirty_expire_centisecs`）到期，则每隔`/proc/sys/vm/dirty_writeback_centisecs`毫秒flusher进程启动一次。
- 用户调用sync或fsync

### Laptop 模式
为提高电池使用时间，内核支持laptop模式（将`/proc/sys/vm/laptop_mod`设置为1即可开启）来减少磁盘启动，即一个page到期后，刷新所以dirty page到磁盘中，显然，只有在`dirty_expire_interval`和`dirty_writeback_interval`都很大时-- 如10分钟，这模式才有意义。但这样带来的负面影响是一台系统宕机，将会丢失很多数据。



## 参考
- [The Page Cache and Page Writeback](http://sylab-srv.cs.fiu.edu/lib/exe/fetch.php?media=paperclub:lkd3ch16.pdf)
- [Examining Linux 2.6 Page-Cache Performance](https://www.kernel.org/doc/ols/2005/ols2005v2-pages-87-98.pdf)
- [从Apache Kafka 重温文件高效读写](http://calvin1978.blogcn.com/articles/kafkaio.html)
