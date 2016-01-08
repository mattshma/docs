这里介绍下 Redis 对象的一些数据结构。

redisObject
---

`redisObject` 是 Redis 的核心对象。Redis 中的键、值、参数等都表示为该对象。其数据结构如下：

```
typedef struct redisObject {
    unsigned type:4;             
    unsigned encoding:4;           
    unsigned lru:REDIS_LRU_BITS;  /* lru time (relative to server.lruclock) */
    int refcount;                 /* 引用计数 */
    void *ptr;                    /* 指向对象的指针 */
} robj; 
```

而其中
```
/* Object types */
#define REDIS_STRING 0
#define REDIS_LIST 1
#define REDIS_SET 2
#define REDIS_ZSET 3
#define REDIS_HASH 4

/* Objects encoding. Some kind of objects like Strings and Hashes can be
* internally represented in multiple ways. The 'encoding' field of the object
* is set to one of this fields for this object. */
#define REDIS_ENCODING_RAW 0          /* Raw representation */
#define REDIS_ENCODING_INT 1          /* Encoded as integer */
#define REDIS_ENCODING_HT 2           /* Encoded as hash table */
#define REDIS_ENCODING_ZIPMAP 3       /* Encoded as zipmap */
#define REDIS_ENCODING_LINKEDLIST 4   /* Encoded as regular linked list */
#define REDIS_ENCODING_ZIPLIST 5      /* Encoded as ziplist */
#define REDIS_ENCODING_INTSET 6       /* Encoded as intset */
#define REDIS_ENCODING_SKIPLIST 7     /* Encoded as skiplist */
#define REDIS_ENCODING_EMBSTR 8       /* Embedded sds string encoding */
```

`robj` 定义了一个 redis Object 的结构，`type` 和 `encoding` 意思如上， `refcount` 保存了对象的引用次数，新建一个对象时，其 `refcount` 属性被设为 1，以后每次被引用时， 该字段加 1，取消对对象的引用时，该字段减 1，若该字段值为 0，该对象值将会被销毁。 `ptr` 属性指向保存该对象的底层数据结构。如当 `type` 为 `REDIS_LIST`，`encoding` 为 `REDIS_ENCODING_LINKEDLIST` 时，那么可以知道，这实际定义了一个 List 对象，该对象保存在一个双向链表中， `ptr` 指针指向链表，保存的数据结构类型为 `list`。

下面是 Redis Object，Redis Type 与 Redis Object Encoding 的一个关系图，其关系可以见[源码](https://github.com/antirez/redis/blob/unstable/src/object.c)。



从上可以看到 `REDIS_ENCODING_RAW` 和 `REDIS_ENCODING_EMBSTR` 都使用了 `sdshdr` 做为底层数据结构，而它们的区别在于`REDIS_ENCODING_RAW` 分两次分别申请 `robj` 和 `sdshdr` 的内存，而 `REDIS_ENCODING_EMBSTR` 一次性申请需要的内存大小（`sizeof(robj)+sizeof(struct sdshdr)+len+1)`）。 `REDIS_ENCODING_EMBSTR` 好处在于一次申请和释放所使用的内存，且使用一块连续的内存数据结构，有更好的缓存性能，见[EMBSTR](https://gitcandy.com/Repository/Commit/redis/0b740ddc351e75253519b5f0eecca6d73e3cb369)。 当字符串小于等于 `REDIS_ENCODING_EMBSTR_SIZE_LIMIT` 大小（目前`#define REDIS_ENCODING_EMBSTR_SIZE_LIMIT 39`，之前版本定义为 32 字节）时，使用 `REDIS_ENCODING_EMBSTR` 存储，否则使用 `REDIS_ENCODING_RAW` 存储。 

当字符串为整数时， Redis String 会将 `sdshdr` 的 `ptr`属性[设为](https://github.com/antirez/redis/blob/unstable/src/object.c#L104) `long` 型，并将 `encoding` 属性设为 `REDIS_ENCODING_INT`。`REDIS_ENCODING_RAW` 和 `REDIS_ENCODING_EMBSTR` 的 `ptr` 属性为`sdshdr` 型。

对于其他四种类型，`redis.conf` 定义了存储分界值。如下 
```
# hash 对象保存的所有键值对的 **键**、**值** 长度都小于 64 字节，且键值对数量小于 512 个时，使用 ziplist 保存 hash 对象。
hash-max-ziplist-entries 512
hash-max-ziplist-value 64

# list 对象保存的所有键值对的 **键**、**值** 长度都小于 64 字节，且键值对数量小于 512 个时，使用 ziplist 保存 list 对象。
list-max-ziplist-entries 512
list-max-ziplist-value 64

# 仅当 set 对象的元素值为不大于 64 位的整数字符串组成才能使用 intset 结构。
# set 对象的键值对数量小于 512 个时，使用 intset 保存 set 对象。
set-max-intset-entries 512

# zset 对象保存的所有键值对的 **键**、**值** 长度都小于 64 字节，且键值对数量小于 128 个时，使用 ziplist 保存 zset 对象。
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
```


