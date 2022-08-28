---
title: Redis分布式缓存
categories:
  - 编程中间件
tags:
  - Java
  - Redis
  - 缓存
date: 2022-08-28 08:52:56
---

# 0. 基于Redis集群解决单机Redis存在的问题
单机的Redis存在四大问题：
1. 数据丢失问题--解决：实现redis数据持久化
2. 并发能力问题--解决：搭建主从集群，实现读写分离
3. 故障恢复问题--解决：利用redis哨兵，实现健康检测和自动恢复
4. 存储能力问题--解决：搭建分片集群，利用插槽机制实现动态扩容

# 1. Redis持久化

Redis有两种持久化方案：

- RDB持久化
- AOF持久化



## 1.1 RDB持久化

RDB全称Redis Database Backup file（Redis数据备份文件），也被叫做Redis数据快照。简单来说就是把内存中的所有数据都记录到磁盘中。当Redis实例故障重启后，从磁盘读取快照文件，恢复数据。快照文件称为RDB文件，默认是保存在当前运行目录。

### 1.1.1 执行时机

RDB持久化在四种情况下会执行：

- 执行save命令
- 执行bgsave命令
- Redis停机时
- 触发RDB条件时

**1）save命令**
save命令会导致主进程执行RDB，这个过程中其它所有命令都会被阻塞。只有在数据迁移时可能用到。由Redis主进程来执行RDB，会阻塞所有命令

**2）bgsave命令**
这个命令执行后会开启独立进程完成RDB，主进程可以持续处理用户请求，不受影响。
开启子进程执行RDB，避免主进程受到影响

**3）停机时**

Redis停机时会执行一次save命令，实现RDB持久化。

**4）触发RDB条件**

Redis内部有触发RDB的机制，可以在redis.conf文件中找到，格式如下：

```properties
# 900秒内，如果至少有1个key被修改，则执行bgsave ， 如果是save "" 则表示禁用RDB
save 900 1  
save 300 10  
save 60 10000 
```

RDB的其它配置也可以在redis.conf文件中设置：

```properties
# 是否压缩 ,建议不开启，压缩也会消耗cpu，磁盘很便宜，为了性能，一般不开启
rdbcompression yes

# RDB文件名称
dbfilename dump.rdb  

# 文件保存的路径目录
dir ./ 
```

### 1.1.2 RDB-bgsave保存原理
bgsave开始时会fork主进程得到子进程，子进程共享主进程的内存数据。完成fork后读取内存数据并写入 RDB 文件。
fork采用的是copy-on-write技术：
- 当主进程执行读操作时，访问共享内存；
- 当主进程执行写操作时，则会拷贝一份数据，执行写操作。

RDB的缺点？
- RDB执行间隔时间长，两次RDB之间写入数据有丢失的风险
- fork子进程、压缩、写出RDB文件都比较耗时

## 1.2 AOF持久化
### 1.2.1 AOF原理
AOF全称为Append Only File（追加文件）。Redis处理的每一个写命令都会记录在AOF文件，可以看做是命令日志文件。
### 1.2.2 AOF配置
AOF默认是关闭的，需要修改redis.conf配置文件来开启AOF：
```properties
# 是否开启AOF功能，默认是no
appendonly yes
# AOF文件的名称
appendfilename "appendonly.aof"
```

AOF的命令记录的频率也可以通过redis.conf文件来配：

```properties
# 表示每执行一次写命令，立即记录到AOF文件，同步刷盘，可靠性高，性能差，
appendfsync always 
# 写命令执行完先放入AOF缓冲区，然后表示每隔1秒将缓冲区数据写到AOF文件，是默认方案，适中
appendfsync everysec 
# 写命令执行完先放入AOF缓冲区，由操作系统决定何时将缓冲区内容写回磁盘，性能好，可靠性差
appendfsync no
```

### 1.2.3 AOF文件重写

因为是记录命令，AOF文件会比RDB文件大的多。而且AOF会记录对同一个key的多次写操作，但只有最后一次写操作才有意义。通过执行bgrewriteaof命令，可以让AOF文件执行重写功能，用最少的命令达到相同效果。

Redis也会在触发阈值时自动去重写AOF文件。阈值也可以在redis.conf中配置：
```properties
# AOF文件比上次文件 增长超过多少百分比则触发重写
auto-aof-rewrite-percentage 100
# AOF文件体积最小多大以上才触发重写 
auto-aof-rewrite-min-size 64mb 
```

## 1.3.RDB与AOF对比
RDB和AOF各有自己的优缺点，如果对数据安全性要求较高，在实际开发中往往会**结合**两者来使用。
|   |RDB|APF|
|---|---|---|
|持久化方式|定时对整个内存做快照|记录每一次执行的命令|
|数据完整性|不完整，两次备份之间会丢失|相对完整，取决于刷盘策略|
|文件大小|会有压缩，文件体积小|记录命令，文件体积很大|
|宕机恢复速度|很快|慢|
|数据恢复优先级|低，因为数据完整性不如AOF|高，因为数据完整性更高|
|系统资源占用|高，大量CPU和内存消耗|低，主要是磁盘IO资源，但AOF重写时会占用大量CPU和内存资源|
|使用场景|可以容忍数分钟的数据丢失，追求更快的启动速度|对数据安全性要求较高|

# 2. Redis主从相关
## 2.1 主从数据同步原理
### 2.1.1 全量同步
主从第一次建立连接时，会执行**全量同步**，将master节点的所有数据都拷贝给slave节点，流程：
1. slave执行replicaof(老版本中使用slaveof命令)后与主实例master进行连接并与主实例master同步数据。 
	1.1 mastermaster判断发现slave发送来的replid与自己的是否一致，如果是第一次，返回master自己的replid和offset
	1.2 slave保存版本信息
2. master执行bgsave，生成RDB文件并发送给slave，在生成RDB文件之后的命令用repl_baklog文件记录下来，
	2.1 slave清空本地数据，加载master发送过来的RDB文件
3. master发送repl_baklog文件中的命令给slave，slave执行接收到的repl_baklog文件中的命令

- **Replication Id**：简称replid，是数据集的标记，id一致则说明是同一数据集。每一个master都有唯一的replid，slave则会继承master节点的replid
- **offset**：偏移量，随着记录在repl_baklog中的数据增多而逐渐增大。slave完成同步时也会记录当前同步的offset。如果slave的offset小于master的offset，说明slave数据落后于master，需要更新。

master如何得知salve是第一次来连接呢？
因此slave做数据同步，必须向master声明自己的replication id 和offset，master才可以判断到底需要同步哪些数据。
因为slave原本也是一个master，有自己的replid和offset，当第一次变成slave，与master建立连接时，发送的replid和offset是自己的replid和offset。
master判断发现slave发送来的replid与自己的不一致，说明这是一个全新的slave，就知道要做全量同步了。
master会将自己的replid和offset都发送给这个slave，slave保存这些信息。以后slave的replid就与master一致了。
因此，**master判断一个节点是否是第一次同步的依据，就是看replid是否一致**。

- 完整流程描述：
1. slave节点请求增量同步
2. master节点判断replid，发现不一致，拒绝增量同步
3. master将完整内存数据生成RDB，发送RDB到slave
4. slave清空本地数据，加载master的RDB
5. master将RDB期间的命令记录在repl_baklog，并持续将log中的命令发送给slave
6. slave执行接收到的命令，保持与master之间的同步

### 2.2.2 增量同步
全量同步需要先做RDB，然后将RDB文件通过网络传输个slave，成本太高了。因此除了第一次做全量同步，其它大多数时候slave与master都是做**增量同步**。
什么是增量同步？就是只更新slave与master存在差异的部分数据。
1.  slave执行psync携带replid和offset与主实例master同步数据。master判断请求replid是否一致，如果不一致，直接拒绝
2.  如果一致则去repl_baklog中获取offset后的数据，而后发送offset后的命令，slave收到命令后执行命令

- master怎么知道slave与自己的数据差异在哪里呢?
这就要说到全量同步时的repl_baklog文件了。这个文件是一个固定大小的数组，只不过数组是环形，也就是说**角标到达数组末尾后，会再次从0开始读写**，这样数组头部的数据就会被覆盖。repl_baklog中会记录Redis处理过的命令日志及offset，包括master当前的offset，和slave已经拷贝到的offset：
repl_baklog大小有上限，写满后会覆盖最早的数据，如果slave断开时间过久，导致尚未备份的数据被覆盖，则无法基于log做增量同步，只能再次全量同步。


## 2.3 主从同步优化
- 在master中配置repl-diskless-sync yes启用无磁盘复制，避免全量同步时的磁盘IO。
- Redis单节点上的内存占用不要太大，减少RDB导致的过多磁盘IO
- 适当提高repl_baklog的大小，发现slave宕机时尽快实现故障恢复，尽可能避免全量同步
- 限制一个master上的slave节点数量，如果实在是太多slave，则可以采用主-从-从链式结构，减少master压力

简述全量同步和增量同步区别？
- 全量同步：master将完整内存数据生成RDB，发送RDB到slave。后续命令则记录在repl_baklog，逐个发送给slave。
- 增量同步：slave提交自己的offset到master，master获取repl_baklog中从offset之后的命令给slave

什么时候执行全量同步？
- slave节点第一次连接master节点时
- slave节点断开时间太久，repl_baklog中的offset已经被覆盖时

什么时候执行增量同步？
- slave节点断开又恢复，并且在repl_baklog中能找到offset时

# 3.Redis哨兵（Sentinel）
### 哨兵的作用如下：
- **监控**：Sentinel 会不断检查您的master和slave是否按预期工作
- **自动故障恢复**：如果master故障，Sentinel会将一个slave提升为master。当故障实例恢复后也以新的master为主
- **通知**：Sentinel充当Redis客户端的服务发现来源，当集群发生故障转移时，会将最新信息推送给Redis的客户端

### 集群监控原理
- Sentinel基于心跳机制监测服务状态，每隔1秒向集群的每个实例发送ping命令：
- 主观下线：如果某sentinel节点发现某实例未在规定时间响应，则认为该实例**主观下线**。
- 客观下线：若超过指定数量（quorum）的sentinel都认为该实例主观下线，则该实例**客观下线**。quorum值最好超过Sentinel实例数量的一半。

### 集群故障恢复原理
一旦发现master故障，sentinel需要在salve中选择一个作为新的master，选择依据是这样的：
- 首先会判断slave节点与master节点断开时间长短，如果超过指定值（down-after-milliseconds * 10）则会排除该slave节点
- 然后判断slave节点的slave-priority值，越小优先级越高，如果是0则永不参与选举
- 如果slave-prority一样，则判断slave节点的offset值，越大说明数据越新，优先级越高
- 最后是判断slave节点的运行id大小，越小优先级越高。

### 当选出一个新的master后，该如何实现切换呢？
- sentinel给备选的slave1节点发送slaveof no one命令，让该节点成为master
- sentinel给所有其它slave发送slaveof 192.168.150.101 7002 命令，让这些slave成为新master的从节点，开始从新的master上同步数据。
- 最后，sentinel将故障节点标记为slave，当故障节点恢复后会自动成为新的master的slave节点

### Sentinel的三个作用是什么？
- 监控
- 故障转移
- 通知

### Sentinel如何判断一个redis实例是否健康？
- 每隔1秒发送一次ping命令，如果超过一定时间没有相向则认为是主观下线
- 如果大多数sentinel都认为实例主观下线，则判定服务下线

### 故障转移步骤有哪些？
- 首先选定一个slave作为新的master，执行slaveof no one
- 然后让所有节点都执行slaveof 新master
- 修改故障节点配置，添加slaveof 新master

## 3.1 RedisTemplate
在Sentinel集群监管下的Redis主从集群，其节点会因为自动故障转移而发生变化，Redis的客户端必须感知这种变化，及时更新连接信息。Spring的RedisTemplate底层利用lettuce实现了节点的感知和自动切换。

### 引入依赖
在项目的pom文件中引入依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```
### 配置Redis地址
然后在配置文件application.yml中指定redis的sentinel相关信息：

```yaml
spring:
  redis:
    sentinel:
      master: mymaster
      nodes:
        - 192.168.150.101:27001
        - 192.168.150.101:27002
        - 192.168.150.101:27003
```
### 配置读写分离

在项目的启动类中，添加一个新的bean：

```java
@Bean
public LettuceClientConfigurationBuilderCustomizer clientConfigurationBuilderCustomizer(){
    return clientConfigurationBuilder -> clientConfigurationBuilder.readFrom(ReadFrom.REPLICA_PREFERRED);
}
```

这个bean中配置的就是读写策略，包括四种：
- MASTER：从主节点读取
- MASTER_PREFERRED：优先从master节点读取，master不可用才读取replica
- REPLICA：从slave（replica）节点读取
- REPLICA \_PREFERRED：先从slave（replica）节点读取，所有的slave都不可用才读取master

# 4. Redis分片集群（Redis Cluster）
### 分片集群特征：
- 集群中有多个master，每个master保存不同数据
- 每个master都可以有多个slave节点
- master之间通过ping监测彼此健康状态
- 客户端请求可以访问集群任意节点，最终都会被转发到正确节点

## 4.1 散列插槽
### 插槽原理
Redis会把每一个master节点映射到0~16383共16384个插槽（hash slot）上，数据key不是与节点绑定，而是与插槽绑定。redis会根据key的有效部分计算插槽值，计算方式是利用CRC16算法得到一个hash值，然后对16384取余，得到的结果就是slot插槽值。分两种情况：
- key中包含"{}"，且“{}”中至少包含1个字符，“{}”中的部分是有效部分
- key中不包含“{}”，整个key都是有效部分

### Redis如何判断某个key应该在哪个实例？
- 将16384个插槽分配到不同的实例
- 根据key的有效部分计算哈希值，对16384取余
- 余数作为插槽，寻找插槽所在实例即可

### 如何将同一类数据固定的保存在同一个Redis实例？
- 这一类数据使用相同的有效部分，例如key都以{typeId}为前缀

### 为什么是16384个插槽？
1. 如果槽位为65536，发送心跳信息的消息头达8k，发送的心跳包过于庞大。
如上所述，在消息头中，最占空间的是 myslots[CLUSTER_SLOTS/8]。
当槽位为65536时，这块的大小是:
65536÷8÷1024=8kb
2. 因为每秒钟，redis节点需要发送一定数量的ping消息作为心跳包，如果槽位为65536，这个ping消息的消息头太大了，浪费带宽。
redis的集群主节点数量基本不可能超过1000个。
如上所述，集群节点越多，心跳包的消息体内携带的数据越多。如果节点过1000个，也会导致网络拥堵。因此redis作者，不建议redis cluster节点数量超过1000个。
那么，对于节点数在1000以内的redis cluster集群，16384个槽位够用了。没有必要拓展到65536个。
3. 槽位越小，节点少的情况下，压缩比高
Redis主节点的配置信息中，它所负责的哈希槽是通过一张bitmap的形式来保存的，在传输过程中，会对bitmap进行压缩，但是如果bitmap的填充率slots / N很高的话(N表示节点数)，bitmap的压缩率就很低。
如果节点数很少，而哈希槽数量很多的话，bitmap的压缩率就很低。
ps：文件压缩率指的是，文件压缩前后的大小比。
综上所述，作者决定取16384个槽，不多不少，刚刚好！

## 4.2 集群伸缩
命令：redis-cli --cluster add-node
执行命令：
```sh
redis-cli --cluster add-node  192.168.150.101:7004 192.168.150.101:7001
```
调整插槽：redis-cli --cluster reshard 192.168.150.101:7001
后续会有选项提示
- all：代表全部，也就是三个节点各转移一部分
- 具体的id：目标节点的id
- done：没有了

## 4.3 RedisTemplate访问分片集群
RedisTemplate底层同样基于lettuce实现了分片集群的支持，而使用的步骤与哨兵模式基本一致：
1.引入redis的starter依赖
2.配置分片集群地址
3.配置读写分离
与哨兵模式相比，其中只有分片集群的配置方式略有差异，如下：

```yaml
spring:
  redis:
    cluster:
      nodes:
        - 192.168.150.101:7001
        - 192.168.150.101:7002
        - 192.168.150.101:7003
        - 192.168.150.101:8001
        - 192.168.150.101:8002
        - 192.168.150.101:8003
```

