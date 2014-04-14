twemproxy测试
===
下面就twemproxy对代理memcached的情况进行了测试，结果如下

命令测试
---
支持的命令：

- set/add/replace/append/prepend/cas
- get/gets
- incr/decr
- delete

不支持的命令：

- stats/stats items/stats slabs/stats sizes/stats settings
- flush_all

性能测试
---
比较单个memcached和twemproxy+单个memcached，及twemproxy+多个memcached，存取时间基本一致。

功能测试
---
- 自动摘除故障memcached instance及自动恢复
- 权重


数据一致性测试
---

- 当以twemproxy做代理时，若越过twemproxy直接在后端memcached存数据时，twemproxy可能取不到。

- 若分布格式为ketama（一致性hash），从一个twemproxy中设置值，从其他twemproxy都可以取出。若分布格式为random，根据服务器情况取出值。




- 自动摘除后，若恢复了，数据还在吗？

经测试，还在的。

- 设置之后取不到？

random的distribution



