# Java



## 并发、线程

锁 - 最简单的为斥锁，自旋锁





并发模型



并行worker模型

优点：1、容易理解；2、Worker -> Delegator -> Client 的过程是`异步`的。

缺点：1、共享状态变得复杂；2、无状态的worker；3、作业顺序不确定



流水线并发模型：也成为**响应式**、**事件驱动系统**

​		Actor模型：

​		Channel模型：



优点：1、不会存在共享状态；2、有状态worker3、更好的硬件整合4、使任务更加有效执行

缺点：流水线并发模型的缺点是任务会涉及多个 worker，因此可能会分散在项目代码的多个类中。产生回调地狱。



函数性并行模型：





# 计算机网络

## TCP

控制可靠、按序地传输以及端与端之间的流量控制和拥塞控制。

![img](https://mmbiz.qpic.cn/mmbiz_png/azicia1hOY6Q8OxKoZp2WX3FqUGWqPC3WotC8rJxducKgmovDgJBcDDgsGRf0WiabU0eTEDJmz9mRv47QYa23nvUQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

可以看到 TCP 包只有端口，没有 IP。

Seq 就是 Sequence Number 即序号，它是用来解决乱序问题的。

ACK 就是 Acknowledgement Numer 即确认号，它是用来解决丢包情况的，告诉发送方这个包我收到啦。

标志位就是 TCP flags 用来标记这个包是什么类型的，用来控制 TPC 的状态。

窗口就是滑动窗口，Sliding Window，用来流控。

### 三次握手

![img](https://mmbiz.qpic.cn/mmbiz_png/azicia1hOY6Q8OxKoZp2WX3FqUGWqPC3WoBAbxNgZSCpE74PyCgckpGyKLaFpUgrvpaAFyN4QRy8OSsHMHTa3VyQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

sjc：（1、c 发送自己的序列号给s；2、s确认c的序列号，发送确认消息同事给c发送s的序列号；3、c确认s的序列号并发消息）



**握手的重点就是同步初始序列号**





### 重传机制

常见的重传机制：

- 超时重传
- 快速重传
- SACK
- D-SACK



**超时重传**：发送数据时，设定一个定时器，当超过指定的时间后，没有收到对方的 `ACK` 确认应答报文，就会重发该数据。

TCP 会在以下两种情况发生超时重传：

- 数据包丢失
- 确认应答丢失

![img](https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeuicRMlA8rKvl5AVLibhibDhgObdlvOJianvD0oj586rMc8wVs4hlzUtgRibWfD0WBpAJhRtHxOPd9ibibQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



 `RTT`（Round-Trip Time 往返时延），是**数据从网络一端传送到另一端所需的时间**，也就是包的往返时间。

`RTO` （Retransmission Timeout 超时重传时间）

![img](https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeuicRMlA8rKvl5AVLibhibDhgyx80wY6Zsy2xW0U5egNib7icspUPsE9PKYq5WLaoJGpkTIoZ1c5gIjCw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**超时重传时间 RTO 的值应该略大于报文往返  RTT 的值**。

如果超时重发的数据，再次超时的时候，又需要重传的时候，TCP 的策略是**超时间隔加倍。**

也就是**每当遇到一次超时重传的时候，都会将下一次超时时间间隔设为先前值的两倍。两次超时，就说明网络环境差，不宜频繁反复发送。**



TCP 还有另外一种**快速重传（Fast Retransmit）机制**，它**不以时间为驱动，而是以数据驱动重传**。

![img](https://mmbiz.qpic.cn/mmbiz_png/J0g14CUwaZeuicRMlA8rKvl5AVLibhibDhggRhxrfiaRIsia0z3T4l7TxVM7J9vjFktFRKHkdva953UY6vFIpsYsOicQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)













# Redis

## 基础

1. **纯内存操作**：读取不需要进行磁盘 I/O，所以比传统数据库要快上不少；*(但不要有误区说磁盘就一定慢，例如 Kafka 就是使用磁盘顺序读取但仍然较快)*
2. **单线程，无锁竞争**：这保证了没有线程的上下文切换，不会因为多线程的一些操作而降低性能；
3. **多路 I/O 复用模型，非阻塞 I/O**：采用多路 I/O 复用技术可以让单个线程高效的处理多个网络连接请求（尽量减少网络 IO 的时间消耗）；
4. **高效的数据结构，加上底层做了大量优化**：Redis 对于底层的数据结构和内存占用做了大量的优化，例如不同长度的字符串使用不同的结构体表示，HyperLogLog 的密集型存储结构等等

redis不支持事务



### 锁

先拿**setnx**来争抢锁，抢到之后，再用**expire**给锁加一个过期时间防止锁忘记了释放。

https://zhuanlan.zhihu.com/p/248150523

### Pipeline

使用Pipeline执行速度比逐条执行要快，特别是客户端与服务端的网络延迟越大，性能体能越明显。

![未使用Pipeline执行N条命令](https://img-blog.csdnimg.cn/20181122105251930.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3cxbGd5,size_16,color_FFFFFF,t_70)

![使用了pipeline执行N条命令](https://img-blog.csdnimg.cn/20181122105343203.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3cxbGd5,size_16,color_FFFFFF,t_70)



### 线程模型

**Redis** 内部使用文件事件处理器 `file event handler`，这个文件事件处理器是单线程的，所以 **Redis** 才叫做单线程的模型。它采用 IO 多路复用机制同时监听多个 **Socket**，根据 **Socket** 上的事件来选择对应的事件处理器进行处理。

文件事件处理器的结构包含 4 个部分：

- 多个 **Socket**
- IO 多路复用程序
- 文件事件分派器
- 事件处理器（连接应答处理器、命令请求处理器、命令回复处理器）

多个 **Socket** 可能会并发产生不同的操作，每个操作对应不同的文件事件，但是 IO 多路复用程序会监听多个 **Socket**，会将 **Socket** 产生的事件放入队列中排队，事件分派器每次从队列中取出一个事件，把该事件交给对应的事件处理器进行处理。



### 过期策略

**Redis**的过期策略，是有**定期删除+惰性删除**两种。

定期删除：默认100ms随其抽取一些设置额过期时间的key，判断是否过期，是的话删除。

惰性删除：等查询了再删除。

如果定期删除和惰性删除都没有查到则使用内存淘汰机制：

**内存淘汰机制**

- **noeviction**:返回错误当内存限制达到并且客户端尝试执行会让更多内存被使用的命令（大部分的写入指令，但DEL和几个例外）

- **allkeys-lru**: 尝试回收最少使用的键（LRU），使得新添加的数据有空间存放。

- **volatile-lru**: 尝试回收最少使用的键（LRU），但仅限于在过期集合的键,使得新添加的数据有空间存放。

- **allkeys-random**: 回收随机的键使得新添加的数据有空间存放。

- **volatile-random**: 回收随机的键使得新添加的数据有空间存放，但仅限于在过期集合的键。

- **volatile-ttl**: 回收在过期集合的键，并且优先回收存活时间（TTL）较短的键,使得新添加的数据有空间存放。

  如果没有键满足回收的前提条件的话，策略**volatile-lru**, **volatile-random**以及**volatile-ttl**就和noeviction 差不多

## 数据结构

![img](https://mmbiz.qpic.cn/mmbiz_png/ia1kbU3RS1H7iacDCtsE2vxokt1kuyOCzFNJb8e6NgCp3gtEMhK1HXial8SQbpwbqeic95e6ibeutyx8SOibV5UrzD9g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



### SDS （*Simple Dynamic String* ）

数据结构

```c
struct sdshdr{
 int len;//当前长度
 int free; //未使用长度
 char buf[];//具体数据
}
```



与C字符串的不同

1、C语言为重头遍历，获取长度的时间复杂度为O（n），Redis则为O（1）；

2、杜绝缓冲区溢出（使用sdsMakeRoomFor来决定是否扩容）

```cpp
sds makeRoomFor(sds s, size_t addlen) {
    struct sdshdr *sh, *newsh;
    size_t free = sdsavail(s); //获取s目前的剩余空间
    if (free > addlen) return s; //无需扩展

    size_t len, newlen;
    len = sdslen(s);
    sh = (void*)(s-sizeof(struct sdshdr))):

    newlen = (len + addlen);
    //redis扩容规则 SDS_MAX_PREALLOC默认为1MB
    if  (newlen < SDS_MAX_PREALLOC)
        newlen *= 2;
    else 
        newlen += SDS_MAX_PERALLOC

    newsh = zrelloc(sh, sizeof(struct sdshdr) + newlen + 1);
    if  (news == NULL) return NULL;
    newsh->free = newlen - len;
    return newsh->buf;
}
```

3、减少修改字符串时带来的内存重分配次数

- 空间预分配：扩容过程中，如果需求长度小于1MB,则double长度，如果大于1MB，则需求长度+1MB.通过此策略，减少内存重分配次数。

```cpp
 //redis扩容规则
   if  (newlen < SDS_MAX_PREALLOC)
       newlen *= 2;
   else 
       newlen += SDS_MAX_PERALLOC
```

- 惰性空间释放：当修改并缩短sds字符串，sds并不回收多出来的字节，而是仅仅通过free属性来记录可以使用的空间。以sdstrim代码为例：

```rust
sds sdstrim(sds s, const char *cset) {
    struct sdshdr *sh = (void*)(s- sizeof(struct sdshdr));
    char *start, *end, *sp, *ep;
    sp = start = s;
    ep = end = s + sdslen(s) + 1;
    while(sp <= end && strchr(cset, *sp)) sp++;
    while(ep > start && strchr(cset, *ep)) ep--;
    len = (sp > ep) ? 0 : ((ep-sp)+1); //计算trim后的字符串长度
    if  (sh->buf != sp) memmove(sh->buf, sp, len);
    sp->buf[len] = '\0';
    sh->free = sh->free + (sh->len -len);
    sh->len = len;
    return s;
}
```

4、二进制安全

由于Redis保存了len长度，所以可以直接判断长度，而不是边路读取判断空字符串。这样就可以存储二进制文件了，而不担心空字符串问题。

### 列表List

相当于Java的LinkedList，通过 `prev` 和 `next` 指针组成双向链表：

```rust
/* Node, List, and Iterator are the only data structures used currently. */

typedefstruct listNode {
    struct listNode *prev;
    struct listNode *next;
    void *value;
} listNode;

typedefstruct listIter {
    listNode *next;
    int direction;
} listIter;

typedefstruct list {
    listNode *head;
    listNode *tail;
    void *(*dup)(void *ptr);
    void (*free)(void *ptr);
    int (*match)(void *ptr, void *key);
    unsignedlong len;
} list;
```

![img](https://mmbiz.qpic.cn/mmbiz_png/ia1kbU3RS1H5tIHy6ebDys4qIicGZ07dLVMSR6DRWHhHDX27DLPia0CicIJ3xHcZwZUsz0VvATLY77xIItAUULS5icw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/ia1kbU3RS1H5tIHy6ebDys4qIicGZ07dLVgH9lk2mm8gBp2oDLiaacDiaJT88pO28m9CRJbyWADibAJjIFzgCRNuibpg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)









### Hash 

**Redis** 中的字典相当于 Java 中的 **HashMap**，内部实现也差不多类似，都是通过 **"数组 + 链表"** 的 **链地址法** 来解决部分 哈希冲突，同时这样的结构也吸收了两种不同数据结构的优点。



```cpp
typedefstruct dictht {
    // 哈希表数组
    dictEntry **table;
    // 哈希表大小
    unsignedlong size;
    // 哈希表大小掩码，用于计算索引值，总是等于 size - 1
    unsignedlong sizemask;
    // 该哈希表已有节点的数量
    unsignedlong used;
} dictht;

typedefstruct dict {
    dictType *type;
    void *privdata;
    // 内部有两个 dictht 结构
    dictht ht[2];
    long rehashidx; /* rehashing not in progress if rehashidx == -1 */
    unsignedlong iterators; /* number of iterators currently running */
} dict;
```

`table` 属性是一个数组，数组中的每个元素都是一个指向 `dict.h/dictEntry` 结构的指针，而每个 `dictEntry` 结构保存着一个键值对：

```cpp
typedefstruct dictEntry {
    // 键
    void *key;
    // 值
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
        double d;
    } v;
    // 指向下个哈希表节点，形成链表
    struct dictEntry *next;
} dictEntry;
```



字典结构内部包含 **两个 hashtable**，通常情况下只有一个 `hashtable` 有值，但是在字典扩容缩容时，需要分配新的 `hashtable`，然后进行 **渐进式搬迁** *(rehash)*，这时候两个 `hashtable` 分别存储旧的和新的 `hashtable`，待搬迁结束后，旧的将被删除，新的 `hashtable` 取而代之。

正常情况下，当 hash 表中 **元素的个数等于第一维数组的长度时**，就会开始扩容，扩容的新数组是 **原数组大小的 2 倍**。不过如果 Redis 正在做 `bgsave(持久化命令)`，为了减少内存也得过多分离，Redis 尽量不去扩容，但是如果 hash 表非常满了，**达到了第一维数组长度的 5 倍了**，这个时候就会 **强制扩容**。

当 hash 表因为元素逐渐被删除变得越来越稀疏时，Redis 会对 hash 表进行缩容来减少 hash 表的第一维数组空间占用。所用的条件是 **元素个数低于数组长度的 10%**，缩容不会考虑 Redis 是否在做 `bgsave`。

![img](https://mmbiz.qpic.cn/mmbiz_png/ia1kbU3RS1H5tIHy6ebDys4qIicGZ07dLVzL3lGyrSnjpfThSUnBTibic9O1HoWRXwtLq3FfJSwqhu8UDib74VRMyXw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

渐进式 rehash 会在 rehash 的同时，保留新旧两个 hash 结构，如上图所示，查询时会同时查询两个 hash 结构，然后在后续的定时任务以及 hash 操作指令中，循序渐进的把旧字典的内容迁移到新字典中。当搬迁完成了，就会使用新的 hash 结构取而代之。

### Set

相当于 Java 语言中的 **HashSet**，它内部的键值对是无序、唯一的。它的内部实现相当于一个特殊的字典，字典中所有的 value 都是一个值 NULL。

### Zset

似于 Java 中 **SortedSet** 和 **HashMap** 的结合体，一方面它是一个 set，保证了内部 value 的唯一性，另一方面它可以为每个 value 赋予一个 score 值，用来代表排序的权重。基于跳跃表实现

#### 跳跃表

![img](https://mmbiz.qpic.cn/mmbiz_png/ia1kbU3RS1H5ZLiaicqeR9mzkQuQLwvtFfQXCslkXHAWsN5hZK4GDvYxiaWxZQeS2TaEdaIIFMlYKYTcnFlsMOG6gQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

为什么使用跳跃表而不是红黑树

1. **性能考虑：** 在高并发的情况下，树形结构需要执行一些类似于 rebalance 这样的可能涉及整棵树的操作，相对来说跳跃表的变化只涉及局部 *(下面详细说)*；
2. **实现考虑：** 在复杂度与红黑树相同的情况下，跳跃表实现起来更简单，看起来也更加直观；



![img](https://mmbiz.qpic.cn/mmbiz_png/ia1kbU3RS1H5ZLiaicqeR9mzkQuQLwvtFfQ5qUqf8c0vC3bfbc710Tz6iadcOlDYb39pApOUP9pCaUDQtuicUn9Jibvg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![img](https://mmbiz.qpic.cn/mmbiz_png/ia1kbU3RS1H5ZLiaicqeR9mzkQuQLwvtFfQclc3x7f7KuQtUMpfn1rP3I7mGVuOyydQjyhujAgTzo9z8XiacD0oJmA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



Redis 中的跳跃表由 `server.h/zskiplistNode` 和 `server.h/zskiplist` 两个结构定义，前者为跳跃表节点，后者则保存了跳跃节点的相关信息，同之前的 `集合 list` 结构类似，其实只有 `zskiplistNode` 就可以实现了，但是引入后者是为了更加方便的操作：

```
/* ZSETs use a specialized version of Skiplists */
typedefstruct zskiplistNode {
    // value
    sds ele;
    // 分值
    double score;
    // 后退指针
    struct zskiplistNode *backward;
    // 层
    struct zskiplistLevel {
        // 前进指针
        struct zskiplistNode *forward;
        // 跨度
        unsignedlong span;
    } level[];
} zskiplistNode;

typedefstruct zskiplist {
    // 跳跃表头指针
    struct zskiplistNode *header, *tail;
    // 表中节点的数量
    unsignedlong length;
    // 表中层数最大的节点的层数
    int level;
} zskiplist;
```

对于每一个新插入的节点，都需要调用一个随机算法给它分配一个合理的层数，源码在 `t_zset.c/zslRandomLevel(void)` 中被定义：

```
int zslRandomLevel(void) {
    int level = 1;
    while ((random()&0xFFFF) < (ZSKIPLIST_P * 0xFFFF))
        level += 1;
    return (level<ZSKIPLIST_MAXLEVEL) ? level : ZSKIPLIST_MAXLEVEL;
}
```

直观上期望的目标是 50% 的概率被分配到 `Level 1`，25% 的概率被分配到 `Level 2`，12.5% 的概率被分配到 `Level 3`，以此类推...有 2-63 的概率被分配到最顶层，因为这里每一层的晋升率都是 50%。

**Redis 跳跃表默认允许最大的层数是 32**，被源码中 `ZSKIPLIST_MAXLEVEL` 定义，当 `Level[0]` 有 264 个元素时，才能达到 32 层，所以定义 32 完全够用了。

搜索当前节点插入位置

```
serverAssert(!isnan(score));
x = zsl->header;
// 逐步降级寻找目标节点，得到 "搜索路径"
for (i = zsl->level-1; i >= 0; i--) {
    /* store rank that is crossed to reach the insert position */
    rank[i] = i == (zsl->level-1) ? 0 : rank[i+1];
    // 如果 score 相等，还需要比较 value 值
    while (x->level[i].forward &&
            (x->level[i].forward->score < score ||
                (x->level[i].forward->score == score &&
                sdscmp(x->level[i].forward->ele,ele) < 0)))
    {
        rank[i] += x->level[i].span;
        x = x->level[i].forward;
    }
    // 记录 "搜索路径"
    update[i] = x;
}
```

有一种极端的情况，就是跳跃表中的所有 score 值都是一样，zset 的查找性能会不会退化为 O(n) 呢？

从上面的源码中我们可以发现 zset 的排序元素不只是看 score 值，也会比较 value 值 *（字符串比较）*



### Bloom Filter 



### **HyperLogLog** 



### zipList压缩列表 

这是 Redis **为了节约内存** 而使用的一种数据结构，**zset** 和 **hash** 容器对象会在元素个数较少的时候，采用压缩列表（ziplist）进行存储。压缩列表是 **一块连续的内存空间**，元素之间紧挨着存储，没有任何冗余空隙。

当元素个数较少时，Redis 用 ziplist 来存储数据，当元素个数超过某个值时，链表键中会把 ziplist 转化为 linkedlist，字典键中会把 ziplist 转化为 hashtable。

在3.2之后，ziplist被quicklist替代。但是仍然是zset底层实现之一。



### quicklist快速列表

https://zhuanlan.zhihu.com/p/102422311

本质上来说，quicklist里面保存着一个一个小的ziplist。结构如下：

![preview](https://pic4.zhimg.com/v2-800dbf77bba29897de1ad769d0149f8f_r.jpg)





## 持久化



### RDB-快照

快照是最简单的持久性模式。每个快照生成一个`.rdb`文件

原理：fork和cow。fork是指redis通过创建子进程来进行RDB操作，cow指的是**copy on write**，子进程创建后，父子进程共享数据段，父进程继续提供读写服务，写脏的页面数据会逐渐和子进程分离开来。

优点:

1. 只有一个文件 `dump.rdb`，**方便持久化**。
2. **容灾性好**，一个文件可以保存到安全的磁盘。
3. **性能最大化**，`fork` 子进程来完成写操作，让主进程继续处理命令，所以使 IO 最大化。使用单独子进程来进行持久化，主进程不会进行任何 IO 操作，保证了 Redis 的高性能。
4. 相对于数据集大时，比 AOF 的 **启动效率** 更高。

缺点：

1. **数据安全性低**。RDB 是间隔一段时间（默认五分钟）进行持久化，如果持久化之间 Redis 发生故障，会发生数据丢失。所以这种方式更适合数据要求不严谨的时候；
2. 

### AOF（Append only file--仅追加文件）

每次执行 **修改内存** 中数据集的写操作时，都会 **记录** 该操作。

优点：

1. **数据安全**，aof 持久化可以配置 `appendfsync` 属性，有 `always`，每进行一次命令操作就记录到 aof 文件中一次。
2. 通过 append 模式写文件，即使中途服务器宕机，可以通过 redis-check-aof 工具解决数据一致性问题。
3. AOF 机制的 rewrite 模式。AOF 文件没被 rewrite 之前（文件过大时会对命令 进行合并重写），可以删除其中的某些命令（比如误操作的 flushall）

缺点：

1. AOF 文件比 RDB **文件大**，且 **恢复速度慢**。
2. **数据集大** 的时候，比 rdb **启动效率低**。



**RDB**更适合做**冷备**，**AOF**更适合做**热备**





### 数据恢复策略

**Redis** 的数据恢复有着如下的优先级：

1. 如果只配置 AOF ，重启时加载 AOF 文件恢复数据；
2. 如果同时配置了 RDB 和 AOF ，启动只加载 AOF 文件恢复数据；
3. 如果只配置 RDB，启动将加载 dump 文件恢复数据。







## 集群



### 主从复制

**主从复制**，是指将一台 Redis 服务器的数据，复制到其他的 Redis 服务器。前者称为 **主节点(master)**，后者称为 **从节点(slave)**。且数据的复制是 **单向** 的，只能由主节点到从节点。Redis 主从复制支持 **主从同步** 和 **从从同步** 两种，后者是 Redis 后续版本新增的功能，以减轻主节点的同步负担。

#### 基础

##### 作用:

- **数据冗余：** 主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式。
- **故障恢复：** 当主节点出现问题时，可以由从节点提供服务，实现快速的故障恢复 *(实际上是一种服务的冗余)*。
- **负载均衡：** 在主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务 *（即写 Redis 数据时应用连接主节点，读 Redis 数据时应用连接从节点）*，分担服务器负载。尤其是在写少读多的场景下，通过多个从节点分担读负载，可以大大提高 Redis 服务器的并发量。
- **高可用基石：** 除了上述作用以外，主从复制还是哨兵和集群能够实施的 **基础**，因此说主从复制是 Redis 高可用的基础。



##### 实现原理：

**简化成三个阶段：准备阶段-数据同步阶段-命令传播阶段**。

<img src="https://mmbiz.qpic.cn/mmbiz_png/ia1kbU3RS1H7iacDCtsE2vxokt1kuyOCzFNX5lppTictyXka2AzEmCQ4dnhzUJl3oxicmZK38IbMBgfcTA8ZvAH72A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1" alt="img" style="zoom:200%;" />

你启动一台slave 的时候，他会发送一个**psync**命令给master ，如果是这个slave第一次连接到master，他会触发一个全量复制（主节点做**bgsave**）。master就会启动一个线程，生成**RDB**快照，还会把新的写请求都缓存在内存中，**RDB**文件生成后，master会将这个**RDB**发送给slave的，slave拿到之后做的第一件事情就是写进本地的磁盘，然后加载进内存，然后master会把内存里面缓存的那些通过AOF日志都发给slave。



#### 哨兵

![img](https://mmbiz.qpic.cn/mmbiz_png/ia1kbU3RS1H7iacDCtsE2vxokt1kuyOCzFffJ6s9BvGqNTsO6AercmUstX26L4wJJnZsERx7xUUgD27YP5jxdhxg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

- **哨兵节点：** 哨兵系统由一个或多个哨兵节点组成，哨兵节点是特殊的 Redis 节点，不存储数据；
- **数据节点：** 主节点和从节点都是数据节点；

##### 哨兵的功能

- **监控（Monitoring）：** 哨兵会不断地检查主节点和从节点是否运作正常。
- **自动故障转移（Automatic failover）：** 当 **主节点** 不能正常工作时，哨兵会开始 **自动故障转移操作**，它会将失效主节点的其中一个 **从节点升级为新的主节点**，并让其他从节点改为复制新的主节点。
- **配置提供者（Configuration provider）：** 客户端在初始化时，通过连接哨兵来获得当前 Redis 服务的主节点地址。
- **通知（Notification）：** 哨兵可以将故障转移的结果发送给客户端。

其中，监控和自动故障转移功能，使得哨兵可以及时发现主节点故障并完成转移。而配置提供者和通知功能，则需要在与客户端的交互中才能体现。



##### 如何挑选新的主服务器

**故障转移操作的第一步** 要做的就是在已下线主服务器属下的所有从服务器中，挑选出一个状态良好、数据完整的从服务器，然后向这个从服务器发送 `slaveof no one` 命令，将这个从服务器转换为主服务器。

挑选规则：

1. 在失效主服务器属下的从服务器当中， 那些被标记为主观下线、已断线、或者最后一次回复 PING 命令的时间大于五秒钟的从服务器都会被 **淘汰**。
2. 在失效主服务器属下的从服务器当中， 那些与失效主服务器连接断开的时长超过 down-after 选项指定的时长十倍的从服务器都会被 **淘汰**。
3. 在 **经历了以上两轮淘汰之后** 剩下来的从服务器中， 我们选出 **复制偏移量（replication offset）最大** 的那个 **从服务器** 作为新的主服务器；如果复制偏移量不可用，或者从服务器的复制偏移量相同，那么 **带有最小运行 ID** 的那个从服务器成为新的主服务器。



### **Redis Cluster**

#### 原理

![img](https://mmbiz.qpic.cn/mmbiz_png/ia1kbU3RS1H7iacDCtsE2vxokt1kuyOCzFSGHNepicHbGiaBibe9vp3viaBrDtYheKUTydYplUY8X8nKqA6sghiaicHnuw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

Redis 集群中内置了 `16384` 个哈希槽。当客户端连接到 Redis 集群之后，会同时得到一份关于这个 **集群的配置信息**，当客户端具体对某一个 `key` 值进行操作时，会计算出它的一个 Hash 值，然后把结果对 `16384` **求余数**，这样每个 `key` 都会对应一个编号在 `0-16383` 之间的哈希槽，Redis 会根据节点数量 **大致均等** 的将哈希槽映射到不同的节点。

再结合集群的配置信息就能够知道这个 `key` 值应该存储在哪一个具体的 Redis 节点中，如果不属于自己管，那么就会使用一个特殊的 `MOVED` 命令来进行一个跳转，告诉客户端去连接这个节点以获取数据：

```
GET x
-MOVED 3999 127.0.0.1:6381
```

`MOVED` 指令第一个参数 `3999` 是 `key` 对应的槽位编号，后面是目标节点地址，`MOVED` 命令前面有一个减号，表示这是一个错误的消息。客户端在收到 `MOVED` 指令后，就立即纠正本地的 **槽位映射表**，那么下一次再访问 `key` 时就能够到正确的地方去获取了。

#### 作用

1. **数据分区：** 数据分区 *(或称数据分片)* 是集群最核心的功能。集群将数据分散到多个节点，**一方面** 突破了 Redis 单机内存大小的限制，**存储容量大大增加**；**另一方面** 每个主节点都可以对外提供读服务和写服务，**大大提高了集群的响应能力**。Redis 单机内存大小受限问题，在介绍持久化和主从复制时都有提及，例如，如果单机内存太大，`bgsave` 和 `bgrewriteaof` 的 `fork` 操作可能导致主进程阻塞，主从环境下主机切换时可能导致从节点长时间无法提供服务，全量复制阶段主节点的复制缓冲区可能溢出……
2. **高可用：** 集群支持主从复制和主节点的 **自动故障转移** *（与哨兵类似）*，当任一节点发生故障时，集群仍然可以对外提供服务。

#### 分区策略

##### 一致性哈希算法

一致性哈希算法将 **整个哈希值空间** 组织成一个虚拟的圆环，范围是 *[0 - 232 - 1]*，对于每一个数据，根据 `key` 计算 hash 值，确数据在环上的位置，然后从此位置沿顺时针行走，找到的第一台服务器就是其应该映射到的服务器：

![img](https://mmbiz.qpic.cn/mmbiz_png/ia1kbU3RS1H7iacDCtsE2vxokt1kuyOCzFeTRLvicdbHE4XNW1QHzTSEq9dbqu08YpJ59Hu1J0OICjy1Rb766vicFA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

与哈希取余分区相比，一致性哈希分区将 **增减节点的影响限制在相邻节点**。以上图为例，如果在 `node1` 和 `node2` 之间增加 `node5`，则只有 `node2` 中的一部分数据会迁移到 `node5`；如果去掉 `node2`，则原 `node2` 中的数据只会迁移到 `node4` 中，只有 `node4` 会受影响。

一致性哈希分区的主要问题在于，当 **节点数量较少** 时，增加或删减节点，**对单个节点的影响可能很大**，造成数据的严重不平衡。还是以上图为例，如果去掉 `node2`，`node4` 中的数据由总数据的 `1/4` 左右变为 `1/2` 左右，与其他节点相比负载过高。



##### **带有虚拟节点的一致性哈希分区**



该方案在 **一致性哈希分区的基础上**，引入了 **虚拟节点** 的概念。Redis 集群使用的便是该方案，其中的虚拟节点称为 **槽（slot）**。槽是介于数据和实际节点之间的虚拟概念，每个实际节点包含一定数量的槽，每个槽包含哈希值在一定范围内的数据。

在使用了槽的一致性哈希分区中，**槽是数据管理和迁移的基本单位**。槽 **解耦** 了 **数据和实际节点** 之间的关系，增加或删除节点对系统的影响很小。仍以上图为例，系统中有 `4` 个实际节点，假设为其分配 `16` 个槽(0-15)；

- 槽 0-3 位于 node1；4-7 位于 node2；以此类推....

如果此时删除 `node2`，只需要将槽 4-7 重新分配即可，例如槽 4-5 分配给 `node1`，槽 6 分配给 `node3`，槽 7 分配给 `node4`；可以看出删除 `node2` 后，数据在其他节点的分布仍然较为均衡。



### 节点之间的通信

在 **哨兵系统** 中，节点分为 **数据节点** 和 **哨兵节点**：前者存储数据，后者实现额外的控制功能。在 **集群** 中，没有数据节点与非数据节点之分：**所有的节点都存储数据，也都参与集群状态的维护**。为此，集群中的每个节点，都提供了两个 TCP 端口：

- **普通端口：** 即我们在前面指定的端口 *(7000等)*。普通端口主要用于为客户端提供服务 *（与单机节点类似）*；但在节点间数据迁移时也会使用。
- **集群端口：** 端口号是普通端口 + 10000 *（10000是固定值，无法改变）*，如 `7000` 节点的集群端口为 `17000`。**集群端口只用于节点之间的通信**，如搭建集群、增减节点、故障转移等操作时节点间的通信；不要使用客户端连接集群接口。为了保证集群可以正常工作，在配置防火墙时，要同时开启普通端口和集群端口。

#### Gossip 协议

节点间通信，按照通信协议可以分为几种类型：单对单、广播、Gossip 协议等。重点是广播和 Gossip 的对比。

- 广播是指向集群内所有节点发送消息。**优点** 是集群的收敛速度快(集群收敛是指集群内所有节点获得的集群信息是一致的)，**缺点** 是每条消息都要发送给所有节点，CPU、带宽等消耗较大。
- Gossip 协议的特点是：在节点数量有限的网络中，**每个节点都 “随机” 的与部分节点通信** *（并不是真正的随机，而是根据特定的规则选择通信的节点）*，经过一番杂乱无章的通信，每个节点的状态很快会达到一致。Gossip 协议的 **优点** 有负载 *(比广播)* 低、去中心化、容错性高 *(因为通信有冗余)* 等；**缺点** 主要是集群的收敛速度慢。

#### 消息类型

集群中的节点采用 **固定频率（每秒10次）** 的 **定时任务** 进行通信相关的工作：判断是否需要发送消息及消息类型、确定接收节点、发送消息等。如果集群状态发生了变化，如增减节点、槽状态变更，通过节点间的通信，所有节点会很快得知整个集群的状态，使集群收敛。

节点间发送的消息主要分为 `5` 种：`meet 消息`、`ping 消息`、`pong 消息`、`fail 消息`、`publish 消息`。不同的消息类型，通信协议、发送的频率和时机、接收节点的选择等是不同的：

- **MEET 消息：** 在节点握手阶段，当节点收到客户端的 `CLUSTER MEET` 命令时，会向新加入的节点发送 `MEET` 消息，请求新节点加入到当前集群；新节点收到 MEET 消息后会回复一个 `PONG` 消息。
- **PING 消息：** 集群里每个节点每秒钟会选择部分节点发送 `PING` 消息，接收者收到消息后会回复一个 `PONG` 消息。**PING 消息的内容是自身节点和部分其他节点的状态信息**，作用是彼此交换信息，以及检测节点是否在线。`PING` 消息使用 Gossip 协议发送，接收节点的选择兼顾了收敛速度和带宽成本，**具体规则如下**：(1)随机找 5 个节点，在其中选择最久没有通信的 1 个节点；(2)扫描节点列表，选择最近一次收到 `PONG` 消息时间大于 `cluster_node_timeout / 2` 的所有节点，防止这些节点长时间未更新。
- **PONG消息：** `PONG` 消息封装了自身状态数据。可以分为两种：**第一种** 是在接到 `MEET/PING` 消息后回复的 `PONG` 消息；**第二种** 是指节点向集群广播 `PONG` 消息，这样其他节点可以获知该节点的最新信息，例如故障恢复后新的主节点会广播 `PONG` 消息。
- **FAIL 消息：** 当一个主节点判断另一个主节点进入 `FAIL` 状态时，会向集群广播这一 `FAIL` 消息；接收节点会将这一 `FAIL` 消息保存起来，便于后续的判断。
- **PUBLISH 消息：** 节点收到 `PUBLISH` 命令后，会先执行该命令，然后向集群广播这一消息，接收节点也会执行该 `PUBLISH` 命令。



## 应用

1. 缓存雪崩问题；（加随机数）
2. 缓存穿透问题；（1、空对象也放到缓存 2、加布隆过滤器）
3. 缓存和数据库双写一致性问题（消息队列的重试机制）
4. 判断热key（hotkeys命令）
5. 性能解决方案
   1. Master 最好不要做任何持久化工作，包括内存快照和 AOF 日志文件，特别是不要启用内存快照做持久化。
   2. 如果数据比较关键，某个 Slave 开启 AOF 备份数据，策略为每秒同步一次。
   3. 为了主从复制的速度和连接的稳定性，Slave 和 Master 最好在同一个局域网内。
   4. 尽量避免在压力较大的主库上增加从库。
   5. Master 调用 BGREWRITEAOF 重写 AOF 文件，AOF 在重写的时候会占大量的 CPU 和内存资源，导致服务 load 过高，出现短暂服务暂停现象。
   6. 为了 Master 的稳定性，主从复制不要用图状结构，用单向链表结构更稳定，即主从关系为：Master<–Slave1<–Slave2<–Slave3…，这样的结构也方便解决单点故障问题，实现 Slave 对 Master 的替换，也即，如果 Master 挂了，可以立马启用 Slave1 做 Master，其他不变。

如何找出指定key

使用 `keys` 指令可以扫出指定模式的 key 列表。但是要注意 keys 指令会导致线程阻塞一段时间，线上服务会停顿，直到指令执行完毕，服务才能恢复。这个时候可以使用 `scan` 指令，`scan` 指令可以无阻塞的提取出指定模式的 `key` 列表，但是会有一定的重复概率，在客户端做一次去重就可以了，但是整体所花费的时间会比直接用 `keys` 指令长。





# MySQL



## 基础



数据库基础组件





![img](https://mmbiz.qpic.cn/mmbiz_jpg/uChmeeX1Fpz8Sib2PsicNMxibibNic5p0JicUdsIkRX56rPzJ9S1lj1svQZHcwxH43DYuJRJk0uD8vmS7pibTqqa62Zuw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



InnoDB 采用 MVCC 来支持高并发，并且实现了四个标准隔离级别(未提交读、提交读、可重复读、可串行化)。其默认级别时可重复读（REPEATABLE READ），在可重复读级别下，通过 MVCC + Next-Key Locking 防止幻读。

主索引是聚簇索引，在索引中保存了数据，从而避免直接读取磁盘，因此对主键查询有很高的性能。

InnoDB 内部做了很多优化，包括从磁盘读取数据时采用的可预测性读，能够自动在内存中创建 hash 索引以加速读操作的自适应哈希索引，以及能够加速插入操作的插入缓冲区等。

InnoDB 支持真正的在线热备份，MySQL 其他的存储引擎不支持在线热备份，要获取一致性视图需要停止对所有表的写入，而在读写混合的场景中，停止写入可能也意味着停止读取。

innodb架构



![img](https://mmbiz.qpic.cn/mmbiz_png/lA1CtgibZZmyiaicGXrxl6QZicE8Ms4ny8M2PRyKZKibWGlfBDde8MVjy4FyMb7QQY9MZOtTu405hwsMiaPC9SpWy3Mg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**1、Buffer Pool**

> The buffer pool is an area in main memory where InnoDB caches table and index data as it is accessed.

正如之前提到的，MySQL 不会直接去修改磁盘的数据，因为这样做太慢了，MySQL 会先改内存，然后记录 redo log，等有空了再刷磁盘，如果内存里没有数据，就去磁盘 load。

而这些数据存放的地方，就是 Buffer Pool。



MySQL 是以「页」（page）为单位从磁盘读取数据的，Buffer Pool 里的数据也是如此，实际上，Buffer Pool 是`a linked list of pages`，一个以页为元素的链表。

Buffer Pool 采用基于 LRU（least recently used） 的算法来管理内存：

![img](https://mmbiz.qpic.cn/mmbiz_png/lA1CtgibZZmyiaicGXrxl6QZicE8Ms4ny8M2hYhdUAiaBnGljDySByIwrqCEJa7cEuHJtsQTx0B18mvcv16yF37H2yQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



**2、Change Buffer**

上面提到过，如果内存里没有对应「页」的数据，MySQL 就会去把数据从磁盘里 load 出来，如果每次需要的「页」都不同，或者不是相邻的「页」，那么每次 MySQL 都要去 load，这样就很慢了。

于是如果 MySQL 发现你要修改的页，不在内存里，就把你要对页的修改，先记到一个叫 Change Buffer 的地方，同时记录 redo log，然后再慢慢把数据 load 到内存，load 过来后，再把 Change Buffer 里记录的修改，应用到内存（Buffer Pool）中，这个动作叫做 **merge**；而把内存数据刷到磁盘的动作，叫 **purge**：

- **merge：Change Buffer -> Buffer Pool**
- **purge：Buffer Pool -> Disk**

Change Buffer 只在操作「二级索引」（secondary index）时才使用，原因是「聚簇索引」（clustered indexes）必须是「唯一」的，也就意味着每次插入、更新，都需要检查是否已经有相同的字段存在，也就没有必要使用 Change Buffer 了；另外，「聚簇索引」操作的随机性比较小，通常在相邻的「页」进行操作，比如使用了自增主键的「聚簇索引」，那么 insert 时就是递增、有序的，不像「二级索引」，访问非常随机。

**3、Adaptive Hash Index**

MySQL 索引，不管是在磁盘里，还是被 load 到内存后，都是 B+ 树，B+ 树的查找次数取决于树的深度。你看，数据都已经放到内存了，还不能“一下子”就找到它，还要“几下子”，这空间牺牲的是不是不太值得？

尤其是那些频繁被访问的数据，每次过来都要走 B+ 树来查询，这时就会想到，我用一个指针把数据的位置记录下来不就好了？

这就是「自适应哈希索引」（Adaptive Hash Index）。自适应，顾名思义，MySQL 会自动评估使用自适应索引是否值得，如果观察到建立哈希索引可以提升速度，则建立。

**4、Log Buffer**

> The log buffer is the memory area that holds data to be written to the log files on disk.

从上面架构图可以看到，Log Buffer 里的 redo log，会被刷到磁盘里：

![img](https://mmbiz.qpic.cn/mmbiz_png/lA1CtgibZZmyiaicGXrxl6QZicE8Ms4ny8M2uH048BxxvGmG3dzpU6Cibwhg2auN7sy8vrmFSqcheed7ZZ7trqcZbzA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



### 磁盘架构

磁盘里有什么呢？除了表结构定义和索引，还有一些为了高性能和高可靠而设计的角色，比如 redo log、undo log、Change Buffer，以及 Doublewrite Buffer 等等.

**1、表空间（Tablespaces）**

从架构图可以看到，Tablespaces 分为五种：

- The System Tablespace
- File-Per-Table Tablespaces
- General Tablespace
- Undo Tablespaces
- Temporary Tablespaces

其中，我们平时创建的表的数据，可以存放到 The System Tablespace 、File-Per-Table Tablespaces、General Tablespace 三者中的任意一个地方，具体取决于你的配置和创建表时的 sql 语句。

**2、Doublewrite Buffer**

**如果说 Change Buffer 是提升性能，那么 Doublewrite Buffer 就是保证数据页的可靠性。**

怎么理解呢？

前面提到过，MySQL 以「页」为读取和写入单位，一个「页」里面有多行数据，写入数据时，MySQL 会先写内存中的页，然后再刷新到磁盘中的页。

这时问题来了，假设在某一次从内存刷新到磁盘的过程中，一个「页」刷了一半，突然操作系统或者 MySQL 进程奔溃了，这时候，内存里的页数据被清除了，而磁盘里的页数据，刷了一半，处于一个中间状态，不尴不尬，可以说是一个「不完整」，甚至是「坏掉的」的页。

有同学说，不是有 Redo Log 么？其实这个时候 Redo Log 也已经无力回天，Redo Log 是要在磁盘中的页数据是正常的、没有损坏的情况下，才能把磁盘里页数据 load 到内存，然后应用 Redo Log。而如果磁盘中的页数据已经损坏，是无法应用 Redo Log 的。

所以，MySQL 在刷数据到磁盘之前，要先把数据写到另外一个地方，也就是 Doublewrite Buffer，写完后，再开始写磁盘。Doublewrite Buffer 可以理解为是一个备份（recovery），万一真的发生 crash，就可以利用 Doublewrite Buffer 来修复磁盘里的数据。



## log日志







### 事务

### ACID

事务最基本的莫过于 ACID 四个特性了，这四个特性分别是：

- Atomicity：原子性
- Consistency：一致性
- Isolation：隔离性
- Durability：持久性

**原子性**

事务被视为不可分割的最小单元，事务的所有操作要么全部成功，要么全部失败回滚。

**一致性**

数据库在事务执行前后都保持一致性状态，在一致性状态下，所有事务对一个数据的读取结果都是相同的。

**隔离性**

一个事务所做的修改在最终提交以前，对其他事务是不可见的。

**持久性**

一旦事务提交，则其所做的修改将会永远保存到数据库中。即使系统发生崩溃，事务执行的结果也不能丢。



隔离级别

**未提交读（READ UNCOMMITTED）**

事务中的修改，即使没有提交，对其他事务也是可见的。

**提交读（READ COMMITTED）**

一个事务只能读取已经提交的事务所做的修改。换句话说，一个事务所做的修改在提交之前对其他事务是不可见的。

**可重复读（REPEATABLE READ）**

保证在同一个事务中多次读取同样数据的结果是一样的。

**可串行化（SERIALIZABLE）**

强制事务串行执行。

需要加锁实现，而其它隔离级别通常不需要。

| 隔离级别 | 脏读 | 不可重复读 | 幻影读 |
| :------: | :--: | :--------: | :----: |
| 未提交读 |  √   |     √      |   √    |
|  提交读  |  ×   |     √      |   √    |
| 可重复读 |  ×   |     ×      |   √    |
| 可串行化 |  ×   |     ×      |   ×    |



### 锁

**共享锁（S Lock）**

允许事务读一行数据

**排他锁（X Lock）**

允许事务删除或者更新一行数据

**意向共享锁（IS Lock）**

事务想要获得一张表中某几行的共享锁

**意向排他锁**

事务想要获得一张表中某几行的排他锁





#### 锁算法

Record Lock

锁定一个记录上的索引，而不是记录本身。

如果表没有设置索引，InnoDB 会自动在主键上创建隐藏的聚簇索引，因此 Record Locks 依然可以使用。



Gap Lock

锁定索引之间的间隙，但是不包含索引本身。例如当一个事务执行以下语句，其它事务就不能在 t.c 中插入 15。

```
SELECT c FROM t WHERE c BETWEEN 10 and 20 FOR UPDATE;
```



Next-Key Lock

它是 Record Locks 和 Gap Locks 的结合，不仅锁定一个记录上的索引，也锁定索引之间的间隙。例如一个索引包含以下值：10, 11, 13, and 20，那么就需要锁定以下区间：

```
(-∞, 10]
(10, 11]
(11, 13]
(13, 20]
(20, +∞)
```

> **在 InnoDB 存储引擎中，SELECT 操作的不可重复读问题通过 MVCC 得到了解决，而 UPDATE、DELETE 的不可重复读问题通过 Record Lock 解决，INSERT 的不可重复读问题是通过 Next-Key Lock（Record Lock + Gap Lock）解决的。**

从上面第二张图可以看到，InnoDB 主要分为两大块：

- InnoDB In-Memory Structures
- InnoDB On-Disk Structures

内存和磁盘，让我们先从内存开始。

**1、Buffer Pool**

> The buffer pool is an area in main memory where InnoDB caches table and index data as it is accessed.

正如之前提到的，MySQL 不会直接去修改磁盘的数据，因为这样做太慢了，MySQL 会先改内存，然后记录 redo log，等有空了再刷磁盘，如果内存里没有数据，就去磁盘 load。

而这些数据存放的地方，就是 Buffer Pool。

MySQL 是以「页」（page）为单位从磁盘读取数据的，Buffer Pool 里的数据也是如此，实际上，Buffer Pool 是`a linked list of pages`，一个以页为元素的链表。

Buffer Pool 采用基于 LRU（least recently used） 的算法来管理内存：

![img](https://mmbiz.qpic.cn/mmbiz_png/lA1CtgibZZmyiaicGXrxl6QZicE8Ms4ny8M2hYhdUAiaBnGljDySByIwrqCEJa7cEuHJtsQTx0B18mvcv16yF37H2yQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)





### MVCC

多版本并发控制（Multi-Version Concurrency Control, MVCC）是 MySQL 的 InnoDB 存储引擎实现隔离级别的一种具体方式，用于实现提交读和可重复读这两种隔离级别。而未提交读隔离级别总是读取最新的数据行，无需使用 MVCC。可串行化隔离级别需要对所有读取的行都加锁，单纯使用 MVCC 无法实现。

#### 基础概念

**版本号**

- 系统版本号：是一个递增的数字，每开始一个新的事务，系统版本号就会自动递增。
- 事务版本号：事务开始时的系统版本号

**隐藏的列**

MVCC 在每行记录后面都保存着两个隐藏的列，用来存储两个版本号：

- 创建版本号：指示创建一个数据行的快照时的系统版本号；
- 删除版本号：如果该快照的删除版本号大于当前事务版本号表示该快照有效，否则表示该快照已经被删除了。

**Undo 日志**

MVCC 使用到的快照存储在 Undo 日志中，该日志通过回滚指针把一个数据行（Record）的所有快照连接起来。

![img](https://mmbiz.qpic.cn/mmbiz_jpg/uChmeeX1FpwqHyYbEIPyeesNicgZ2s5NTicpeQdfCyQT59DuCHeT2QEoPfpjz0zvjYlAmX4vD3IN1kAE9aNibBibdg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

#### 实现过程

以下实现过程针对可重复读隔离级别。

当开始一个事务时，该事务的版本号肯定大于当前所有数据行快照的创建版本号，理解这一点很关键。数据行快照的创建版本号是创建数据行快照时的系统版本号，系统版本号随着创建事务而递增，因此新创建一个事务时，这个事务的系统版本号比之前的系统版本号都大，也就是比所有数据行快照的创建版本号都大。

**SELECT**

多个事务必须读取到同一个数据行的快照，并且这个快照是距离现在最近的一个有效快照。但是也有例外，如果有一个事务正在修改该数据行，那么它可以读取事务本身所做的修改，而不用和其它事务的读取结果一致。

把没有对一个数据行做修改的事务称为 T，T 所要读取的数据行快照的创建版本号必须小于等于 T 的版本号，因为如果大于 T 的版本号，那么表示该数据行快照是其它事务的最新修改，因此不能去读取它。除此之外，T 所要读取的数据行快照的删除版本号必须是未定义或者大于 T 的版本号，因为如果小于等于 T 的版本号，那么表示该数据行快照是已经被删除的，不应该去读取它。

**INSERT**

将当前系统版本号作为数据行快照的创建版本号。

**DELETE**

将当前系统版本号作为数据行快照的删除版本号。

**UPDATE**

将当前系统版本号作为更新前的数据行快照的删除版本号，并将当前系统版本号作为更新后的数据行快照的创建版本号。可以理解为先执行 DELETE 后执行 INSERT。

#### 快照读与当前读

在可重复读级别中，通过MVCC机制，虽然让数据变得可重复读，但我们读到的数据可能是历史数据，是不及时的数据，不是数据库当前的数据！这在一些对于数据的时效特别敏感的业务中，就很可能出问题。

对于这种读取历史数据的方式，我们叫它快照读 (snapshot read)，而读取数据库当前版本数据的方式，叫当前读 (current read)。很显然，在MVCC中：

**快照读**

MVCC 的 SELECT 操作是快照中的数据，不需要进行加锁操作。

```
select * from table ….;
```

**当前读**

MVCC 其它会对数据库进行修改的操作（INSERT、UPDATE、DELETE）需要进行加锁操作，从而读取最新的数据。可以看到 MVCC 并不是完全不用加锁，而只是避免了 SELECT 的加锁操作。

```
INSERT;
UPDATE;
DELETE;
```

在进行 SELECT 操作时，可以强制指定进行加锁操作。以下第一个语句需要加 S 锁，第二个需要加 X 锁。

```
- select * from table where ? lock in share mode;
- select * from table where ? for update;
```

事务的隔离级别实际上都是定义的当前读的级别，MySQL为了减少锁处理（包括等待其它锁）的时间，提升并发能力，引入了快照读的概念，使得select不用加锁。而update、insert这些“当前读”的隔离性，就需要通过加锁来实现了。





## 优化

### 执行计划

MySQL8.0之前的数据库explain执行的时候加上SQL  NOCache去跑sql

MySQL版本支持缓存且开启缓存后，每次请求的查询语句和结果都会以key-value的形式缓存在内存中。

explain字段及含义

![img](https://mmbiz.qpic.cn/mmbiz_jpg/uChmeeX1Fpz8Sib2PsicNMxibibNic5p0JicUd3meF9QxKeCibc1CaVKMNsO7Jmic486yrgzTYhyIib9PSjmZMFJ2oPg6sQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

select_type:常用的有 SIMPLE 简单查询，UNION 联合查询，SUBQUERY 子查询等。

table:要查询的表

type： 索引查询类型，type表示访问方式，从好到坏依次为：NULL,system,const,eq_ref,ref，fulltext,ref_or_null,unique_subquery,range,index_merge,index,ALL。

pessible_keys:可选择的索引

key：实际使用的索引

rows：扫描的行数

key_len：表示使用的索引在表定义中的长度

ref：根据官方文档，ref是指用来与key中所选索引列比较的常量(const)或者连接查询列（显示为该列名字）

filtered：表示存储引擎返回的数据在server层过滤后，剩下多少满足查询的记录数量的比例。即实际剩下的行数为rows xfiltered/100

extra：这是mysql认为很关键，但是又不应该出现在前面所述字段的信息。





*NULL：mysql在优化过程中通过访问引擎统计信息直接得到结果，不用访问表或者索引，比如获取myisam表的行数，或者从索引列选取最小值。

*const，system：mysql会将查询可能的部分转换为常量。它们的区别是，system的表中仅有一行，而const则是表中仅有一行匹配行。

*eq_ref：使用多表连接时，仅返回一行记录，即使用primary key或者unique key。

*ref：使用非唯一索引或者唯一索引的前缀等值匹配时。可以返回多行，但仅匹配一个值。

*ref_or_null：与ref类似，只是增加了null值的比较。

*fulltext：全文索引，优先级很高，与普通索引同时存在时，mysql优先选择全文索引，由于导入的示例数据库没有MyISAM的表，故无法测试该type了。

*unique_subquery：用于where中in形式的子查询，子查询返回不重复的唯一值

*index_subquery：类似unique_subquery，不过可以返回重复的值。

*range：索引范围扫描，当索引使用<,>,is null,between,in,like时出现。

*index_merge：查询使用了两个以上的索引，mysql优化后对结果去交集或者并集。

*index：索引全表扫描，用索引把全表从头到尾扫一遍。当用force index或者覆盖索引全表扫描时出现（如下）。

*ALL：全表扫描，然后在server层过滤记录。





至于MySQL索引可能走错也很好理解，如果走A索引要扫描100行，B所有只要20行，但是他可能选择走A索引，你可能会想MySQL是不是有病啊，其实不是的。

一般走错都是因为优化器在选择的时候发现，走A索引没有额外的代价，比如走B索引并不能直接拿到我们的值，还需要回到主键索引才可以拿到，多了一次回表的过程，这个也是会被优化器考虑进去的。

他发现走A索引不需要回表，没有额外的开销，所有他选错了。

如果是上面的统计信息错了，那简单，我们用analyze table tablename 就可以重新统计索引信息了，所以在实践中，如果你发现explain的结果预估的rows值跟实际情况差距比较大，可以采用这个方法来处理。

还有一个方法就是force index强制走正确的索引，或者优化SQL，最后实在不行，可以新建索引，或者删掉错误的索引。

### 聚集索引

```
select itemId from itemCenter where size between 1 and 6
```

因为商品id itemId一般都是主键，在size索引上肯定会有我们这个值，这个时候就不需要回主键表去查询id信息了。

由于覆盖索引可以减少树的搜索次数，显著提升查询性能，所以使用覆盖索引是一个常用的性能优化手段。



### 联合索引

还是商品表举例，我们需要根据他的名称，去查他的库存，假设这是一个很高频的查询请求，你会怎么建立索引呢？是的建立一个，名称和库存的联合索引，这样名称查出来就可以看到库存了，不需要查出id之后去回表再查询库存了，联合索引在我们开发过程中也是常见的，但是并不是可以一直建立的，大家要思考索引占据的空间。



### 最左匹配原则

大家在写sql的时候，最好能利用到现有的SQL最大化利用，像上面的场景，如果利用一个模糊查询 itemname like ’敖丙%‘，这样还是能利用到这个索引的，而且如果有这样的联合索引，大家也没必要去新建一个商品名称单独的索引了。

### 索引下推

联合索引时都差一个



### 唯一索引和普通索引如何选择

要回答到change buffer。

当需要更新一个数据页时，如果数据页在内存中就直接更新，而如果这个数据页还没有在内存中的话，在不影响数据一致性的前提下，InooDB会将这些更新操作缓存在change buffer中，这样就不需要从磁盘中读入这个数据页了。

在下次查询需要访问这个数据页的时候，将数据页读入内存，然后执行change buffer中与这个页有关的操作，通过这种方式就能保证这个数据逻辑的正确性。

需要说明的是，虽然名字叫作change buffer，实际上它是可以持久化的数据。也就是说，change buffer在内存中有拷贝，也会被写入到磁盘上。

将change buffer中的操作应用到原数据页，得到最新结果的过程称为merge。

除了访问这个数据页会触发merge外，系统有后台线程会定期merge。在数据库正常关闭（shutdown）的过程中，也会执行merge操作。

![img](https://mmbiz.qpic.cn/mmbiz_jpg/uChmeeX1Fpz8Sib2PsicNMxibibNic5p0JicUdJDYUT3ic7xe6r3o64DgNojLb6S26DOibWfksos9sCRdIYTN9AdGkc1Eg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

显然，如果能够将更新操作先记录在change buffer，减少读磁盘，语句的执行速度会得到明显的提升。而且，数据读入内存是需要占用buffer pool的，所以这种方式还能够避免占用内存，提高内存利用率。

对于唯一索引来说，所有的更新操作都要先判断这个操作是否违反唯一性约束。

要判断表中是否存在这个数据，而这必须要将数据页读入内存才能判断，如果都已经读入到内存了，那直接更新内存会更快，就没必要使用change buffer了。

因此，唯一索引的更新就不能使用change buffer，实际上也只有普通索引可以使用。

change buffer用的是buffer pool里的内存，因此不能无限增大，change buffer的大小，可以通过参数innodb_change_buffer_max_size来动态设置，这个参数设置为50的时候，表示change buffer的大小最多只能占用buffer pool的50%。

将数据从磁盘读入内存涉及随机IO的访问，是数据库里面成本最高的操作之一，change buffer因为减少了随机磁盘访问，所以对更新性能的提升是会很明显的。

因为merge的时候是真正进行数据更新的时刻，而change buffer的主要目的就是将记录的变更动作缓存下来，所以在一个数据页做merge之前，change buffer记录的变更越多（也就是这个页面上要更新的次数越多），收益就越大。

因此，对于写多读少的业务来说，页面在写完以后马上被访问到的概率比较小，此时change buffer的使用效果最好，这种业务模型常见的就是账单类、日志类的系统。

反过来，假设一个业务的更新模式是写入之后马上会做查询，那么即使满足了条件，将更新先记录在change buffer，但之后由于马上要访问这个数据页，会立即触发merge过程。这样随机访问IO的次数不会减少，反而增加了change buffer的维护代价，所以，对于这种业务模式来说，change buffer反而起到了副作用。

### 前缀索引

MySQL是支持前缀索引的，也就是说，你可以定义字符串的一部分作为索引。默认地，如果你创建索引的语句不指定前缀长度，那么索引就会包含整个字符串。

### 函数与索引

对索引字段做函数操作，可能会破坏索引值的有序性，因此优化器就决定放弃走树搜索功能。

这个时候大家可以用一些取巧的方法，比如 select * from tradelog where id + 1 = 10000 就走不上索引，select * from tradelog where id = 9999就可以。

select * from t where id = 1

如果id是字符类型的，1是数字类型的，你用explain会发现走了全表扫描，根本用不上索引，为啥呢？

因为MySQL底层会对你的比较进行转换，相当于加了 CAST( id AS signed int) 这样的一个函数，上面说过函数会导致走不上索引。

还是一样的问题，如果两个表的字符集不一样，一个是utf8mb4，一个是utf8，因为utf8mb4是utf8的超集，所以一旦两个字符比较，就会转换为utf8mb4再比较。

### flush

redo log大家都知道，也就是我们对数据库操作的日志，他是在内存中的，每次操作一旦写了redo log就会立马返回结果，但是这个redo log总会找个时间去更新到磁盘，这个操作就是flush。

在更新之前，当内存数据页跟磁盘数据页内容不一致的时候，我们称这个内存页为“脏页”。

发生flush的场景

![img](https://mmbiz.qpic.cn/mmbiz_jpg/uChmeeX1Fpz8Sib2PsicNMxibibNic5p0JicUdIDDZn9DklW7ib3T6HgTQLEJDbJutVebGaic4kibqx9Uv90IxL7GbHrkgA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



Innodb刷脏页控制策略，我们每个电脑主机的io能力是不一样的，你要正确地告诉InnoDB所在主机的IO能力，这样InnoDB才能知道需要全力刷脏页的时候，可以刷多快。

​	这就要用到innodb_io_capacity这个参数了，它会告诉InnoDB你的磁盘能力，这个值建议设置成磁盘的IOPS，磁盘的IOPS可以通过fio这个工具来测试。

正确地设置innodb_io_capacity参数，可以有效的解决这个问题。



## 索引

### B+ Tree

B Tree 指的是 Balance Tree，也就是平衡树，平衡树是一颗查找树，并且所有叶子节点位于同一层。

**B+ Tree 是 B 树的一种变形，它是基于 B Tree 和叶子节点顺序访问指针进行实现，通常用于数据库和操作系统的文件系统中。**

B+ 树有两种类型的节点：内部节点（也称索引节点）和叶子节点，内部节点就是非叶子节点，内部节点不存储数据，只存储索引，数据都存在叶子节点。

内部节点中的 key 都按照从小到大的顺序排列，对于内部节点中的一个 key，左子树中的所有 key 都小于它，右子树中的 key 都大于等于它，叶子节点的记录也是按照从小到大排列的。

每个叶子节点都存有相邻叶子节点的指针。

![img](https://mmbiz.qpic.cn/mmbiz_jpg/uChmeeX1FpwqHyYbEIPyeesNicgZ2s5NT3GcgC6qgN63zsZk932mZdib9nuGQYHHxgd3icfOG32nZh973IgrmBwNA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

与红黑树比较

红黑树等平衡树也可以用来实现索引，但是文件系统及数据库系统普遍采用 B+ Tree 作为索引结构，主要有以下两个原因：

（一）磁盘 IO 次数

B+ 树一个节点可以存储多个元素，相对于红黑树的树高更低，磁盘 IO 次数更少。

（二）磁盘预读特性

为了减少磁盘 I/O 操作，磁盘往往不是严格按需读取，而是每次都会预读。预读过程中，磁盘进行顺序读取，顺序读取不需要进行磁盘寻道。每次会读取页的整数倍。

操作系统一般将内存和磁盘分割成固定大小的块，每一块称为一页，内存与磁盘以页为单位交换数据。数据库系统将索引的一个节点的大小设置为页的大小，使得一次 I/O 就能完全载入一个节点。

与B树比较

**B+ 树的磁盘 IO 更低**

B+ 树的内部节点并没有指向关键字具体信息的指针。因此其内部节点相对 B 树更小。如果把所有同一内部结点的关键字存放在同一盘块中，那么盘块所能容纳的关键字数量也越多。一次性读入内存中的需要查找的关键字也就越多。相对来说IO读写次数也就降低了。

**B+ 树的查询效率更加稳定**

由于非叶子结点并不是最终指向文件内容的结点，而只是叶子结点中关键字的索引。所以任何关键字的查找必须走一条从根结点到叶子结点的路。所有关键字查询的路径长度相同，导致每一个数据的查询效率相当。

**B+ 树元素遍历效率高**

B 树在提高了磁盘IO性能的同时并没有解决元素遍历的效率低下的问题。正是为了解决这个问题，B+树应运而生。B+树只要遍历叶子节点就可以实现整棵树的遍历。而且在数据库中基于范围的查询是非常频繁的，而 B 树不支持这样的操作（或者说效率太低）。



数据库中的B+Tree索引可以分为聚集索引（clustered index）和辅助索引（secondary index）。

上面的B+Tree示例图在数据库中的实现即为聚集索引，聚集索引的B+Tree中的叶子节点存放的是整张表的行记录数据，辅助索引与聚集索引的区别在于辅助索引的叶子节点并不包含行记录的全部数据，而是存储相应行数据的聚集索引键，即主键。

当通过辅助索引来查询数据时，InnoDB存储引擎会遍历辅助索引找到主键，然后再通过主键在聚集索引中找到完整的行记录数据。

InnoDB存储引擎中页的大小为16KB，一般表的主键类型为INT（占用4个字节）或BIGINT（占用8个字节），指针类型也一般为4或8个字节，也就是说一个页（B+Tree中的一个节点）中大概存储16KB/(8B+8B)=1K个键值（因为是估值，为方便计算，这里的K取值为〖10〗^3）。

也就是说一个深度为3的B+Tree索引可以维护10^3 * 10^3 * 10^3 = 10亿 条记录。（这种计算方式存在误差，而且没有计算叶子节点，如果计算叶子节点其实是深度为4了）





### 聚簇索引

InnoDB 的 B+Tree 索引分为主索引和辅助索引。主索引的叶子节点 data 域记录着完整的数据记录，这种索引方式被称为聚簇索引。因为无法把数据行存放在两个不同的地方，所以一个表只能有一个聚簇索引。

![img](https://mmbiz.qpic.cn/mmbiz_jpg/uChmeeX1FpwqHyYbEIPyeesNicgZ2s5NT4u7Q1jkb9dRLSkQN6KnUyIAGGPg8fCgYsFs06yZSpr1pe9hE4IDtGA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

辅助索引的叶子节点的 data 域记录着主键的值，因此在使用辅助索引进行查找时，需要先查找到主键值，然后再到主索引中进行查找，这个过程也被称作回表。

![img](https://mmbiz.qpic.cn/mmbiz_jpg/uChmeeX1FpwqHyYbEIPyeesNicgZ2s5NTVwSEmEEtZFEiahLGibwt2SPzhA2Sym5NSiaeEdcGciasVuIIia6jDl8REqQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 哈希索引

哈希索引能以 O(1) 时间进行查找，但是失去了有序性：

- 无法用于排序与分组；
- 只支持精确查找，无法用于部分查找和范围查找。

InnoDB 存储引擎有一个特殊的功能叫“自适应哈希索引”，当某个索引值被使用的非常频繁时，会在 B+Tree 索引之上再创建一个哈希索引，这样就让 B+Tree 索引具有哈希索引的一些优点，比如快速的哈希查找。



### 全文索引

用于查找文本中的关键词，而不是直接比较是否相等。全文索引使用倒排索引实现，它记录着关键词到其所在文档的映射。

### 空间数据索引

MyISAM 存储引擎支持空间数据索引（R-Tree），可以用于地理数据存储。空间数据索引会从所有维度来索引数据，可以有效地使用任意维度来进行组合查询。

必须使用 GIS 相关的函数来维护数据。







## 分区

### 水平切分

水平切分又称为 Sharding，它是将同一个表中的记录拆分到多个结构相同的表中。

当一个表的数据不断增多时，Sharding 是必然的选择，它可以将数据分布到集群的不同节点上，从而缓存单个数据库的压力。

![img](https://mmbiz.qpic.cn/mmbiz_jpg/uChmeeX1FpwqHyYbEIPyeesNicgZ2s5NTDY4S3c0lral4cRic5FGqO5U5RicuhKx7VqPgk75CC6Ra9Vhc7PCdM12w/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### 垂直切分

垂直切分是将一张表按列分成多个表，通常是按照列的关系密集程度进行切分，也可以利用垂直气氛将经常被使用的列喝不经常被使用的列切分到不同的表中。

在数据库的层面使用垂直切分将按数据库中表的密集程度部署到不通的库中，例如将原来电商数据部署库垂直切分称商品数据库、用户数据库等。

![img](https://mmbiz.qpic.cn/mmbiz_jpg/uChmeeX1FpwqHyYbEIPyeesNicgZ2s5NTjeVApdic37kA3icLTWwFHHj33WLFiaFBTlqxqPvBpZ8icUWv0f3FWJh27w/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### Sharding 策略

- 哈希取模：hash(key)%N
- 范围：可以是 ID 范围也可以是时间范围
- 映射表：使用单独的一个数据库来存储映射关系

### Sharding 存在的问题

**事务问题**

使用分布式事务来解决，比如 XA 接口

**连接**

可以将原来的连接分解成多个单表查询，然后在用户程序中进行连接。

**唯一性**

- 使用全局唯一 ID （GUID）
- 为每个分片指定一个 ID 范围
- 分布式 ID 生成器（如 Twitter 的 Snowflake 算法）







## 集群

### 主从复制

主从复制是指将主数据库的DDL和DML操作通过二进制日志传到从数据库上，然后在从数据库上对这些日志进行重新执行，从而使从数据库和主数据库的数据保持一致。

原理

- MySql主库在事务提交时会把数据变更作为事件记录在二进制日志Binlog中；
- 主库推送二进制日志文件Binlog中的事件到从库的中继日志Relay Log中，之后从库根据中继日志重做数据变更操作，通过逻辑复制来达到主库和从库的数据一致性；
- MySql通过三个线程来完成主从库间的数据复制，其中Binlog Dump线程跑在主库上，I/O线程和SQL线程跑着从库上；
- 当在从库上启动复制时，首先创建I/O线程连接主库，主库随后创建Binlog Dump线程读取数据库事件并发送给I/O线程，I/O线程获取到事件数据后更新到从库的中继日志Relay Log中去，之后从库上的SQL线程读取中继日志Relay Log中更新的数据库事件并应用，如下图所示。

![img](https://mmbiz.qpic.cn/mmbiz_png/CKvMdchsUwlc9vLQe9Ugu4Q3ZUQu9ezDKRmmK2hsjoicDbdDF7PTgTxAqZy9CD3mow4wOhlMK4JTVRBVFygSnWg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)





# RocketMQ

核心模块：

- rocketmq-broker：接受生产者发来的消息并存储（通过调用rocketmq-store），消费者从这里取得消息

- rocketmq-client：提供发送、接受消息的客户端API。

- rocketmq-namesrv：NameServer，类似于Zookeeper，这里保存着消息的TopicName，队列等运行时的元信息。

- rocketmq-common：通用的一些类，方法，数据结构等。

- rocketmq-remoting：基于Netty4的client/server + fastjson序列化 + 自定义二进制协议。

- rocketmq-store：消息、索引存储等。

- rocketmq-filtersrv：消息过滤器Server，需要注意的是，要实现这种过滤，需要上传代码到MQ！（一般而言，我们利用Tag足以满足大部分的过滤需求，如果更灵活更复杂的过滤需求，可以考虑filtersrv组件）。

- rocketmq-tools：命令行工具。

  ![img](https://mmbiz.qpic.cn/mmbiz_jpg/uChmeeX1Fpy6iciaD6rYR19SPHCBETqSo4oias9HIyejEiaQIXS5ybI537nC6OoGpJ6tsiar3hlgjH0l5I0CibGMoAeA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



他主要有四大核心组成部分：**NameServer**、**Broker**、**Producer**以及**Consumer**四部分。

![img](https://mmbiz.qpic.cn/mmbiz_jpg/uChmeeX1Fpy6iciaD6rYR19SPHCBETqSo4qEfYsQxwVmhcwpqxrvDFGJJ5kNNS0QGOIrXB2QrNSd9hs4fnnyKhUQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



### NameServer

> 主要负责对于源数据的管理，包括了对于**Topic**和路由信息的管理。

**NameServer**是一个功能齐全的服务器，其角色类似Dubbo中的Zookeeper，但NameServer与Zookeeper相比更轻量。主要是因为每个NameServer节点互相之间是独立的，没有任何信息交互。

**NameServer**压力不会太大，平时主要开销是在维持心跳和提供Topic-Broker的关系数据。

但有一点需要注意，Broker向NameServer发心跳时， 会带上当前自己所负责的所有**Topic**信息，如果**Topic**个数太多（万级别），会导致一次心跳中，就Topic的数据就几十M，网络情况差的话， 网络传输失败，心跳失败，导致NameServer误认为Broker心跳失败。

**NameServer** 被设计成几乎无状态的，可以横向扩展，节点之间相互之间无通信，通过部署多台机器来标记自己是一个伪集群。

每个 Broker 在启动的时候会到 NameServer 注册，Producer 在发送消息前会根据 Topic 到 **NameServer** 获取到 Broker 的路由信息，Consumer 也会定时获取 Topic 的路由信息。

所以从功能上看NameServer应该是和 ZooKeeper 差不多，据说 RocketMQ 的早期版本确实是使用的 ZooKeeper ，后来改为了自己实现的 NameServer 。

### Producer

> 消息生产者，负责产生消息，一般由业务系统负责产生消息。

- **Producer**由用户进行分布式部署，消息由**Producer**通过多种负载均衡模式发送到**Broker**集群，发送低延时，支持快速失败。
- **RocketMQ** 提供了三种方式发送消息：同步、异步和单向
- **同步发送**：同步发送指消息发送方发出数据后会在收到接收方发回响应之后才发下一个数据包。一般用于重要通知消息，例如重要通知邮件、营销短信。
- **异步发送**：异步发送指发送方发出数据后，不等接收方发回响应，接着发送下个数据包，一般用于可能链路耗时较长而对响应时间敏感的业务场景，例如用户视频上传后通知启动转码服务。
- **单向发送**：单向发送是指只负责发送消息而不等待服务器回应且没有回调函数触发，适用于某些耗时非常短但对可靠性要求并不高的场景，例如日志收集。



### Broker

> 消息中转角色，负责**存储消息**，转发消息。

- **Broker**是具体提供业务的服务器，单个Broker节点与所有的NameServer节点保持长连接及心跳，并会定时将**Topic**信息注册到NameServer，顺带一提底层的通信和连接都是**基于Netty实现**的。
- **Broker**负责消息存储，以Topic为纬度支持轻量级的队列，单机可以支撑上万队列规模，支持消息推拉模型。
- 官网上有数据显示：具有**上亿级消息堆积能力**，同时可**严格保证消息的有序性**。



### Consumer

> 消息消费者，负责消费消息，一般是后台系统负责异步消费。

- **Consumer**也由用户部署，支持PUSH和PULL两种消费模式，支持**集群消费**和**广播消息**，提供**实时的消息订阅机制**。
- **Pull**：拉取型消费者（Pull Consumer）主动从消息服务器拉取信息，只要批量拉取到消息，用户应用就会启动消费过程，所以 Pull 称为主动消费型。
- **Push**：推送型消费者（Push Consumer）封装了消息的拉取、消费进度和其他的内部维护工作，将消息到达时执行的回调接口留给用户应用程序来实现。所以 Push 称为被动消费类型，但从实现上看还是从消息服务器中拉取消息，不同于 Pull 的是 Push 首先要注册消费监听器，当监听器处触发后才开始消费消息。





## 消息领域模型

![img](https://mmbiz.qpic.cn/mmbiz_png/uChmeeX1Fpy6iciaD6rYR19SPHCBETqSo4BqkagB4km60k4fpJuynd2awiciciad45MvFib5Wiaf50cJiczzbHABwNjKKg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



#### Message

**Message**（消息）就是要传输的信息。

一条消息必须有一个主题（Topic），主题可以看做是你的信件要邮寄的地址。

一条消息也可以拥有一个可选的标签（Tag）和额处的键值对，它们可以用于设置一个业务 Key 并在 Broker 上查找此消息以便在开发期间查找问题。

#### Topic

**Topic**（主题）可以看做消息的规类，它是消息的第一级类型。比如一个电商系统可以分为：交易消息、物流消息等，一条消息必须有一个 Topic 。

**Topic** 与生产者和消费者的关系非常松散，一个 Topic 可以有0个、1个、多个生产者向其发送消息，一个生产者也可以同时向不同的 Topic 发送消息。

一个 Topic 也可以被 0个、1个、多个消费者订阅。

#### Tag

**Tag**（标签）可以看作子主题，它是消息的第二级类型，用于为用户提供额外的灵活性。使用标签，同一业务模块不同目的的消息就可以用相同 Topic 而不同的 **Tag** 来标识。比如交易消息又可以分为：交易创建消息、交易完成消息等，一条消息可以没有 **Tag** 。

标签有助于保持您的代码干净和连贯，并且还可以为 **RocketMQ** 提供的查询系统提供帮助。

#### Group

分组，一个组可以订阅多个Topic。

分为ProducerGroup，ConsumerGroup，代表某一类的生产者和消费者，一般来说同一个服务可以作为Group，同一个Group一般来说发送和消费的消息都是一样的

#### Queue

在**Kafka**中叫Partition，每个Queue内部是有序的，在**RocketMQ**中分为读和写两种队列，一般来说读写队列数量一致，如果不一致就会出现很多问题。

#### Message Queue

**Message Queue**（消息队列），主题被划分为一个或多个子主题，即消息队列。

一个 Topic 下可以设置多个消息队列，发送消息时执行该消息的 Topic ，RocketMQ 会轮询该 Topic 下的所有队列将消息发出去。

消息的物理管理单位。一个Topic下可以有多个Queue，Queue的引入使得消息的存储可以分布式集群化，具有了水平扩展能力。

#### Offset

在**RocketMQ** 中，所有消息队列都是持久化，长度无限的数据结构，所谓长度无限是指队列中的每个存储单元都是定长，访问其中的存储单元使用Offset 来访问，Offset 为 java long 类型，64 位，理论上在 100年内不会溢出，所以认为是长度无限。

也可以认为 Message Queue 是一个长度无限的数组，**Offset** 就是下标。



![img](https://mmbiz.qpic.cn/mmbiz_png/azicia1hOY6QibLp3AbOORg5CnMu7EI8xMoDf2uzylnKvoibHP5USPQ6TbWxLLgPQia2LjCshCrpqjlgP2fVxZ4R2iag/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)





顺序消费



**一个topic下有多个队列**，为了保证发送有序，**RocketMQ**提供了**MessageQueueSelector**队列选择机制，他有三种实现:

![img](https://mmbiz.qpic.cn/mmbiz_png/uChmeeX1FpzjaiaLbo3bKFDic8zvesZQQYqNXMwicnafsv3oSedFeWxvQXxd7iaK1nYwHCVu2ria6iaJsBbqHhl7hZqw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

我们可使用**Hash取模法**，让同一个订单发送到同一个队列中，再使用同步发送，只有同个订单的创建消息发送成功，再发送支付消息。这样，我们保证了发送有序。

**RocketMQ**的topic内的队列机制,可以保证存储满足**FIFO**（First Input First Output 简单说就是指先进先出）,剩下的只需要消费者顺序消费即可。

**RocketMQ**仅保证顺序发送，顺序消费由消费者业务保证!!!





















# RPC



## 分布式事务



### 2pc

2PC（Two-phase commit protocol），中文叫二阶段提交。**二阶段提交是一种强一致性设计**，2PC 引入一个事务协调者的角色来协调管理各参与者（也可称之为各本地资源）的提交和回滚，二阶段分别指的是准备（投票）和提交两个阶段。

**准备阶段**协调者会给各参与者发送准备命令，你可以把准备命令理解成除了提交事务之外啥事都做完了。

![img](https://mmbiz.qpic.cn/mmbiz_jpg/uChmeeX1Fpx2WyATKXiadnNrvpBPNyme8FLuMib3gXy1SPIYeaaxFtoib1JWtVo7iaba3FQTs88FuZ4poDARXmg2qA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



假如在第一阶段有一个参与者返回失败，那么协调者就会向所有参与者发送回滚事务的请求，即分布式事务执行失败。

![img](https://mmbiz.qpic.cn/mmbiz_jpg/uChmeeX1Fpx2WyATKXiadnNrvpBPNyme8scMRr5aRqR1uiaHN0bKNAcbcmqL0Sf2G5TNtuEDcYtQQqiczXq58HjwQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)







假如在第一阶段有一个参与者返回失败，那么协调者就会向所有参与者发送回滚事务的请求，即分布式事务执行失败。

![img](https://mmbiz.qpic.cn/mmbiz_jpg/uChmeeX1Fpx2WyATKXiadnNrvpBPNyme8scMRr5aRqR1uiaHN0bKNAcbcmqL0Sf2G5TNtuEDcYtQQqiczXq58HjwQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

那可能就有人问了，那第二阶段提交失败的话呢？

这里有两种情况。

第一种是**第二阶段执行的是回滚事务操作**，那么答案是不断重试，直到所有参与者都回滚了，不然那些在第一阶段准备成功的参与者会一直阻塞着。

第二种是**第二阶段执行的是提交事务操作**，那么答案也是不断重试，因为有可能一些参与者的事务已经提交成功了，这个时候只有一条路，就是头铁往前冲，不断的重试，直到提交成功，到最后真的不行只能人工介入处理。

首先 2PC 是一个**同步阻塞协议**，像第一阶段协调者会等待所有参与者响应才会进行下一步操作，当然第一阶段的**协调者有超时机制**，假设因为网络原因没有收到某参与者的响应或某参与者挂了，那么超时后就会判断事务失败，向所有参与者发送回滚命令。

在第二阶段协调者的没法超时，因为按照我们上面分析只能不断重试！

### 3PC

3PC 的出现是为了解决 2PC 的一些问题，相比于 2PC 它在**参与者中也引入了超时机制**，并且**新增了一个阶段**使得参与者可以利用这一个阶段统一各自的状态。

让我们来详细看一下。

3PC 包含了三个阶段，分别是**准备阶段、预提交阶段和提交阶段**，对应的英文就是：`CanCommit、PreCommit 和 DoCommit`。

![img](https://mmbiz.qpic.cn/mmbiz_jpg/uChmeeX1Fpx2WyATKXiadnNrvpBPNyme86EFEmv5s5sricicFs5t4J3XT8AVTWvhzic9wMYCkzwGVNBX2zJiaoGLt7w/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



首先**准备阶段的变更成不会直接执行事务**，而是会先去询问此时的参与者是否有条件接这个事务，因此**不会一来就干活直接锁资源**，使得在某些资源不可用的情况下所有参与者都阻塞着。

而**预提交阶段的引入起到了一个统一状态的作用**，它像一道栅栏，表明在预提交阶段前所有参与者其实还未都回应，在预处理阶段表明所有参与者都已经回应了。

但是多引入一个阶段也多一个交互，因此**性能会差一些**，而且**绝大部分的情况下资源应该都是可用的**，这样等于每次明知可用执行还得询问一次。

我们知道 2PC 是同步阻塞的，上面我们已经分析了协调者挂在了提交请求还未发出去的时候是最伤的，所有参与者都已经锁定资源并且阻塞等待着。

那么引入了超时机制，参与者就不会傻等了，**如果是等待提交命令超时，那么参与者就会提交事务了**，因为都到了这一阶段了大概率是提交的，**如果是等待预提交命令超时，那该干啥就干啥了，反正本来啥也没干**。

然而超时机制也会带来数据不一致的问题，比如在等待提交命令时候超时了，参与者默认执行的是提交事务操作，但是**有可能执行的是回滚操作，这样一来数据就不一致了**。

当然 3PC 协调者超时还是在的，具体不分析了和 2PC 是一样的。

### TCC

**2PC 和 3PC 都是数据库层面的，而 TCC 是业务层面的分布式事务**，就像我前面说的分布式事务不仅仅包括数据库的操作，还包括发送短信等，这时候 TCC 就派上用场了！

TCC 指的是`Try - Confirm - Cancel`。

- Try 指的是预留，即资源的预留和锁定，**注意是预留**。
- Confirm 指的是确认操作，这一步其实就是真正的执行了。
- Cancel 指的是撤销操作，可以理解为把预留阶段的动作撤销了。

其实从思想上看和 2PC 差不多，都是先试探性的执行，如果都可以那就真正的执行，如果不行就回滚。

比如说一个事务要执行A、B、C三个操作，那么先对三个操作执行预留动作。如果都预留成功了那么就执行确认操作，如果有一个预留失败那就都执行撤销动作。

![img](https://mmbiz.qpic.cn/mmbiz_jpg/uChmeeX1Fpx2WyATKXiadnNrvpBPNyme8g6iaCMWDl6yFUgf5eBha1TlG2k2MBUkYtO9uuYuILWRicbmkEMvNBMgQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



可以看到流程还是很简单的，难点在于业务上的定义，对于每一个操作你都需要定义三个动作分别对应`Try - Confirm - Cancel`。

因此 **TCC 对业务的侵入较大和业务紧耦合**，需要根据特定的场景和业务逻辑来设计相应的操作。

还有一点要注意，撤销和确认操作的执行可能需要重试，因此还需要保证**操作的幂等**。

相对于 2PC、3PC ，TCC 适用的范围更大，但是开发量也更大，毕竟都在业务上实现，而且有时候你会发现这三个方法还真不好写。不过也因为是在业务上实现的，所以**TCC可以跨数据库、跨不同的业务系统来实现事务**。



### 消息事务

RocketMQ 就很好的支持了消息事务，让我们来看一下如何通过消息实现事务。

第一步先给 Broker 发送事务消息即半消息，**半消息不是说一半消息，而是这个消息对消费者来说不可见**，然后**发送成功后发送方再执行本地事务**。

再根据**本地事务的结果向 Broker 发送 Commit 或者 RollBack 命令**。

并且 RocketMQ 的发送方会提供一个**反查事务状态接口**，如果一段时间内半消息没有收到任何操作请求，那么 Broker 会通过反查接口得知发送方事务是否执行成功，然后执行 Commit 或者 RollBack 命令。

如果是 Commit 那么订阅方就能收到这条消息，然后再做对应的操作，做完了之后再消费这条消息即可。

如果是 RollBack 那么订阅方收不到这条消息，等于事务就没执行过。

可以看到通过 RocketMQ 还是比较容易实现的，RocketMQ 提供了事务消息的功能，我们只需要定义好事务反查接口即可。

![img](https://mmbiz.qpic.cn/mmbiz_jpg/uChmeeX1Fpx2WyATKXiadnNrvpBPNyme85yCYdK8oUfjehf9OA3A3icLzX1TwiatNU6PkdqmQp0Y4pzY3HxHFJyvQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



可以看出 2PC 和 3PC 是一种强一致性事务，不过还是有数据不一致，阻塞等风险，而且只能用在数据库层面。

而 TCC 是一种补偿性事务思想，适用的范围更广，在业务层面实现，因此对业务的侵入性较大，每一个操作都需要实现对应的三个方法。

本地消息、事务消息和最大努力通知其实都是最终一致性事务，因此适用于一些对时间不敏感的业务。













