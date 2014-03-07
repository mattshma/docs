Memcached Tutorial
===

Some memcached options
---
以下介绍memcached的一些可选参数：

- M  
内存不够时禁止LRU，并返回一个错误。


commands
---
memcached的命令行语句形式如下：  

    <command name> <key> <flags> <exptime> <bytes> [noreply]\r\n
    cas <key> <flags> <exptime> <bytes> <cas unique> [noreply]\r\n

有几点需要说明的：  
1. `flags` 是一个任意的无符号的16bit整数。  
2. memcached新增的CAS协议(Check and Set)是为了解决多个Memcached client 并发 **修改** 或者 **新增** item的问题。其类似于“版本号”，只不过只有在新增和修改item时都会增加，删除item版本值不会减小。

in-depth  
---
- lazy expiration  
memcached内部不会监视item是否过期，而是在get时查看记录的时间戳，检查其是否过期。

- lazy remove  
删除item对象时，memcached不会释放已分配的内存，而是做删除标记，然后将该标记内存的指针放入slots回收插槽，待下次分配时可以直接使用。

Using Namespaces
---
memcached是一个很大的 `key/value` 缓存系统。有时候需要手动删除部分有规则的数据，但memcached只提供了一个`flush_all`，这些数据该怎么删除呢？可以参过构建“伪命名空间”来删除一些有规则的数。原理其实很简——将原来的key重新构造成新key之后再操作：

    newkey = namespace_name + namespace_version + key

如namespace为'foo'，原本要执行的命令是`set("ten", 10)`，现在将这条命令替换成`set("foo_0001_ten", 10)`。如果需要清除这个namespace下的所有数据，只需要在下一请求过来时`increment(namespace_version)`即可，（设此处变为0002）。这时请求相应会被处理成“foo\_0002\_ten”，而这条key以前是不存在的，memcached返回`mis`，重新请求数据库并缓存结果。之前数据（foo\_0001\_ten）的内存会因为LRU或者expired time到期而被重新分配。

Data Expiry
---
除了等待LRU删除cache的数据外，设置expiration time也是让数据过期的方法。明确指定过期时间在某些场景也是一种非常有效的措施：如处理session的时候。

List of Reference
---
- [memcached's protocol.txt](https://github.com/memcached/memcached/blob/master/doc/protocol.txt)
- [NewPerformance](http://code.google.com/p/memcached/wiki/NewPerformance)
