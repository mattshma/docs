这里介绍下 Redis 内部的几种数据结构

Redis 字符串(sds.h/sds.c)
---

Redis 是一个 key-value 型的数据库，其 key 和 value 都由字符串来实现。 C语言中的字符串不能高效的进行 `append` 和 `strlen`操作，且 Redis 需要字符串是[二进制安全的](http://en.wikipedia.org/wiki/Binary-safe)，因此Redis 自定义 sds（Simple Dynamic String） 类型来表示字符串。其定义如下：

```
/*
 * 指向 sdshdr 中的 buf 属性
 */
typedef char *sds;

struct sdshdr {
    unsigned int len;
    unsigned int free;
    char buf[];
};

static inline size_t sdslen(const sds s) {
    struct sdshdr *sh = (void*)(s-(sizeof(struct sdshdr)));
    return sh->len;
}
```

正因为 Redis 的键是二进制安全的，所以可以使用任何二进制序列作为其键值，如一张图片的内容，但太大的键值（最大键值大小为512M）既消耗内存又会增加查询成本，因此合理的键值设计也是需要的。

### 对 `struct sdshdr *sh = (void*)(s-(sizeof(struct sdshdr)));` 的解释

sdsnewlen用于新建一个 Redis 字符串。
```
sds sdsnewlen(const void *init, size_t initlen) {
    struct sdshdr *sh;

    if (init) {
        sh = zmalloc(sizeof(struct sdshdr)+initlen+1);
    } else {
        sh = zcalloc(sizeof(struct sdshdr)+initlen+1);
    }
   
    if (sh == NULL) return NULL;
    sh->len = initlen;
    sh->free = 0;
    if (initlen && init)
    memcpy(sh->buf, init, initlen);
    sh->buf[initlen] = '\0';
    return (char*)sh->buf;
}
```

首先 **Reids 字符串是一个 `struct sdshdr` 型的变量**。当以 `sdsnewlen("redis", 5);` 新建一个 Redis 字符串时，返回值是一个字符指针，该指针指向 Redis 字符串中的 `buf` 属性，同时 `len` 属性被设置为5， `free` 属性被设置为0。创建的Redis 字符串为

```
-----------  
|5|0|redis|  
-----------  
^   ^  
sh  sh->buf  
```

`sdsnewlen` 返回指向 `sh->buf`的指针， 当知道指向 `sh->buf` 的指针，怎么得到指向 `sh` 的指针呢？通过指针运算，指向 `sh->buf` 的指针减去 `len` 和 `free` 占用的空间即可得到指向 `sh` 指针地址。

怎样得到 `len` 和 `free` 的大小呢？ `sds.h` 中没有直接减去两个 `unsigned int` 的大小，而是减去 `sizeof(struct sdshdr))`。为什么这么做呢？回到 `sdshr` 中 `buf` 属性的定义，这里没有使用指针，而是使用了空数组。

在C语言中，在结构体中最后定义一个空数组是个广泛使用的常见技巧，常用来 **构成缓冲区**。比起指针，用空数组的优势在于：

- 指针需要占用int长度的空间，而空数据不占用任何空间。（节省内存）
- 数组名即最后元素的指针地址。（无需初始化，直接使用数组名）

再看 `sdsnewlen` 中分配内存时： `zmalloc(sizeof(struct sdshdr)+initlen+1)`，一次性分配了结构体和`buf`的内存大小。若使用指针，还需要给指针赋值，这样 `malloc` 和 `free` 都会增加次数。

因此上述 `(void*)(s-(sizeof(struct sdshdr)))` 得到的是指向 Redis 字符串的指针。


双向链表(adlist.h/adlist.c)
---
链表是 Redis 底层的一种通用数据结构。Redis 有两种类型的链表：双向链表和压缩链表。这里先介绍双向链表。

### 双向链表的实现

双向链表由 `list` 和 `listNode` 两种数据结构组成，其中 `list` 表示双向链表本身，而 `listNode` 表示双向链表中的每个节点。

`list` 的结构如下：
```
typedef struct list {
    listNode *head;                          /* 头指针 */
    listNode *tail;                          /* 尾指针 */
    void *(*dup)(void *ptr);                 /* 函数： 复制节点 */
    void (*free)(void *ptr);                 /* 函数： 释放节点 */
    int (*match)(void *ptr, void *key);      /* 函数： 匹配节点 */
    unsigned long len;                       /* 链表长度 */
} list;
```

`listNode` 的结构如下：
```
typedef struct listNode {
    struct listNode *prev;
    struct listNode *next;
    void *value;
} listNode;
```

value的类型为 `void *`，即可保存任何类型的值。

双向链表的结构如下图：

![双向链表](http://redissrc.readthedocs.org/en/latest/_images/graphviz-caa48c97a30851a5d976fd0b496a0da80caf4b05.png)

Redis 提供 `listIter` 来表示迭代：

```
typedef struct listIter {
    listNode *next;
    int direction;
} listIter;
```

在 [adlist.c](https://github.com/antirez/redis/blob/unstable/src/adlist.c) 中，可以看到关于链表的一些常规操作。


字典(dict.h/dict.c)
---
实现字典的方式有很多，如：

- 使用链表或者数组，但数据查找会很麻烦。适合小数据量的情况。
- 使用hash，兼顾简单高效。
- 使用二叉树，其需要两个指针指向左右孩子节点，在数据量时会消耗更多内存。

这里 Redis 的字典采用哈希表实现：每个字典使用两个哈希表。一般情况下只使用第一个哈希表，只有当 rehash 时才会同时使用两个哈希表。Redis 用链地址法来解决键冲突的问题。

字典的数据结构如下：
```
typedef struct dict {
    dictType *type;
    void *privdata;
    dictht ht[2];
    long rehashidx; /* rehashing not in progress if rehashidx == -1 */
    int iterators; /* number of iterators currently running */
} dict;
```

`dictType` 的定义如下：
```
typedef struct dictType {
    unsigned int (*hashFunction)(const void *key);                              /* 计算 hash 值的函数 */
    void *(*keyDup)(void *privdata, const void *key);                           /* 复制 key 的函数 */
    void *(*valDup)(void *privdata, const void *obj);                           /* 复制 value 的函数 */
    int (*keyCompare)(void *privdata, const void *key1, const void *key2);      /* 比较 key 的函数 */
    void (*keyDestructor)(void *privdata, void *key);                           /* 销毁 key 的函数 */
    void (*valDestructor)(void *privdata, void *obj);                           /* 销毁 value 的函数 */
} dictType;
```
不同 `dictType` 的是不同的字典，如键值复制不同， 比较key不同等。

哈希表的数据结构如下：
```
typedef struct dictht {
    dictEntry **table;
    unsigned long size;             /* 哈希表大小 */
    unsigned long sizemask;         /* 哈希表大小掩码，用于计算 hash 值，其值为 size-1 */
    unsigned long used;             /* 已使用的节点数 */
} dictht;
```
每个 `dictht[index]` 都是一个 `dictEntry *`，这样 hash 值相同的两个 `dictEntry` 能以链表保存起来。

字典中每个节点的数据结构如下：
```
typedef struct dictEntry {
    void *key;
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
        double d;
    } v;
    struct dictEntry *next;          /* 指向下一节点 */
} dictEntry;
```

以上数据结构的关系如下图：

![dict 结构](http://redissrc.readthedocs.org/en/latest/_images/graphviz-110ef3a81e3eb42cc35756d79a3f6280470c12f7.png)

更多 dict 相关的操作可参考 [dict.c](https://github.com/antirez/redis/blob/unstable/src/dict.c)。

### 添加新 key-value 

整个过程如下：

![dictAdd](http://origin.redisbook.com/en/latest/_images/graphviz-68f4129c529e0c49d38cfe664cad48af4412770a.svg)

### rehash

[源码](https://github.com/antirez/redis/blob/unstable/src/dict.c#L868)中 rehash 的条件如下：

```
if (d->ht[0].used >= d->ht[0].size &&
   (dict_can_resize || 
    d->ht[0].used/d->ht[0].size > dict_force_resize_ratio))
{
   return dictExpand(d, d->ht[0].used*2);
}
```
即：

- used:size >= 1:1 且  `dict_can_resize` 为真。
- used/size 大于 `dict_force_resize_ratio`。 （目前版本中 `dict_force_resize_ratio = 5 `，见 [源码](https://github.com/antirez/redis/blob/unstable/src/dict.c#L59)）

满足以上条件之一时，触发 rehash。

rehash 扩展过程如下：  

- 创建一个比 ht[0]->table 更大(大小至少为 `d->ht[0].used*2`)的 ht[1]->table;
- 将 ht[0]->table 中的所有键值对迁移到 ht[1]->table;
- 将原有 ht[0] 的数据清空，并将 ht[1] 替换为新的 ht[0];

字典的收缩和扩展差不多，不同的是扩展是自动进行的，而收缩需要由程序触发。

rehash 的过程是 **渐进** 的。其主要函数为 [_dictRehashStep](https://github.com/antirez/redis/blob/unstable/src/dict.c#L303) 和 [dictRehashMilliseconds](https://github.com/antirez/redis/blob/unstable/src/dict.c#L284)。

- `_dictRehashStep` 被动进行rehash， 将 `ht[0]->table` 中第一个不为空的索引上所有节点迁移至 `ht[1]->table`
- `dictRehashMilliseconds` 在指定时间（ms）内主动 rehash。
 
ht[1] 替换为 ht[0] 的[代码](https://github.com/antirez/redis/blob/unstable/src/dict.c#L244)。在 rehash 的过程中会同时使用两个哈希表，此时所有的查找，删除等操作，除了在 `ht[0]` 上进行外，还会在 `ht[1]` 上进行，而增加节点的操作只会在 `ht[1]` 上进行。

skiplist
---
[跳表](http://en.wikipedia.org/wiki/Skip_list)由 William Pugh 在论文 [Skip Lists: A Probabilistic Alternative to
Balanced Trees](http://www.cl.cam.ac.uk/teaching/0506/Algorithms/skiplists.pdf) 中提出，其效率和平衡树差不多，但比平衡树容易实现多了。

在 Redis 中，跳表用来做 Sorted Set 的底层数据结构。Redis 基于 William Pugh 描述的跳表做了部分修改，[zskiplist](https://github.com/antirez/redis/blob/unstable/src/redis.h#L581)定义如下：

```
typedef struct zskiplist {
    struct zskiplistNode *header, *tail;          /* 头尾节点指针 */
    unsigned long length;                         /* 节点数目 */
    int level;                                    /* 层数 */
} zskiplist;
```
[zskiplistNode](https://github.com/antirez/redis/blob/unstable/src/redis.h#L571) 定义如下:
```
typedef struct zskiplistNode {
    robj *obj;                                    /* member */
    double score;                                 /* socre */
    struct zskiplistNode *backward;               /* 后退指针，用于逆序迭代 */
    struct zskiplistLevel {                       /* 层 */           
        struct zskiplistNode *forward;            /* 同一水平层的前进指针 */
        unsigned int span;                        /* 该节点在这层跨跃的节点数 */
    } level[];
} zskiplistNode;
```
关于 `span` 的信息，见[span](http://stackoverflow.com/questions/10458572/what-does-the-skiplistnode-variable-span-mean-in-redis-h)。

intset(intset.h/intset.c)
---
intset 是 Set 的底层数据结构之一，其保存的元素是有序的。如果一个集合只保存整数元素且保存元素不多，那么 Redis 会使用 intset 来保存集合元素。

intset 的结构如下：
```
typedef struct intset {
    uint32_t encoding;         /* 编码方式 */
    uint32_t length;           /* 集合中元素的个数 */
    int8_t contents[];         /* 保存元素的数组 */
} intset;
```

关于 `int8_t contents[]`，这里只是申明其有 `contents` 这个属性，不会分配空间，且根据[源码](https://github.com/antirez/redis/blob/unstable/src/intset.c)来看，在使用 `contents` 时，基本都做了类型转换。

新建 intset 时，[代码](https://github.com/antirez/redis/blob/unstable/src/intset.c#L97)如下：
```
intset *intsetNew(void) {
    intset *is = zmalloc(sizeof(intset)); 
    is->encoding = intrev32ifbe(INTSET_ENC_INT16);
    is->length = 0;
    return is;
}
```
可以看到 `encoding` 使用 `INTSET_ENC_INT16` 作为初始值。

再看 [intsetAdd](https://github.com/antirez/redis/blob/unstable/src/intset.c#L204)，可以发现整个过程是：先调用 [_intsetValueEncoding](https://github.com/antirez/redis/blob/unstable/src/intset.c#L45) 返回 value 的字节数，并将其与 `encoding` 比较，若 value 使用的字节数大于 `encoding`，则将整个 intset 的编码[升级](https://github.com/antirez/redis/blob/unstable/src/intset.c#L157)再插入，若元素已在集合中，不做任何动作，否则查找到 value 的插入位置后将其插入，并更新该集合的信息。

补充说明两点：
1. 查找（[intsetSearch](https://github.com/antirez/redis/blob/unstable/src/intset.c#L115)）元素的方法是二分查找法。
2. intset 只支持升级，不支持降级。

ziplist(ziplist.h/ziplist.c)
---
ziplist 是编码后的内存块构成的列表，每个 ziplist 可以包含多个 entry， 每个 entry 保存一个字符指针。使用 ziplist 比双向链表更省内存，其将双向链表中指向前后节点的指针（32位系统上两个指针共占 8 个字节），转为存储上一节点与当前节点的大小（最好情况下占用 2 个字节）。不过当增加或删除节点时， ziplist 都需要重新分配内存，因此比较小的列表，集合，字典使用 ziplist 存储。

ziplist 的[结构](https://github.com/antirez/redis/blob/unstable/src/ziplist.c#L11)如下：

```
<zlbytes><zltail><zllen><entry><entry><zlend>
```

各项意思如下：

- zlbytes  
  整个 ziplist 所占用的字节数，保存这个值的意思在于对 ziplist 进行内存重分配时无需再遍历它。
- zltail  
  到达 ziplist 尾节点的偏移量，通过该值可以在不遍历 ziplist 的情况下弹出尾节点。
- zllen  
  ziplist 中的节点数。
- entry  
  节点元素。
- zlend  
  标记节点结束，占用一个字节，值为 255 。  

根据 [ZIPLIST ENTRIES](https://github.com/antirez/redis/blob/unstable/src/ziplist.c#L28) 所言， 每个节点的存储结构如下：

```
|<----------------- header -------------------------->|<- content ->|
|                                                     |             |
+-------------------+------------------+--------------+-------------+ 
|  上一节点占用的长度  | 当前节点占用的长度 |     编码      | 当前节点数据 |
| (pre_entry_len) |      (len)     | (encoding) |  content  | 
+-----------------+---------------+------------+-----------+
```

每个 [entry](https://github.com/antirez/redis/blob/unstable/src/ziplist.c#L157) 具体的数据结构如下：

```
typedef struct zlentry {
   
    /* 
     * prevrawlen: 前一节点的长度
     * prevrawlensize: 编码 prevrawlen 所需的字节大小
     */ 
    unsigned int prevrawlensize, prevrawlen; 

    /* 
     * len: 当前节点的字节长度
     * lensize: 编码 len 的字节长度
     */
    unsigned int lensize, len;
    
    /* 当前节点的头部大小，headersize = prevrawlensize + lensize */
    unsigned int headersize;
    
    /* 当前节点的编码类型 */
    unsigned char encoding;
    
    /* 指向当前节点的指针 */
    unsigned char *p;
} zlentry;
```

计算前一节点长度的方法如下：

- 若前一节点的长度小于 254 字节，那么使用 1 个字节来保存这个长度值。
- 若前一节点的长度大于等于 254 字节，那么使用 5 个字节来保存这个长度值：
 1. 第 1 个字节被设置为 254 ，用来说明拉下来 4 个字节用来保存长度值。
 2. 后面 4 个字节存储前一节点的真实长度。

接下来说下 ziplist 的连锁更新现象。

当插入一个节点（非尾节点）时，next 的 `prevrawlensize`, `prevrawlen` 属性均需要更新为 new 的长度。因此可能有如下三种情况（见[__ziplistCascadeUpdate](https://github.com/antirez/redis/blob/unstable/src/ziplist.c#L456)）：
- next.prevrawlen == rawlen   
  new 的大小与 prev 的大小一样，保存它们大小时同为 1 个字节或同为 5 个字节。
- next.prevrawlensize < rawlensize  
  保存 prev 的长度值需要 1 个字节， 保存 new 的长度值需要 5 个字节。
- next.prevrawlensize > rawlensize  
  保存 prev 的长度值需要 5 个字节， 保存 new 的长度值需要 1 个字节。

对于第 1，3 两种情况，更新 next 的 `prevrawlensize`，`prevrawlen` 属性即可。对于第二种情况，需要对 ziplist 重新分配内存，来扩充 next 的空间保存 new 的值，而随着 next 的空间发生变化， next+1 也要重新计算 `prevrawlen` 属性，依此类推，会发生连锁更新。同理，删除也可能有这个问题产生。不过，只有在添加/删除的节点大小大于等于 254 字节且后面同时也有多个连续的节点长度大于等于 254 字节时，这种连锁更新才会发生。


```
 add new entry 
        |
       v
+------+------+--------+-----+
| prev | next | next+1  | ... |
+------+------+--------+-----+
```

zipmap
---

以前小的 hash 使用 zipmap 实现，而被爆有[性能问题](https://github.com/antirez/redis/issues/188)后，Redis 在 2.6 及之后的版本中使用 ziplist 来实现小哈希。因此 zipmap 此不赘述。

参考
---

- [redis 源码解析](http://redissrc.readthedocs.org/en/latest/index.html)
- [Redis 设计与实现: 字典](http://origin.redisbook.com/en/latest/internal-datastruct/dict.html)
- [SkipList 跳表](http://kenby.iteye.com/blog/1187303)


