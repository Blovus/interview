# 计算机网络

### 网络分为几层及各层含义

主要有TCP/IP五层和OSI七层两种模式。

从底到高排序分别为：

TCP/IP五层：物理层 -》 数据链路层 - 》 网络层 -》 传输层 -》 应用层

OSI七层：物理层 -》 数据链路层 - 》 网络层 -》 传输层 -》会话层-》表示层-》 应用层

现实生活中主要用的是第一种。不用第二种的原因是，第二种过去理想化，并且出来的太晚了。当OSI出现时，实际生活中已经被TCP/IP五层完全覆盖，难以更改了。

物理层：实际物理内容，比如高低电频、引脚等配置。

 数据链路层：增加报头报尾保证数据完整。实现直连的两个设备之间进行通信。

 网络层：各设备之间通信。负责没有直连的两个网络之间进行通信。

传输层：各设备之间端口通信。

应用层：具体应用。



### TCP与UDP之间的区别

TCP：一般为20个字节；连接可靠，一对一连接；可靠；速度慢；支持分片；支持重传；数据包有顺序；有流控；基于字节流	

UDP：8字节；一般为无连接，可多对多；不可靠；快；不支持要交给IP去做，这样的话发生丢失，IP要把所有数据全部重发；不支持重传；数据包无序；无流控



### TCP三次握手过程

1、c 发送自己的同步序列编号SYN给s。比如SeqNum=X；

2、s确认c的序列号，发送确认消息的同时给c发送s的序列号。AckNum=X+1；SeqNum=Y；

3、c确认s的序列号并发消息。AckNum=Y+1；

服务端操作系统会有半连接队列和全连接队列。三次握手建立过程中，服务端 会把客户端信息放入半连接队列，握手结束后则会移到全连接队列，半连接队列会保持63s才会断开。linux默认，队列满的话就会丢弃新来的连接，但可以基于设置修改。

另外，linux可以设置TCP Fast Open功能可以绕过三次握手。第一次连接还需要三次握手。之后发送请求时都带Cookie，如果Cookie命中则步骤2之后可以直接发送数据，无需等待步骤三的确认。 

三次握手双方同步了序列号，防止旧的重复连接造成初始化混乱。两次的话序列号同步可能频繁重复分配连接造成资源浪费。

初始序列号ISN是基于时钟的随机算法。可保证4.5小时不重复。

### TCP四次握手过程

1、c发送FIN消息给s表明断开连接。

2、s发送ack给c表示确认断开。

3、s发送FIN给c表明断开连接。

4、c发送Ack表示确认断开。此时挥手结束。

之所以四次是因为TCP是全双工协议。但操作系统开启延迟确认时候，挥手可以只有三次。

### 简述TCP的重传、滑动窗口、拥塞控制

TCP为了提供可靠传输，采用了重传机制。

超时重传基于时间驱动，就是过了一段时间后没有收到另一方响应就重传数据。

快速重传则基于数据驱动。如果连续三次收到相同的ack确认号。则发送该包。sack的引入则是为了让快速重传，知道发送哪些数据。比如c已经发送了1，2，3，4，5五条消息.s的ack三次返回都为2，那么c是只重传2，还是要把后面3，4，5也重传，就需要看s的ack返回时附带的sack了。里面会写附带其他收到的消息序号。D-SACK则SACK的扩展，让s知道那些消息发重复了。

滑动窗口则用来控制发送的速率。接受方的窗口如果为0，则发送发不能再发送数据，发送方会发送探测接口直到接受方告诉发送方窗口不为0之后才能发送数据。另外如果接受方能接受的数据太小的话，也会告诉用户方窗口为0，避免每次浪费资源。或者发送方使用Nagle算法，等待接受变大了再发送数据。

滑动窗口协调了发送方和接收方的数据发送接收能力。拥塞控制则基于当前的网络状况来控制双方发送数据包的大小。核心为拥塞点ssthresh的测算。拥塞控制为慢启动，发包的大小cwnd从1开始，后续指数递增，到了拥塞点则变为每次cwnd+1线性递增。如果发生超时重传，拥塞点则变为当前的cwnd，cwnd再变为1。如果发生快速重传，则有两种算法，一种和超时重传一样。另一种拥塞点变为cwnd/2，cwnd变为cwnd/2+3，3是因为快速重传有三次；



### 什么是HTTP

HTTP全名为超文本传输协议。用于服务端和客户端之间的通信。是基于请求与响应模式应用层协议。特点是灵活、无状态、无连接。灵活指可以传输各种类型的数据对象，如音频、视频、图片、文件等；无状态指，每次HTTP请求都是独立的，两个请求之间没有关系，每个请求不会记忆；无连接指的是一次请求和响应结束后，通信就会断开。

HTTP之间的状态可以通过引入浏览器设置Cookie来保存。HTTP1.1支持持久连接。

（无连接这个可以不用提）

### 简述 HTTP1.0/1.1/2.0/3.0的区别

1.0:产生于1996年；不支持断点续传；使用短链接，每次发送数据都需要三次握手和四次挥手，效率低；

1.1:1999年出现；做常见的版本；默认长连接，连接时常基于请求头的keep-alive来设置；支持断点续传；使用了虚拟网络，一台物理机支持多个IP地址。	使用了管道。

2.0：使用HPACK进行了头部压缩；使用了二进制格式而非ASCLL码；支持多路复用，每一个请求都是连接共享；使用数据流，而非按照顺序发送

3.0：下层使用UDP而非TCP；头部压缩改用QPack算法。





### HTTP和HTTPS之间的区别

HTTP的端口是80。HTTPS的默认端口是443；

简单来说，HTTPS就是披了一层SSL的HTTP。对数据进行了加密，使得HTTPS成为安全的协议。通过密钥交换算法-签名算法-对称加密算法-摘要算法。

过程为三次握手之后发生之后。

1、c端向s端发送消息，里面会携带c的密码套件列表-即一些加密算法，如RAS算法、一个c生成的随机数c1以及支持的SSL/TLS协议版本。

2、s端收到c的消息后，会发送证书，s确认的密码套件，s生成的随机数s1，s的SSL/TLS协议版本。如果s的协议版本不支持，则直接关闭。

3、c确认s的证书的真实性。证书无问题则从证书中提取公钥。向s发送一个随机数p1，这个随机数会基于公钥加密，以及确认的加密算法。

4s端收到加密的随机数p1，用证书的私钥和双方确认的解密算法解密。以后双发的密钥为 （c1+s1+p1），以此来进行对称加密解密。



### Get和Post之间的区别

Get：不安全；有长度限制；会被浏览器cache缓存；发送过程中只有一个tcp数据包；

Post：安全；无长度限制；除非手动设置不然不会被浏览器cache缓存；发送过程中产生两个tcp数据包；



### IP协议有哪些相关协议

ARP通过广播获取IP数据包的下一跳的MAC地址。操作系统通常会把这个MAC缓存起来。

RAAP和ARP相反，是通过MAC获取IP。

ICMP为互联网报文控制协议。IP如果发送途中失败，具体原因就由ICMP控制。还可用于诊断查询。

IGMP则为因特网组管理协议，工作在主机（组播成员）和最后一跳上。



### 地址输入url会发生什么

1、根据url解析，浏览器确定了WEB服务器和文件名。

2、根据url地址查询web服务器的ip地址。

2.1查询本地DNS缓存

2.1.1、如果浏览器支持缓存，则查询浏览器缓存；

2.1.2如果没有找到，则查询本地hosts文件是否有配置该域名的ip地址；

2.2如果没有，则向网络发起DNS查询。

2.2.1向本地DNS服务器查询。

2.2.2如果没找到，本地DNS服务器向根服务器发送DNS查询。根域名服务器为全局服务器

2.2.3根域名服务器此时分为递归和迭代查询两种。如果能告知本地DNS下一步需要访问哪个顶级域名服务器则为前者，不然为后者。

2.2.4本地服务器经过根域名服务器、顶级域名服务器——即com、org、edu这种服务器，再到权威域名服务器——即具体需要查询的域名的服务器获得了IP。本地在告诉客户端。客户端开始准备和服务端的连接

3.整理数据

3.1浏览器调用Socket库，来委托操作系统的协议栈工作。

3.2.添加TCP报文

3.3添加IP报文

3.4添加MAC报文

3.5.数据经过网卡发送。变为电信号。

4.开始前往目的地址

4.1中间的MAC地址交换则为交换机发原则。交换机根据本地的缓存MAC地址表查找目的MAC地址，没有则发送给源端口之外的所有端口。

4.2.中间为路由器。路由器类似网卡，具有IP地址和MAC地址。如果没到目的地址，则根据目的的IP地址，发送ARP协议寻找MAC地址，找到下一跳地址，中间还是会通过交换机到下一个路由器地址。直至这个路由器的网关表表明自己就是终点目的地址。

5.服务端开始动作。

5.1服务端开始一层层拆掉数据包。

5.2建立TCP三次握手流程。

5.3浏览器向目标服务器发送HTTP-GET请求。

5.4返回响应内容

5.4.1如果页面简单,返回状态码为200，则结束。

5.4.2复杂的话则会返回以3开头的响应码做重定向。找到重定向地址后，在回到1步骤重来。

5.4.3	重新发送新请求，返回200，结束。



# 操作系统



# Redis



### Redis速度快的原因

完全基于内存；

数据结构简单；单线程；

I/O复用模型，采用非阻塞IO；

底层模型不同，自定义了VM机制，冷数据存到磁盘，热数据依然在内存，使用的并非操作系统的swap机制。key依然存放在内存，只将value放到磁盘。swap基于页去操作，一般为4k，对于redis来说太大了。



### Redis的数据结构及各个实现

String：基于SDS数据结构。SDS有长度len，未使用长度free，和具体的字符串char[] buff——但使用的是泛型而不是具体int。

Hash：两个hashtable。。新版本quicklist，旧版本zipList。

List：旧版本元素少时使用zipList，元素多的时候用linked双端链表。新版本使用quicklist。

linked双端链表插入快，但是由于每个节点维护前后内容存储消耗大，存储地址也不连续，容易产生碎片。

zipList存储连续，节省了内存，查询效率高。但难以修改。

quickList是一个ziplist组成的双向链表。每个节点使用ziplist来保存数据。本质上来说，quicklist里面保存着一个一个小的ziplist。

Set：Insert整数类型。加hashtable。

Sorted Set：跳跃表，底层zipList。

跳跃表实现简单，红黑树的rebalance 设计整棵树的操作，跳跃表只设计局部。

跳跃表的更新操作就是删除再新增。

### Redis的rehash

hash结构不仅用在字典外。Reids里所有的key和value也会组成一个全局字典。带过期时间的key也是一个字典存储在redisDB的数据结构中。

hash结构是数组+链表的结构来解决部分hash冲突。包含两个hashtable，通常只有一个HashTable有数值。扩容的时候，旧的hashtable会把数值一点点渐进式搬迁进新的hashtable。rehash期间新数据来了之后都放到，此时读取数据的话两个hashtable都会读。搬迁结束后，旧的会被删除。

扩容的触发条件是，元素的个数要大于第一维数组的长度时。但如果此时在做bgsave持久化，则不扩容。但如果此时达到第一维的5倍的话，就会强制扩容。

如果所有元素个数低于数组长度的百分之10，则会缩容。缩容不考虑bgsave/

### Redis的quickList，zipList，linkedlist





### Redis的雪崩、穿透、击穿

雪崩即同一时刻大量缓存到期，导致数据库查询崩溃。加随机数字就可以解决。

穿透即查无次内容，缓存不会命中，跳过缓存直接查库。请求参数如果做校验，不符合格式直接返回，缓存加空对象或使用布隆过滤器解决。

击穿为key到期后，大量请求查询该key，直接打挂数据了。将key

### Redis的主从复制与哨兵

主从复制流程

slave启动后，向master发送tcp连接。二者建立连接之后。slave向master发送ping消息，确实对方是redis实例。发送一个pysnc命令给master，作为第一次连接，master会把slave放入slave列表中，bgsave生成一次全量复制生成RDB全量快照，发送给slave，这期间生成的新请求会写在内存buffer里，等slave将快照内容装载进内存后，再把master内存buffer的请求发给slave。



哨兵组件的功能主要是

1、集群监控：监控各实例是否正常工作。

2、消息通知：如果某个实例挂了，哨兵负责告诉将故障转移的结果告知客户端。

3、故障转移：主节点挂了，转移到从节点。

4、配置提供：客户端在初始化时，通过连接哨兵来获得当前 Redis 服务的主节点地址。

sentinel 自身基于raft保证高可用。哨兵必须保证三个以上实例不然无用。哨兵+主从保证了高可用，但不保证数据不丢失。

哨兵的选主策略基于三点：1、节点的priority 设置的越低，优先级越高 2、节点复制的数据越多，优先级越高。3、runnid越小，越容易选中。

哨兵选中后，向那个服务器发送slave of no one命令。	



### Redis Cluster

各节点两两互相连接。客户端直连任意一台节点，就可以对其他节点进行读写操作。

redis集群内置了16384个哈希槽，即2的14次方。

如果数据不在自己节点，就会使用move命令进行跳转，告诉客户端去哪个节点操作。

集群主要负责数据分区，增加了容量。也提高了可用性。

集群使用了带有虚拟节点的一致性哈希分区，使用了槽，解耦了实际节点和数据之间的关系。让数据变得较为均匀，节点增删后数据也不会大量迁移。

集群的数据结构为cluster node 和cluster state。前者记录单个节点的状态，后者记录集群整体的状态。



### Redis 节点的通信

分为单对单，广播，Goosip三种。Goosip指随机与部分节点通信。最终所有节点达成一致。





### 谈谈持久化，RDB和AOF

RDB每个一段时间基于快照持久化生成.rdb文件，默认为五分钟；适合冷备；cow，copy on write，由于是fork子进程做的，对性能影响小，子进程和父进程共享数据段。恢复速度比AOF快。但是数据完整性可能会有丢失。

AOF是以append-only即日志添加的方式持久化，类似Mysql的binlog，一般每秒异步刷新日志；适合热备；因为没有什么磁盘寻址开销写入速度很快。但是同样的数据，AOF的文件比RDB的大。

一般集群时，主库压力大的话就不要做持久化。

### Redis内存淘汰机制

热弟说的过期策略为定期删除+惰性删除。如果内存满了则使用内存淘汰机制。常见的淘汰机制主要是LRU算法。

### 布隆过滤器

### 分布式锁

setnx类似于站坑。

锁需要设置超时。



# MySQL

### MySQL执行流程

先走连接器，连接成功后查询缓存，缓存如果命中直接返回，不然走分析器，再走优化器，再走执行器，再走具体的MySQL引擎查询结果。



### EXPLAIN和优化

索引不见得走到最快的索引。因为那个最快的索引可能需要回表操作。sql可以用force index语句强制走指定索引。使用覆盖索引、联合索引可以避免回表。



### InnoDB

新版本也支持全文索引。

内部有很多优化

基于lru算法，将很多数据存入内存。

采用B+树结构，稳定搜寻。

自带自适应哈希索引保存热数据。

插入操作使用change buffer优化。

可预测读，读取的时候从磁盘带出其他相关联的页。

两次写保证数据不丢失。

主索引为聚簇索引，里面自带了这条数据的所有内容。避免了磁盘读取开销。

支持事务。有mvcc，间隙锁，next-key-lock等。



### B+树

只在叶子节点存储数据。数据包含指针指向相邻数据。主索引叶子节点会存储整条数据，辅助索引只存储主键id。



### MVCC





### Binlog

### redolog

### undolog	

# RocketMQ

### 消息队列作用

异步、解耦、消峰

### 各组成部分

NameServer：主要负责对源数据的管理，包括对路由信息和topic的管理，类似于zk。无状态，可无限扩容。每个broker启动后，会到nameServer注册，发送心跳。Producer在发送消息前会根据Topic到nameServer获取Broker的路由信息，consumer也会定时获取Topic信息。

broker：

producer：

consumer：







### 重复消费

强校验bizNo查流水表幂等。

弱校验就存redis

### 顺序消费

RocketMQ提供了MessageQueueSelector选择机制。使用Hash取模法的那个就行了。让消息发送到同一个队列，在使用同步发送，前一个消息发送成功后，后一个消息擦能发送。保证发送有序，让同一个消费者消费。一般只有binlog日志同步才需要顺序消费，反正我业务场景遇不到。









### 消费者推模式还是拉模式



使用拉模式





# Zookeeper







