# Redis源码阅读指导

## String

> sds.h/sdshdr

内存实现采用Simple Dynamic String（SDS），不同长度的字符串使用不同的结构体，在数据类型较少时采用byte，short表示

```C
struct __attribute__ ((__packed__)) sdshdr5 {
    unsigned char flags; /* 3 lsb of type, and 5 msb of string length */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr8 {
    uint8_t len; /* used */
    uint8_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr16 {
    uint16_t len; /* used */
    uint16_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr32 {
    uint32_t len; /* used */
    uint32_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr64 {
    uint64_t len; /* used */
    uint64_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
```

```c
sds sdscatlen(sds s, const void *t, size_t len) {
    size_t curlen = sdslen(s);

    s = sdsMakeRoomFor(s,len);
    if (s == NULL) return NULL;
    memcpy(s+curlen, t, len);
    sdssetlen(s, curlen+len);
    s[curlen+len] = '\0';//结尾拼接\0标志结束
    return s;
}
```

不使用C语言的字符串的原因？

- 复杂度：获取长度C语言中的时间复杂度是O(n)，Redis自定义SDS的复杂度是O(1)

- 缓冲区溢出：C语言中不记录长度，拼接可能导致缓冲区溢出，Redis中有保障

    Redis对SDS进行修改时，先判断空间是否充足，不充足会扩展空间至需求大小再操作

- j减少内存重分配：C语言中内存重分配可能需要系统调用，频繁操作耗时，Redis中采用空间预分配和惰性空间释放

- 二进制安全：C语言字符中间的'\0'识别可能出问题，SDS的API是二进制安全的

- 兼容C：遵循C的末尾'\0'惯例，用于支持复用部分函数库<String.h>

## List

> adlist.h/listNode

定义

```c
typedef struct listNode {
    struct listNode *prev;
    struct listNode *next;
    void *value;
} listNode;

typedef struct listIter {
    listNode *next;
    int direction;
} listIter;

typedef struct list {
    listNode *head;
    listNode *tail;
    void *(*dup)(void *ptr);
    void (*free)(void *ptr);
    int (*match)(void *ptr, void *key);
    unsigned long len;
} list;
```

特点：

- 双向链表
- 无环
- 头尾指针
- 具有链表长度计数器
- 多态：通过为链表设定不同类型的函数，链表可用于存储不同类型的值

## HashMap

> dict.h/dictht

定义

```c
typedef struct dictht {
    dictEntry **table;//hash表数组，dictEntry
    unsigned long size;//hash表大小
    unsigned long sizemask;//hash表掩码 = size - 1，用于计算索引值
    unsigned long used;//已有节点数量
} dictht;

typedef struct dict {
    dictType *type;
    void *privdata;
    dictht ht[2]; //定义了两个表，实际上只有一个在使用另一个在rehash时会用到
    long rehashidx; /* rehashing not in progress if rehashidx == -1 */
    unsigned long iterators; /* number of iterators currently running */
} dict;
```

```c
typedef struct dictEntry {
    void *key;
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
        double d;
    } v;
    struct dictEntry *next;
} dictEntry;
```

查询，当处于渐进式rehash时会同时查询两个表

### 算法

MurmurHash2

### 冲突

Redis处理策略：链地址法（冲突的多个节点使用next指针连接为一个单链表）

### 空间调整

过程：

1. 分配空间，设置ht[1]的空间
    - 扩容大小：[>= used * 2] 的 2^n
    - 缩容大小：[>= used * 2] 的 2^n
2. ht[0]中节点转移至ht[1]
3. 迁移完成，释放ht[0]，ht[1]更名为ht[0]，申请空ht[1]用于下次调整

扩容触发时机：

> load_factor = ht[0].used / ht[0].size

- Server未在执行{BGSAVE | BGREWRITEAOF}，hash表的负载因子[>= 1]
- Server在执行{BGSAVE | BGREWRITEAOF}，hash负载因子[>= 5]

缩容触发时机

- hash负载因子[<= 0.1]

### 渐进式ReHash

过程：

1. 分配ht[1]空间，字典同时持有ht[0]，ht[1]
2. 字典维护索引计数器rehashidx = 0，**START**
3. rehash期间对字典的CRUD操作，除了命令被执行外，顺带会将ht[0]在rehashidx索引上的所有键值对rehas到ht[1]，rehash完成，rehashidx--
4. 步骤3持续执行，直至完成，则rehashidx = -1，**END**

## SortedSet

> server.h/zskiplistNode	跳跃表节点定义
>
> server.h/zskiplist	跳跃节点信息

```c
typedef struct zskiplistNode {
    sds ele;
    double score;                       //分值
    struct zskiplistNode *backward;     //后退分值
    struct zskiplistLevel {
        struct zskiplistNode *forward;  //前进指针
        unsigned long span;             //跨度
    } level[];
} zskiplistNode;
```

同一跳跃表中各个节点保存的对象是唯一的，但是多个节点的值是相同的

分值相同的节点将按照对象在字典序中的大小来排序，成员对象较小的节点会排在前面（考近表头），大的排在后面（靠近表尾）

## Set

## IntSet

集合键底层实现之一

> intset.h

```c
typedef struct intset {
    uint32_t encoding;      //编码方式
    uint32_t length;        //元素长度
    int8_t contents[];      //元素存储数组，存储顺序:小--->大，不重复
} intset;
```

整数集合，当一个集合只包含数值元素，且元素不多时使用

### 升级

新加入数值的长度大于原数组元素的最大长度，将用新数组的长度存储所有数据，复杂度O(n)

好处：提升数组灵活性（C是静态语言，避免类型错误不允许不同类型存放在一起），节约内存

### 降级

不支持

## ziplist

列表键&hash键的底层实现之一

列表包含少量数据&每个列表项都是小整数值 | 长度较段的字符串

previous_entry_length记录压缩列表中前一个结点的长度，单位：字节，取值：{1,5}

- 前一个节点的长度[< 254]，设置为1字节
- 前一个节点的长度[>= 254]，设置为5字节，首字节设置为0xFE(十进制254)之后4字节保存前一个节点长度

encoding保存了数据类型和长度

- 1Byte | 2Byte | 5Byte，值最高位是 00， 01，10，标志节点数据保存着字节数组，数组长度编码除去最高两位
- 1Byte，值最高位是11，content保存着整数值，值长度由编码除去最高两位

### 连锁更新

增加节点，删除节点会触发连续压缩

连锁更新影响性能情况

- 列表中有很多节点值处于250~253之间
- 被更新的节点要很多

实质情况，上述情况发生的几率很低

# 对象

基于数据结构构建了对象系统：

- 字符串对象
- 列表对象
- 哈希对象
- 集合对象
- 有序集合对象

结构定义

> server.h

```c
typedef struct redisObject {
    unsigned type:4;
    unsigned encoding:4;
    unsigned lru:LRU_BITS; /* LRU time (relative to global lru_clock) or
                            * LFU data (least significant 8 bits frequency
                            * and most significant 16 bits access time). */
    int refcount;
    void *ptr;
} robj;
```

### 类型检查

依据type

### 多态命令

我们的通用命令在底层的实现并不是恒定的

使用LLEN获取长度，对于不同的数据结构，其长度计算和获取是不同的，而对具体命令的选取除了确保LLEN操作对象是列表键外，还需要依据键的值对象使用的编码使用正确的实现

DEL，EXPIRE，TYPE——基于type的多态，一个命令处理多种不同类型的键

LLEN——基于编码的多态，一个命令处理多种不同类型的编码

### 回收

引用计数回收机制

每个对象的引用计数使用redisObject结构的refcount属性记录

初始值为1，当refcount = 0内存会被释放

### 对象共享

步骤

1. 数据库键值指向一个现有的值对象
2. 将被共享的值对象的引用计数增1

> Redis初始化服务器时，创建10_000个字符串对象，当服务器用值范围是[0, 9999]字符串对象时会使用这些共享对象

```C
127.0.0.1:6379> set b 333
OK
127.0.0.1:6379> object refcount b
(integer) 2
```

Redis为何不共享**包含**字符串对象的对象？

- redis共享一个对象前会检查key所期望的对象和现有对象是否完全相同，是才可以共享
- 上述检查是必要的，但值的组成关系越复杂则越耗时

### 对象空转时长

lru记录对象最后一次被命令程序访问的时间

```c
127.0.0.1:6379> object idletime e
(integer) 1482899
127.0.0.1:6379> object idletime e
(integer) 1482906
127.0.0.1:6379>
```

