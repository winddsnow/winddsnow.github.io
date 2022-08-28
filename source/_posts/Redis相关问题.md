---
title: Redis相关问题
categories:
  - 编程中间件
tags:
  - Java
  - Redis
  - 缓存
date: 2022-08-28 20:17:49
---

# 简单介绍redis
1. Redis是 C 编写的，高性能非关系型数据库。
2. Redis 键只能为字符串，值支持五种数据类型：string、has、list、set、zset。
3. Redis 的数据存在内存中的，读写速度非常快， redis 被广泛应用于缓存方向，每秒可以处理超过 10万次读写操作，是已知性能最快的Key-Value DB。
4. 支持数据持久化，支持AOF（Append Only File）和RDB（Redis DataBase）两种持久化方式。
5. 支持主从复制，主机会自动将数据同步到从机，可以进行读写分离。
6. Redis 也经常用来做分布式锁和分布式事务。
7. 不能用作海量数据的高性能读写
8. Redis 不具备自动容错和恢复功能。主机从机的宕机都会导致前端部分读写请求失败，需要等待机器重启或者手动切换前端的IP才能恢复。
9. 主机宕机，宕机前有部分数据未能及时同步到从机，切换IP后还会引入数据不一致的问题，降低了系统的可用性。
10. Redis 较难支持在线扩容，在集群容量达到上限时在线扩容会变得很复杂。为避免这一问题，运维人员在系统上线时必须确保有足够的空间，这对资源造成了很大的浪费。

# redis存储的数据类型有哪些？
1. STRING，字符串，最简单的k-v存储，短信验证码，配置信息等，就用这种类型来存储。
2. HASH，包含键值对的无序散列表，一般key为ID或者其他唯一标识，value对应的就是详情了。如商品详情，个人信息详情，新闻详情等
3. LIST，列表，因为list是有序的，比较适合存储一些有序且数据相对固定的数据。如省市区表、字典表。List还可以做消息队列。
4. SET，无序集合，可以简单的理解为ID-List的模式，如微博中一个人有哪些好友，set最牛的地方在于，可以对两个set提供交集、并集、差集操作。例如：查找两个人共同的好友等。
5. ZSET，有序集合，set的增强版本，增加了一个score参数，自动会根据score的值进行排序。比较适合类似于top N等不根据插入的时间来排序的数据。

# Redis有哪些优缺点
1. 快，读的速度是110000次/s，写的速度是81000次/s
2. 结构丰富，支持5种，分别是string、has、list、set、zset。
3. 持久化，支持AOF（Append Only File）和RDB（Redis DataBase）
4. 分布式事务，Redis的事务实质上是命令的集合，在一个事务中要么所有命令都被执行，要么所有事物都不执行。
5. 分布式是锁，分布式锁是控制分布式系统之间同步访问共享资源的一种方式
6. 主从复制，主机会自动将数据同步到从机，可以实现读写分离。
7. 不能存海量数据，受到物理内存的限制
8. 不具备自动容错和恢复功能，主机从机的 宕机都会导致前端部分读写请求失败，需要等待机器重启或者手动切换前端的IP才能恢复。
9. 可能数据不一致，主机宕机，宕机前有部分数据未能及时同步到从机，切换IP后还会引入数据不一致的问题，降低了系统的可用性。
10. Redis 较难支持在线扩容，在集群容量达到上限时在线扩容会变得很复杂，为避免这一问题，上线时必须确保有足够的空间。

# redis的使用场景？
1. 缓存，将热点数据放到内存中。
2. 计数器，可以对 String 进行自增自减运算，从而实现计数器功能。
3. 队列，List 是一个双向链表，可以通过 lpush 和 rpop 写入和读取消息。不过最好使用 Kafka、RabbitMQ 等消息中间件。
4. 分布式锁，在分布式场景下，无法使用单机环境下的锁来对多个节点上的进程进行同步。可以使用 Redis 自带的 SETNX 命令实现分布式锁，除此之外，还可以使用官方提供的 RedLock 分布式锁实现。
5. 会话缓存，可以使用 Redis 来统一存储多台应用服务器的会话信息。
6. 全页缓存（FPC），除基本的会话token之外，Redis还提供很简便的FPC平台。没接触过。
7. 交集、差集、并集，Set 可以实现交集、差集、并集等操作，从而实现共同好友等功能。没接触过。
8. 排行榜，ZSet 可以实现有序性操作，从而实现排行榜等功能。没接触过。
9. 发布/订阅功能，用的少，没有MQ好。没接触过。

# redis的如何做事务支持
1. Redis的事务实质上是命令的集合，在一个事务中要么所有命令都被执行，要么所有事物都不执行。
2. 事务从开始到执行会经历以下三个阶段,MULTI 开始到 EXEC结束前，中间所有的命令都被加入到一个命令队列中；当执行 EXEC命令后，将QUEUE中所有的命令执行。也就是。
- MULTI开始事务。
- 命令入队列（QUEUE)。
- EXEC触发执行事务。
3. Redis的事务没有关系数据库事务提供的回滚（rollback），所以开发者必须在事务执行失败后进行后续的处理
- 如果在一个事务中的命令出现错误，那么所有的命令都不会执行；
- 如果在一个事务中出现运行错误，那么正确的命令会被执行。
4. 此外我们可以使用DISCARD取消事务。

# redis的持久化方式区别及其选择
1. 支持AOF（Append Only File）和RDB（Redis DataBase）两种持久化方式。
2. RDB是默认的持久化方式。按照一定的时间将内存的数据以快照的形式保存到硬盘中，对应产生的数据文件为dump.rdb。通过配置文件中的save参数来定义快照的周期。
3. AOF持久化(即Append Only File持久化)，则是将Redis执行的每次写命令记录到单独的日志文件中，当重启Redis会重新将持久化的日志中文件恢复数据。
4. RDB，安全性低。RDB 是间隔一段时间进行持久化，如果持久化之间 redis 发生故障，会发生数据丢失。
5. AOF，安全性高。即使中途服务器宕机，可以通过 redis-check-aof 工具解决数据一致性问题。
6. RDB，性能好，fork 子进程来完成写操作，让主进程继续处理命令，所以是 IO 最大化。
7. AOF比RDB更安全、更大，RDB比AOF性能更好。
8. 可以同时两种方式，两个都配了优先加载AOF。AOF粒度更细
9. 可以只使用RDB持久化，可以承受数分钟以内的数据丢失。
10. 不推荐只使用AOF持久化，因为定时生成RDB快照（snapshot）非常便于进行数据库备份， 并且 RDB 恢复数据集的速度也要比AOF恢复的速度要快，除此之外，使用RDB还可以避免AOF程序的bug
11. 不使用任何持久化方式，数据在服务器运行的时候存在

# redis的缓存击穿
1. redis缓存击穿是什么？
- 缓存击穿是指缓存中没有但数据库中有的数据（一般是缓存时间到期），这时由于并发用户特别多，同时读缓存没读到数据，又同时去数据库去取数据，引起数据库压力瞬间增大，造成过大压力。
- 关键字：缓存没有数据库有，并发多。
- 例子：活动系统里面查询活动信息，但是在活动进行过程中活动缓存突然过期了。
- 和缓存雪崩不同的是，缓存击穿指并发查同一条数据，缓存雪崩是不同数据都过期了，很多数据都查不到从而查数据库。
- 和缓存穿透不同的是，缓存穿透指缓存和数据库中都没有的数据。缓存击穿指的是缓存没有，数据库有

2. 解决方案
- 设置热点数据永远不过期。简单来讲，就是不设置过期时间。来了就给你。
- 加互斥锁(mutex key)。简单来讲，就是要操作db，需要排队。

# redis的缓存穿透
1. redis缓存穿透是什么？
- 缓存穿透是指缓存和数据库中都没有的数据，而用户不断发起请求
- 关键字：都没有
- 例子：如发起为id为“-1”的数据或id为特别大不存在的数据。这时的用户很可能是攻击者，攻击会导致数据库压力过大。
- 缓存穿透和缓存击穿容易混，缓存穿透强调的是都没有。缓存击穿值得是缓存没有，数据库有，

2. 解决方案
- 增加接口层校验，如用户鉴权校验，id做基础校验，id<=0的直接拦截；
- key-null存入缓存，从缓存取不到的数据，在数据库中也没有取到，这时也可以将key-value对写为key-null，缓存有效时间可以设置短点，如30秒（设置太长会导致正常情况也没法使用）。这样可以防止攻击用户反复用同一个id暴力攻击
- 采用布隆过滤器，将所有可能存在的数据哈希到一个足够大的 bitmap 中，一个一定不存在的数据会被这个 bitmap 拦截掉，从而避免了对底层存储系统的查询压力

# redis的缓存雪崩
1. redis缓存雪崩是什么？
- 在高并发下，大量的缓存key在同一时间失效，导致大量的请求落到数据库上，如活动系统里面同时进行着非常多的活动，但是在某个时间点所有的活动缓存全部过期。
- 关键字：大量key，都失效。
- 例子：如活动系统里面同时进行着非常多的活动，但是在某个时间点所有的活动缓存全部过期。

2. 解决方案
- 缓存数据的过期时间设置随机，防止同一时间大量数据过期现象发生。
- 一般并发量不是特别多的时候，使用最多的解决方案是加锁排队。
- 给每一个缓存数据增加相应的缓存标记，记录缓存的是否失效，如果缓存标记失效，则更新数据缓存。

# redis的缓存降级
1. 当访问量剧增、服务出现问题（如响应时间慢或不响应）或非核心服务影响到核心流程的性能时，仍然需要保证服务还是可用的，即使是有损服务。系统可以根据一些关键数据进行自动降级，也可以配置开关实现人工降级。
2. 缓存降级的最终目的是保证核心服务可用，即使是有损的。而且有些服务是无法降级的（如加入购物车、结算）。
3. 在进行降级之前要对系统进行梳理，看看系统是不是可以丢卒保帅；从而梳理出哪些必须誓死保护，哪些可降级；比如可以参考日志级别设置预案：
- 一般：比如有些服务偶尔因为网络抖动或者服务正在上线而超时，可以自动降级；
- 警告：有些服务在一段时间内成功率有波动（如在95~100%之间），可以自动降级或人工降级，并发送告警；
- 错误：比如可用率低于90%，或者数据库连接池被打爆了，或者访问量突然猛增到系统能承受的最大阀值，此时可以根据情况自动降级或者人工降级；
- 严重错误：比如因为特殊原因数据错误了，此时需要紧急人工降级。
4. 服务降级的目的，是为了防止Redis服务故障，导致数据库跟着一起发生雪崩问题。因此，对于不重要的缓存数据，可以采取服务降级策略，例如一个比较常见的做法就是，Redis出现问题，不去数据库查询，而是直接返回默认值给用户。

# redis的缓存预热
1. 缓存预热就是系统上线后，将相关的缓存数据直接加载到缓存系统。这样就可以避免在用户请求的时候，先查询数据库，然后再将数据缓存的问题！用户直接查询事先被预热的缓存数据！
2. 解决方案
- 直接写个缓存刷新页面，上线时手工操作一下；
- 数据量不大，可以在项目启动的时候自动进行加载；
- 定时刷新缓存；

# Redis的过期键的删除策略
1. Redis中同时使用了惰性过期和定期过期两种过期策略。
2. 常见有3种过期策略，定期过期，定时过期，惰性过期。
3. 惰性过期：只有当访问一个key时，才会判断该key是否已过期，过期则清除。该策略可以最大化地节省CPU资源，却对内存非常不友好。极端情况可能出现大量的过期key没有再次被访问，从而不会被清除，占用大量内存。
4. 定期过期：每隔一定的时间，会扫描一定数量的数据库的expires字典中一定数量的key，并清除其中已过期的key。
5. 定时过期：每个设置过期时间的key都需要创建一个定时器，到过期时间就会立即清除。该策略可以立即清除过期的数据，对内存很友好；但是会占用大量的CPU资源去处理过期的数据，从而影响缓存的响应时间和吞吐量。
6. 定期、定时，是主动删除。惰性，是被动删除。

# Redis缓存淘汰策略
1. Redis内存不足的缓存淘汰策略提供了8种。
- noeviction：当内存使用超过配置的时候会返回错误，不会驱逐任何键
- allkeys-lru：加入键的时候，如果过限，首先通过LRU算法驱逐最久没有使用的键
- volatile-lru：加入键的时候如果过限，首先从设置了过期时间的键集合中驱逐最久没有使用的键
- allkeys-random：加入键的时候如果过限，从所有key随机删除
- volatile-random：加入键的时候如果过限，从过期键的集合中随机驱逐
- volatile-ttl：从配置了过期时间的键中驱逐马上就要过期的键
- volatile-lfu：从所有配置了过期时间的键中驱逐使用频率最少的键
- allkeys-lfu：从所有键中驱逐使用频率最少的键
2. 这八种大体上可以分为4中，lru、lfu、random、ttl
- lru：Least Recently Used)，最近最少使用
- lfu：Least Frequently Used，最不经常使用法
- ttl：Time To Live，生存时间
- random：随机
3. 默认是noeviction。对于写请求不再提供服务，直接返回错误（DEL请求和部分特殊请求除外
4. eviction：“逐出；赶出；收回”
5. volatile：“不稳定的”

# redis主从复制（读写分离）
1. 主从复制就是我们常见的master/slave模式，通常一个主数据库可以有多个从数据库。
2. 可以实现读写分离，Master具有读写权限，Slave只有读权限。当写操作导致数据发生变化时会自动将数据同步给从数据库。从数据库是只读的，并接收主数据库同步过来的数据。
3. 可以实现容灾备份，Master出现故障，利用“sentinel监控”，把Slave其中一个提升为Master,让系统继续执行。
4. slave node主要用来进行横向扩容，读写分离，扩容的slave node可以提高读的吞吐量
5. 如何搭建
- slave node指定master node的ip和端口即可。
6. 主从复制原理是怎样的呢？（了解）
- Slave启动成功连接到master后会发送一个sync命令；
- Master接到命令启动后的存盘进程，同时收集所有接收到的用于修改数据集命令，在后台进程执行完毕之后，master，将传送整个数据文件到slave，以完成一次完全同步；
- 全量复制：而slave服务在数据库文件数据后，将其存盘并加载到内存中；
- 增量复制：Master继续将新的所有收集到的修改命令依次传给slave，完成同步；
- 但是只要是重新连接master，一次完全同步（全量复制）将被自动执行。
7. 主从复制的缺点？
- 延时，由于所有的写操作都是在Master上操作，然后同步更新到Slave上
8. redis高可用经常用两种方案，方案A（Redis+Cluster）和方案B（Redis+Sentinel）。方案B指的就是主从复制+哨兵模式。

# redis的RedLock（红锁）
1. Redlock是一种算法，Redlock也就是 Redis Distributed Lock，可用实现多节点redis的分布式锁。
2. RedLock官方推荐，Redisson完成了对Redlock算法封装。
3. 此种方式具有以下特性：
- 互斥访问：即永远只有一个 client 能拿到锁
- 避免死锁：最终 client 都可能拿到锁，不会出现死锁的情况，即使锁定资源的服务崩溃或者分区，仍然能释放锁。
- 容错性：只要大部分 Redis 节点存活（一半以上），就可以正常提供服务
4. RedLock原理（了解）
- 获取当前Unix时间，以毫秒为单位。
- 依次尝试从N个实例，使用相同的key和随机值获取锁。在步骤2，当向Redis设置锁时,客户端应该设置一个网络连接和响应超时时间，这个超时时间应该小于锁的失效时间。例如你的锁自动失效时间为10秒，则超时时间应该在5-50毫秒之间。这样可以避免服务器端Redis已经挂掉的情况下，客户端还在死死地等待响应结果。如果服务器端没有在规定时间内响应，客户端应该尽快尝试另外一个Redis实例。
- 客户端使用当前时间减去开始获取锁时间（步骤1记录的时间）就得到获取锁使用的时间。当且仅当从大多数（这里是3个节点）的Redis节点都取到锁，并且使用的时间小于锁失效时间时，锁才算获取成功。
- 如果取到了锁，key的真正有效时间等于有效时间减去获取锁所使用的时间（步骤3计算的结果）。
- 如果因为某些原因，获取锁失败（没有在至少N/2+1个Redis实例取到锁或者取锁时间已经超过了有效时间），客户端应该在所有的Redis实例上进行解锁（即便某些Redis实例根本就没有加锁成功）。

# redis的分布式锁怎么实现
1. Java中的锁，只能保证在同一个JVM进程内中执行。分布式集群需要分布式锁。
2. 分布式锁的实现有很多，比如基于数据库、memcached、Redis、系统文件、zookeeper等。
3. 针对单机Redis实例，也就是一个master
- 自己通过setnx获取锁，lua释放锁，或者通过set key value px milliseconds nx+lua（比setnx的方式更新）。
- 因为setnx没法设置过期时间，从 2.6.12 起，SET 涵盖了 SETEX 的功能，并且 SET 本身已经包含了设置过期时间的功能
- 或者通过封装好的redission框架。
4. 针对多个Redis实例，也就是N个Redis master
- 可以使用到RedLock（红锁）算法。
- 或者通过封装好的redission框架，redisson已经有对redlock算法封装。
5. RedLock原理（了解）
- 获取当前Unix时间，以毫秒为单位。
- 依次尝试从N个实例，使用相同的key和随机值获取锁。在步骤2，当向Redis设置锁时,客户端应该设置一个网络连接和响应超时时间，这个超时时间应该小于锁的失效时间。例如你的锁自动失效时间为10秒，则超时时间应该在5-50毫秒之间。这样可以避免服务器端Redis已经挂掉的情况下，客户端还在死死地等待响应结果。如果服务器端没有在规定时间内响应，客户端应该尽快尝试另外一个Redis实例。
- 客户端使用当前时间减去开始获取锁时间（步骤1记录的时间）就得到获取锁使用的时间。当且仅当从大多数（这里是3个节点）的Redis节点都取到锁，并且使用的时间小于锁失效时间时，锁才算获取成功。
- 如果取到了锁，key的真正有效时间等于有效时间减去获取锁所使用的时间（步骤3计算的结果）。
- 如果因为某些原因，获取锁失败（没有在至少N/2+1个Redis实例取到锁或者取锁时间已经超过了有效时间），客户端应该在所有的Redis实例上进行解锁（即便某些Redis实例根本就没有加锁成功）。

# redis的哨兵机制
1. Redis-Sentinel是Redis官方推荐的高可用性(HA)解决方案。
2. 为了解决在主从复制架构中出现宕机的情况。
3. Redis-Sentinel方案和Redis-Cluster方法，是redis搭建集群的常用两种方案。
4. Redis的哨兵(sentinel) 系统用于管理多个 Redis 服务器,该系统执行以下三个任务
- 监控(Monitoring): 哨兵(sentinel) 会不断地检查你的Master和Slave是否运作正常
- 提醒(Notification):当被监控的某个 Redis出现问题时, 哨兵(sentinel) 可以通过 API 向管理员或者其他应用程序发送通知。
- 自动故障迁移(Automatic failover):当一个Master不能正常工作时，哨兵(sentinel) 会开始一次自动故障迁移操作,它会将失效Master的其中一个Slave升级为新的Master, 并让失效Master的其他Slave改为复制新的Master; 当客户端试图连接失效的Master时,集群也会向客户端返回新Master的地址,使得集群可以使用Master代替失效Master。
5. 我们一般在一个架构中运行多个哨兵(sentinel) 。
6. Sentinel的工作原理（自己感兴趣可以看看）
- 每个Sentinel以每秒钟一次的频率向它所知的Master，Slave以及其他 Sentinel 实例发送一个 PING 命令。
- 如果一个实例（instance）距离最后一次有效回复 PING 命令的时间超过 down-after-milliseconds 选项所指定的值， 则这个实例会被 Sentinel 标记为主观下线。
- 如果一个Master被标记为主观下线，则正在监视这个Master的所有 Sentinel 要以每秒一次的频率确认Master的确进入了主观下线状态。
- 当有足够数量的 Sentinel（大于等于配置文件指定的值）在指定的时间范围内确认Master的确进入了主观下线状态， 则Master会被标记为客观下线 。
- 在一般情况下， 每个 Sentinel 会以每 10 秒一次的频率向它已知的所有Master，Slave发送 INFO 命令 。
- 当Master被 Sentinel 标记为客观下线时，Sentinel 向下线的 Master 的所有 Slave 发送 INFO 命令的频率会从 10 秒一次改为每秒一次 。
- 若没有足够数量的 Sentinel 同意 Master 已经下线， Master 的客观下线状态就会被移除。
- 若 Master 重新向 Sentinel 的 PING 命令返回有效回复， Master 的主观下线状态就会被移除。

简要回答
1. 哨兵机制是官方推荐的高可用性(HA)解决方案。一般配合主从复制使用。
2. 为了解决在主从复制架构中出现宕机的情况，它会将失效Master的其中一个Slave升级为新的Master
3. 实际过程中需要搭建哨兵集群。搭建过程比较简单。
4. Redis-Sentinel方案和Redis-Cluster方法，是redis搭建集群的常用两种方案。

# redis集群搭建
1. 可以有Redis-Sentinel、Redis-Cluster两种方案。
2. Redis-Sentinel(哨兵模式)是Redis官方推荐的高可用性(HA)解决方案，当用Redis做Master-slave的高可用方案时，假如master宕机了，Redis本身(包括它的很多客户端)都没有实现自动进行主备切换。
3. Redis-Cluster是Redis官方推荐的分布式集群解决方案，在Redis 3.0版本正式推出的，有效解决了Redis分布式方面的需求
4. 一组Redis Cluster是由多个Redis实例组成，官方推荐我们使用6实例，其中3个主节点，3个从结点
5. Redis Cluster结构特点（了解）
- 所有的redis节点彼此互联(PING-PONG机制),内部使用二进制协议优化传输速度和带宽。
- 节点的fail是通过集群中超过半数的节点检测失效时才生效。
- 客户端与redis节点直连,不需要中间proxy层。客户端不需要连接集群所有节点,连接集群中任何一个可用节点即可。
- redis-cluster把所有的物理节点映射到[0-16383]slot上（不一定是平均分配）,cluster 负责维护node<->slot<->value。
- Redis集群预分好16384个桶，当需要在 Redis 集群中放置一个 key-value 时，根据 CRC16(key) mod 16384的值，决定将一个key放到哪个桶中。
6. 两种方案怎么选择呢？Redis-Sentinel适合用于读多写少的场景，分布式环境用Redis-Cluste。

简要回答
1. 可以有Redis-Sentinel、Redis-Cluster两种方案。
2. Redis-Sentinel是主从模式+哨兵机制。
3. Redis-Cluster是Redis 3.0后推出的，也引入了主从，一般6个结点，3主3从。
4. Redis-Sentinel适合用于读多写少的场景。Redis-Cluster是分布式环境下推出的，适合分布式环境。

# 项目中用到redis的地方
1. 垃圾图片的定时清理。
- 主要利用set结构
- 正确流程是，上传图片，保存套餐，如果只做了上传，没有提交套餐，那么这个图片其实是垃圾图片。
- 上传图片成功，图片地址存入一个set。
- 保存套餐成功，图片地址存入一个set。
- 通过set取差集，找到垃圾图片，然后清理。
2. 短信验证码5分钟过期
- 主要利用String结构
- 生成验证码，存入redis，设置5分钟过期。
- 收到验证码，从redis取出，看是否超过5分钟。
3. 购物车数据存入redis
- 使用了Hash结构
- 某个用户，某一个商品的购物车信息存入redis，key:用户名 ，field:sku的ID， value:购物车数据(order_item)
4. 秒杀
- 使用了List结构，做队列
- 用到了Hash结构

## 垃圾图片的定时清理
1. 当用户上传图片后，将图片名称保存到redis的一个Set集合中，例如集合名称为packagePicResources
```java
@Autowired
 private JedisPool jedisPool;
 //图片上传
 @RequestMapping("/upload")
 public Result upload(@RequestParam("imgFile")MultipartFile imgFile){
   try{
     //获取原始文件名
     String originalFilename = imgFile.getOriginalFilename();
     int lastIndexOf = originalFilename.lastIndexOf(".");
     //获取文件后缀
     String suffix = originalFilename.substring(lastIndexOf - 1);
     //使用UUID随机产生文件名称，防止同名文件覆盖
     String fileName = UUID.randomUUID().toString() + suffix;
     QiniuUtils.upload2Qiniu(imgFile.getBytes(),fileName);
     //图片上传成功
     Result result = new Result(true, MessageConstant.PIC_UPLOAD_SUCCESS);
     result.setData(fileName);
     //将上传图片名称存入Redis，基于Redis的Set集合存储【看这里】
     jedisPool.getResource().sadd(RedisConstant.SETMEAL_PIC_RESOURCES,fileName);
     return result;
  }catch (Exception e){
     e.printStackTrace();
     //图片上传失败
     return new Result(false,MessageConstant.PIC_UPLOAD_FAIL);
  }
 }
```
2. 当用户添加套餐后，将图片名称保存到redis的另一个Set集合中，例如集合名称为packagePicDbResources
```java
@Autowired
 private JedisPool jedisPool;
 //新增套餐
 public void add(Package package, Integer[] checkgroupIds) {
   packageDao.add(package);
   if(checkgroupIds != null && checkgroupIds.length > 0){
     setPackageAndCheckGroup(package.getId(),checkgroupIds);
  }
   //将图片名称保存到Redis
   savePic2Redis(package.getImg());
 }
 //将图片名称保存到Redis
 private void savePic2Redis(String pic){
   //【看这里】
   jedisPool.getResource().sadd(RedisConstant.SETMEAL_PIC_DB_RESOURCES,pic);
 }
```
3. 计算packagePicResources集合与packagePicDbResources集合的差值，结果就是垃圾图片的名称集合，清理这些图片即可
```java
public void doJob() {
    //计算packagePicResources集合与packagePicDbResources集合的差值
    //【看这里】
         Set<String> picNeed2Delete = jedisPool.getResource().sdiff(RedisConstant.SETMEAL_PIC_RESOURCES, RedisConstant.SETMEAL_PIC_DB_RESOURCES);
         if (null != picNeed2Delete && picNeed2Delete.size() > 0) {
             String[] picNeed2DeleteArr = picNeed2Delete.toArray(new String[]{});
             QiNiuUtil.removeFiles(picNeed2DeleteArr);
             // 清理这些图片即可
             //【看这里】
             jedisPool.getResource().srem(RedisConstant.SETMEAL_PIC_RESOURCES, picNeed2DeleteArr);
        }
    }
```

## 短信验证码5分钟过期
```java
 /**
      * 发送手机验证码
      * @param telephone
      * @return
      */
     @PostMapping("/send4Login")
     public Result send4Login(String telephone){
         // 生成验证
         Integer code = ValidateCodeUtils.generateValidateCode(6);
         // 发送
         try {
             // 判断验证码是否发送过了
             String key = RedisMessageConstant.SENDTYPE_LOGIN + "_" + telephone;
             if(null != jedisPool.getResource().get(key)){
                 return new Result(false, "验证已经发送过了，请注意查收");
            }
             SMSUtils.sendShortMessage(SMSUtils.VALIDATE_CODE, telephone, code + "");
             // 写入redis中 setex方法：让redis中的key值在过了有效期就自动清除
             // 验证码的存活时间 5 10
             // String key redis中的key, int seconds 有效时间, String value 值
             //【看这里】
             jedisPool.getResource().setex(key, 5*60, code+"");
 
             return new Result(true, MessageConstant.SEND_VALIDATECODE_SUCCESS);
        } catch (ClientException e) {
             e.printStackTrace();
        }
         return new Result(false, MessageConstant.SEND_VALIDATECODE_FAIL);
    }
```
**验证验证码**
```java
@PostMapping("/check")
     public Result loginCheck(@RequestBody Map<String,String> map, HttpServletResponse res){
         // 从redis获取验证码
         String telephone = map.get("telephone");
         String key = RedisMessageConstant.SENDTYPE_LOGIN + "_" + telephone;
          //【看这里】
         String codeInRedis = jedisPool.getResource().get(key);
         if(null == codeInRedis){
             return new Result(false, "验证码超时，请重新获取");
        }
         // 前端传过来的验证码
         String code = map.get("validateCode");
         if(!codeInRedis.equals(code)){
             return new Result(false, MessageConstant.VALIDATECODE_ERROR);
        }
         // 判断 是否为会员
         // 通过手机号码查询会员表
         Member member = memberService.findByTelephone(telephone);
         if(null == member){
             // 不是会员, 注册会员，由用户登陆后再完善
             member = new Member();
             member.setPhoneNumber(telephone);
             member.setRegTime(new Date());
             memberService.add(member);
        }
         // 用户跟踪，写手机号码到Cookie
         Cookie cookie = new Cookie("login_member_telephone",telephone);
         cookie.setMaxAge(60*60*24*30); // 有效期秒
         cookie.setPath("/");
         res.addCookie(cookie);
 
         return new Result(true, MessageConstant.LOGIN_SUCCESS);
  }
```

## 购物车数据存入redis
从redis里面删除和添加到redis
```java
@Override
     public void add(Long id, Integer num, String username) {
         if(num<=0){
             //删除掉原来的商品
             //【看这里】
             redisTemplate.boundHashOps("Cart_" + username).delete(id);
             return;
        }
         //1.根据商品的SKU的ID 获取sku的数据
         Result<Sku> skuResult = skuFeign.findById(id);
         Sku data = skuResult.getData();
         if (data != null) {
             //2.根据sku的数据对象 获取 该SKU对应的SPU的数据
             Long spuId = data.getSpuId();
             Result<Spu> spuResult = spuFeign.findById(spuId);
             Spu spu = spuResult.getData();
             //3.将数据存储到 购物车对象(order_item)中
             OrderItem orderItem = new OrderItem();
             orderItem.setCategoryId1(spu.getCategory1Id());
             orderItem.setCategoryId2(spu.getCategory2Id());
             orderItem.setCategoryId3(spu.getCategory3Id());
             orderItem.setSpuId(spu.getId());
             orderItem.setSkuId(id);
             orderItem.setName(data.getName());//商品的名称 sku的名称
             orderItem.setPrice(data.getPrice());//sku的单价
             orderItem.setNum(num);//购买的数量
             orderItem.setMoney(orderItem.getNum() * orderItem.getPrice());//单价* 数量
             orderItem.setPayMoney(orderItem.getNum() * orderItem.getPrice());//单价* 数量
             orderItem.setImage(data.getImage());//商品的图片dizhi
              //【看这里】
             //4.数据添加到redis中 key:用户名 field:sku的ID value:购物车数据(order_item)
             redisTemplate.boundHashOps("Cart_" + username).put(id, orderItem);// hset key field value   hget key field
        }
    }
```
**取出购物车里面的数据**
```java
@Override
     public List<OrderItem> list(String username) {
         List<OrderItem> orderItemList = redisTemplate.boundHashOps("Cart_" + username).values();
         return orderItemList;
 }
```

