# Redis

**基本特性：**
**基于键值对、读写速度快、数据结构丰富、简单稳定、支持持久化、支持事务（原子性）、主从复制、高可用与分布式。**

- key-value 存储系统，是跨平台的非关系型数据库 (NoSql)
- Redis 是一个开源的使用 ANSI C 语言编写、遵守 BSD 协议、支持网络、可基于内存、分布式、可选持久性的键值对(Key-Value)存储数据库，并提供多种语言的 API。
- Redis是基于C语言实现的，一般来说C语言距离操作系统更近，执行速度相对更快
- Redis 通常被称为数据结构服务器，因为值（value）可以是字符串(String)、哈希(Hash)、列表(list)、集合(sets)和有序集合(sorted sets)等类型。
- Redis采用了单线程架构，避免了不必要的上下文切换和竞争条件，线程安全
- 使用多路 I/O 复用模型，非阻塞 IO；

## 优势:

- 高性能： 访问内存速度远远快于读取用硬盘,  访问数据库中的某些数据,是从硬盘上读取的。操作缓存就是直接操作内存，所以**速度相当快**。
- 高并发：直接操作缓存能够承受的请求是远远大于直接访问数据库的
  

## 使用场景

#### 作为缓存

-  优势: 减少数据库的访问频率。 提高数据的访问率。

- 什么数据合适作为缓存: 

  1. 热点数据 

  2. 修改频率比较低
  3. 安全系数低的

#### 数据共享分布式

String 类型，因为 Redis 是分布式的独立服务，可以在多个应用之间共享

#### 分布式锁

> 利用redis单线程的的特性,实现分布式锁

- `EX`：设置键的过期时间（单位为秒）
- `PX`：设置键的过期时间（单位为毫秒）
- `NX` ：只在键不存在时，才对键进行设置操作。 `SET key value NX` 效果等同于 `SETNX key value` 。
- `XX` ：只在键已经存在时，才对键进行设置操作。

- setnx

  > 有在 key 不存在时设置 key 的值。返回 1和0

- Expire 

  > 设置过期时间 单位second

```java

public static boolean getLock(String key) {
    Long flag = jedis.setnx(key, "1");
    if (flag == 1) {
        jedis.expire(key, 10);
    }
    return flag == 1;
}
 
public static void releaseLock(String key) {
    jedis.del(key);
}
```



 ####  消息队列

> 有序

三种方案:

- PUB/SUB，订阅/发布模式
- 基于List的 LPUSH+BRPOP 的实现
- 基于Sorted-Set的实现
- Redis5.0的Stream类型

要求: 

- 消息保序：对应消息需要有序消费的场景
- 处理重复消息：如网络抖动引起的同一条消息多次被投递到队列的场景
- 保证消息可靠性：消息从队列取出，此时客户端宕机，消息未正常消费的场景

#### 计数器

> int类型，incr方法

- incr

  > Incr 命令将 key 中储存的数字值增一。
  >
  > 如果 key 不存在，那么 key 的值会先被初始化为 0 ，然后再执行 INCR 操作

例如：文章的阅读量、微博点赞数、允许一定的延迟，先写入Redis再定时同步到数据库

#### 限流

> int类型，incr方法

以访问者的ip和其他信息作为key，访问一次增加一次计数，超过次数则返回false





## Redis支持的数据类型

Redis支持五种数据类型：

- string（字符串）

- hash（哈希）

- list（列表）

- set（集合）

- zset(sorted set：有序集合)

### String

字符串类型是Redis最基础的数据结构（key-value 类型），其它的几种数据结构都是在字符串类型基础上构建的，字符串的值可以是：字符串、数字、二进制，但其值最大不能超过512M。

- 特性：二进制安全的，计算字符串长度高效、追加字符串高效

使用场景： 缓存、计数器、对象存储缓存（共享session）、限速



### **Hash**

Redis hash 是一个 string 类型的 field（字段） 和 value（值） 的映射表，hash 特别适合用于存储对象。

`value={{field1,value1},{field2,value2}…}`

应用场景： 用户信息缓存



### **List**

Redis列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部或者尾部，也可以获取指定范围指定下标的元素等。一个列表最多可以包含 232 - 1 个元素 (4294967295， 每个列表超过40亿个元素)。

两个特点：

- 有序, 可以通过索引下标获取某个元素霍某个某个范围内的元素列表
- 可重复

使用场景： 消息队列、栈、文章列表等。 时间线

### **Set**

Redis的Set是String类型的无序集合，我们不能通过索引获取元素。集合成员是唯一的，这就意味着集合中不能出现重复的数据。Redis中集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是O(1)。集合中最大的成员数为 232 - 1 (4294967295, 每个集合可存储40多亿个成员)。

- 无序
- 不重复

应用场景： 标签（tag）



### Zset

在有序集合中保留了不能有重复成员的特性，但其中的成员是可以排序的，每一个元素都会关联一个double类型的分数（score）作为排序依据，score相同时按字典顺序排序。redis正是通过分数来为集合中的成员进行从小到大的排序。

- 有序
- 不重复

应用场景： 排行榜系统，成绩单，工资表





## 发布和订阅机制(pub/sub)

- 发布者(publisher)

  一个客户端通过` PUBLISH `命令向订阅者发送信息

- 订阅者(subscriber)

  一个客户端使用 `SUBSCRIBE` 或者` PSUBSCRIBE`命令接收信息

- channel (频道)   -中间介

  发布者将信息直接发布给 channel ，而 channel 负责将信息发送给适当的订阅者

触发过程:

1. 订阅通道  `/redis/pubsub/sub?key=chanel`  

2. 发布通道(key)和消息(value) `/redis/pubsub/pub?key=chanel&value=test`  

3. 发布后,将触发执行,订阅写的回调函数

   ```java
       /**
        * 发布消息
        *
        * @param key   通道Key
        * @param value 发送内容
        */
       @GetMapping("/pub")
       public R<Void> pub(String key, String value) {
           RedisUtils.publish(key, value, consumer -> {
               System.out.println("发布通道 => " + key + ", 发送值 => " + value);
           });
           return R.ok("操作成功");
       }
   
       /**
        * 订阅消息
        *
        * @param key 通道Key
        */
       @GetMapping("/sub")
       public R<Void> sub(String key) {
           RedisUtils.subscribe(key, String.class, msg -> {
               //只有在订阅的通道发布消息后才触发
               System.out.println("订阅通道 => " + key + ", 接收值 => " + msg);
           });
           return R.ok("操作成功");
       }
   ```

   

一些局限性:

1. 发送者发送的消息无法持久化，所以可能会造成消息丢失
2. 由于消息无法持久化，所以，消费者无法收到在订阅channel之前发送的消息
3. 发送者与客户端之间的消息发送与接收不存在 ACK 机制



应用场景:

> 由于没有消息持久化与 ACK 的保证，Redis 的发布订阅功能并不可靠
>
> 用于实时与可靠性要求不高的场景

- 消息推送





## Redis消息队列和KafKa优劣对比

> Kafka是一个完整的系统，而Redis PUB/SUB只是一个套件(utility)

### Redis

- 即时消费场景,快产快消 ,生产的消息立即被消费者消费掉
- 速度十分看重的,比如慢了一秒好几千万这种
- 允许出现消息丢失的场景(断电就清空)
- 不需要持久化(有持久化,但非完全可靠不会丢)
- 需要处理的数据量并不是那么巨大 ,小于big size(10K+的文件)



### KafKa

> 支持分布式,高可用的部署,并且对一个大的队列采用分成多个`Partition`(分区),来提高消息入队的吞吐量,分而治之的思想. 并且消费的时候支持`group`的概念,能够支持多个客户端消费同个队列,并且一个`group`中可以增加Consumer的数量来扩展消费的处理量.
>
> KafKa不受生产者数量的影响,因为吞吐量足够支撑,即使在廉价的单机服务器上也可以有**10万每秒**的消息传输量,并且消费者是想什么时候消费都可以,消息它就在那里,十分灵活,不用担心来无影去无踪的恐慌.能把消息持久化,并以一定的策略(例如一定时间内删除,或者到达多大容量的时候清空)

- 稳定的消息队列
- 发送过的消息可以保留一定的时间,有无迹可寻
- 无法忍受数据的丢失
- 速度不需要那么的快
- 需要处理数据量巨大(高吞吐量)
- 多样化的消费处理模型