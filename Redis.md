# Redis

> 1 单机MySQL的年代

网站多数用的静态HTML，建设网站的瓶颈

1. 数据量过大时，一个机器放不下
2. 数据的索引(B+Tree)，一个机器内存也放不下
3. 访问量（读写混合），一个服务器承受不了

> 2 Memcached(缓存)+MySQL+垂直拆分(读写分离)

网站80%情况下都是在读，每次都要去查询数据库的话就很麻烦，为了减轻读数据的压力，可以使用缓存来保证效率！  

发展过程：优化数据结构和索引-->文件缓存（IO）-->Memcached

> 3 分库分表+水平拆分+MySQL集群

慢慢的使用分库分表解决写的压力

## NoSQL

NoSQL=Not only SQL（不仅仅是SQL）泛指非关系型数据库

> NoSQL特点

1. 方便扩展（数据之间没有关系，很好扩展）
2. 大数据量高性能（Redis一秒写8万次，读取11万，NoSQL的缓存记录级，是一种细粒度的缓存，性能较高）
3. 数据类型是多样型的，不需要事先设计数据库，随取随用
4. 传统RDBMS和NoSQL![1615625704612](C:\Users\wjt\AppData\Roaming\Typora\typora-user-images\1615625704612.png)



![1615626456103](C:\Users\wjt\AppData\Roaming\Typora\typora-user-images\1615626456103.png)



## Redis入门

> Redis是什么

Redis(Remote Dictionary Server)，远程字典服务。是一个开源的使用ANSI C语言编写。支持网络，可基于内存亦可持久化的日志型。可用作数据库，缓存和消息中间件MQ，Key-value数据库，并提供多种语言的API。

> Redis能干嘛

1. 内存存储，持久化，可以将内存中的数据保存在磁盘中。内存中是断电即失，所以说持久化很重要
2. 效率高，可用于高速缓存
3. 支持数据的备份，即master-slave模式的数据备份
4. 发布订阅系统，地图信息分析
5. 计数器（浏览量），计时器

> 数据类型

五大基本数据类型：String   List   Set   Hash   Zset   三种特殊数据类型：geo   hyperloglog    bitMap

### 基础知识

Redis有16个数据库，默认使用的是第一个（0号数据库），可以使用select进行切换

``` bash
127.0.0.1:6379> select 3  #选择第三个数据库
OK
127.0.0.1:6379[3]> dbsize  # 查看数据库大小
(integer 0)
127.0.0.1:6379[3]> keys *  #查看当前数据库所有的key
127.0.0.1:6379[3]> flushdb #清空当前数据库的key
127.0.0.1:6379[3]> flushall #清空所有数据库的key
127.0.0.1:6379[3]> move key db  #移除db数据库下的某个key
127.0.0.1:6379[3]> expire key time  #设置key即将过期的时间，单位是秒
127.0.0.1:6379[3]> ttl key   #查看当前key还有多少时间过期
127.0.0.1:6379[3]> exists key   #判断key是否存在 1：存在   0：不存在
127.0.0.1:6379[3]> type key  #查看当前key的类型
```

> 思考：为啥redis默认端口号是6379

6379是`MERZ`九宫格输入法对应的数字，`Alessia Merz`一位意大利舞女（粉丝效应）

> Redis是单线程的

Redis很快，官方表示：Redis是基于内存操作，CPU不是Redis性能瓶颈，他的瓶颈是根据机器的内存和网络的带宽

**Redis为什么单线程还这么快**

1. Redis是c语言写的，官方提供的数据是10万+的QPS，完全不比同样是使用`key-value`的`Memecache`差
2. 存在以下误区，需要纠正，以下是纠正后的
   1. 高性能服务器不一定是多线程的
   2. 多线程（CPU上下文会切换）不一定比单线程效率高。速度：CPU>内存>硬盘
3. **核心**：Redis是将所有的数据全部放在内存中，所以使用单线程去操作效率最高。多线程（CPU上下文会切换，是一个耗时的操作！！！）对于内存系统来说，如果没有上下文切换效率就是最高的。多次读写都是在一个CPU，在内存情况下，这就是最佳方案

### Redis Key

#### 五种基本数据类型

> String

```bash
127.0.0.1:6379[3]> append key "example"  #key是之前已添加的一个字符串，在key的末尾加上example，若key不存在，相当于新建一个key
127.0.0.1:6379[3]> strlen key  #获取key的长度
incr views  #views自增1
decr views  #views减1
incrby views len #views自增len
decrby views len #views自减len
getrange key index1 index2  #截取字符串下标 [index1,index2]
getrange key 0 -1  #获取全部字符串
setrange key index1 xxx  #设置指定位置开始的字符串为XXX
setex #（set with expire）设置过期时间
setnx #（set if not exist）不存在再设置（在分布式锁中常用）
mset k1 v1 k2 v2  #同时设置多个值
mget k1 k2        #同时获取多个值
msetnx k1 v1 k2 v2  #msetnx是一个原子性的操作，要么一起成功，要么一起失败

#对象
set user:1{name:zzx,age:6}   #设置一个uesr:1对象

getset  #先get再set，如果不存在值，则返回nil，如果存在值，就获取原来的值，并设置为新的值

cas #compare and swap 比较并交换
```

> 
>
> List

Redis中，可以把List当成栈，队列，阻塞队列  。所有的List命令都是`l`开头的

```bash
127.0.0.1:6379> lpush list one  #将一个值或多个值插入到列表头部
(integer) 1
127.0.0.1:6379> lpush list two
(integer) 2
127.0.0.1:6379> lrange list 0 -1
1) "two"
2) "one"
rpush list one   	#将一个值或多个值插入到列表尾部
lpop list        	#移除链表第一个元素
rpop list        	#移除链表最后一个元素
lindex list index   #获取链表中下表为index的元素（0开始）
Llen list 			#返回列表长度
lrem list 2 three   #移除链表中的指定个数的value（这里是2个three）
ltrim list index1 index2   #截取指定下标范围的链表 [index1 index2],此时list已经被修改

rpoplpush  # 移除链表中最后一个元素并添加到新的链表中

lset list index value  #将链表中指定下表的值替换为另外一个值  如果存在就相当于更新当前下标的值，  如果链表不存在就会报错
linsert key BEFORE|AFTER pivot value   #将value插入到链表中pivot的前面或者后面
```

> 小结

- 实际上是一个链表 ，left,right都可以插入数据
- 如果key不存在，就创建新的链表，如果存在，就新增内容
- 在两边插入或改动值，效率最高，中间元素，相对效率低一点





> set

set中的值是不能重复的,set相关指令都是s开头的

无序不重复集合

```bash
sadd myset value     #向myset中添加value
smembers myset       #查看myset所有元素
sismember myset member # 判断member是不是myset里面的元素
scard myset    #获取myset集合中元素的个数
srem myset value  #移除myset集合中指定元素
srandmember myset count #随机获取myset集合中count个元素，不加count默认为1
spop myset   #随机删除一些set集合中的元素
smove myset myset1 value# 将一个指定的值(value)移动到另外一个set集合(myset1)
 sdiff myset myset1   #差集（myset- myset1，myset中有而myset1没有的）
 sinter myset myset1   #交集  共同好友就是这样实现的
 sunion myset myset1   #并集
```



> Hash(哈希，map集合)

hash相关指令都是h开头的,本质和String类型没有太大区别，还是一个简单的key-value

![1615720785480](C:\Users\wjt\AppData\Roaming\Typora\typora-user-images\1615720785480.png)

![1615720835113](C:\Users\wjt\AppData\Roaming\Typora\typora-user-images\1615720835113.png)

![1615720918397](C:\Users\wjt\AppData\Roaming\Typora\typora-user-images\1615720918397.png)

![1615720989589](C:\Users\wjt\AppData\Roaming\Typora\typora-user-images\1615720989589.png)

<img src="C:\Users\wjt\AppData\Roaming\Typora\typora-user-images\1615721082671.png" alt="1615721082671" style="zoom:150%;" />

hash变更数据user name age，尤其是用户信息之类的，经常变更的信息。**hash更适合对象的存储，String更加适合字符串存储**



> Zset(有序集合)

在set基础上，增加了一个值    `set k1 v1      Zset k1 score1 v1`

![1615721468632](C:\Users\wjt\AppData\Roaming\Typora\typora-user-images\1615721468632.png)

![1615721650535](C:\Users\wjt\AppData\Roaming\Typora\typora-user-images\1615721650535.png)

![1615721802386](C:\Users\wjt\AppData\Roaming\Typora\typora-user-images\1615721802386.png)

![1615721928187](C:\Users\wjt\AppData\Roaming\Typora\typora-user-images\1615721928187.png)

![1615722096087](C:\Users\wjt\AppData\Roaming\Typora\typora-user-images\1615722096087.png)

排行榜应用实现，取Top N测试

#### 三种特殊数据类型

> geospatial 地理位置

可以推算地理位置的信息，两地之间的距离。

只有六个命令：

```bash
# 1 geoadd  添加地理位置
# 地球两极无法添加，一般会下载城市数据，直接通过Java程序一次性导入
# 参数 key  值（经度，纬度，名称）
geoadd china:city 39.90 116.40 beijing

# 2 geopos  从key里面返回所有给定位置元素的位置（经度和纬度）获得当前定位（坐标值）
geopos china:city beijing   #获取指定城市的经度和纬度


#3 geodist  返回两个给定位置之间的距离


#4 georadius   以给定的经纬度为中心，找出某一半径内的元素
georadius china:city 110 30 1000(半径) km（以km为单位）
georadius china:city 110 30 1000(半径) km（以km为单位）withcoord  #显示他人的定位信息


# 5 georadiusbymember   给出位于指定范围内的元素，中心点是由给定的位置元素决定的
georadiusbymember china:city beijing 1000 km


# 6 geohash 返回一个或多个位置元素的geohash表示  该命令将返回11个字符的geohash字符串 
# 将二维的经纬度转化为一维的字符串，如果两个字符串越接近，那么距离越近
```

> geo底层实现原理其实就是Zset！可以使用Zset命令来操作geo



> Hyperloglog

**基数**：集合中不重复元素的数量，可以接受误差

**用途**：用于基数统计的算法

**优点**：占用的内存是固定的，2^64不同元素的个数，只需要12Kb内存。如果从内存角度来看，Hyperloglog是首选。 0.81%错误率

```bash
pfadd mykey a s d f c   #创建一组元素
pfcount mykey           #统计mykey元素的基数数量
pfadd mykey1 a s d f c  #创建第二组元素
pfmerge mykey3 mykey mykey1  #合并两组mykey mykey1到mykey3  并集
```

如果允许容错，一定可以使用Hyperloglog



> Bitmaps(位存储)

**用途**：统计用户信息，活跃不活跃，打卡统计。只要是总共两个状态的事件都可以使用Bitmaps

Bitmaps位图，数据结构，都是操作二进制位来进行记录，就只有0和1两个状态

```bash
setbit sign 0 1   #设置0号位为1
getbit sign 0     #查看0号位信息
bitcount sign begin end    #统计sign里面1的个数 在[begin end]范围内，不加范围默认是全部
```

## 事务

**Redis单条命令是保证原子性的，但是事务不保证原子性**

**Redis没有隔离级别的概念**

**所有名利在事务中并没有直接被执行，只有发起执行命令的时候才会执行！**

**Redis事务本质**：一组命令的集合，一个事务中所有命令都会被序列化，但事务执行过程中，会按照顺序执行     **一次性，顺序性，排他性！执行一些列的命令**

> Redis事务过程

- 开启事务（multi）
- 命令入队（...）
- 执行事务（exec）

```bash
discard   #放弃事务，事务队列中的命令都不会被执行
```

> 异常

1. 编译型异常（代码有问题，命令有错）事务中**所有命令都不会被执行**
2. 运行时异常（）如果事务队列中存在语法性问题（比如1/0），那么执行命令时，**其他命令是可以执行的**。**错误命令抛出异常**

### 监控！watch（面试常问）

#### 悲观锁

- 很悲观，认为什么时候都会出问题  **无论做什么都加锁，影响性能**

#### 乐观锁

- 很乐观，认为什么时候都不会出问题 。所以不会上锁。会在更新数据时判断在此期间是否有人修改过数据
- 获取version
- 更新时比较version

> Redis监视测试

正常执行成功

![1615813897639](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1615813897639.png)

测试多线程修改值，使用watch可以当作redis的乐观锁操作

![1615813960541](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1615813960541.png)

如果修改失败，获取最新的值就好

![1615814190415](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1615814190415.png)



## Jedis

> 什么是Jedis

是Redis官方推荐的Java连接开发工具，使用Java操作Redis的中间件

连接

```java
public class TestPing{
    public static void main(String[] args){
        Jedis jedis=new Jedis("127.0.0.1",6379);
        // Redis的命令在这里就是方法
        System.out.println(jedis.ping());  //输出pong表示成功
    }
}
```

事务

```java
public class TestPing{
    public static void main(String[] args){
        Jedis jedis=new Jedis("127.0.0.1",6379);
        JSONObject jsonObject=new JSONObject();
        jsonObject.put("hello","world");
        jsonObject.put("name","leifeng");
        //开启事务
        Transaction multi=jedis.multi();
        String res=jsonObject.toJSONString();
        
        try{
            multi.set("user1",res);
            multi.set("user2",res);
            
            multi.exec();  //执行事务
        }catch(Exception e){
            multi.discard(); //放弃事务
            e.printStackTrace();
        }finally{
            System.out.println(jedis.get("user1"));
            System.out.println(jedis.get("user2"));
            jedis.close(); //关闭连接
        }
    }
}
```





## SpringBoot整合

在SpringBoot 2.X以后，原来使用的jedis被替换为了lettuce

- **jedis**：采用的直连，多个线程操作的话是不安全的，如果想要避免不安全，使用jedis pool连接池   更像**BIO**模式
- **lettuce：** 采用netty，实例中可以在多个线程进行共享，不存在线程不安全，可以减少线程数量  更像**NIO**模式。(同步非阻塞)

> 整合测试

1. 导入依赖

   ```xml
   操作redis
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-data-redis</artifactId>
   </dependency>
   
   ```

   

2. 配置连接

   ```properties
   # 配置redis
   spring.redis.host=127.0.0.1
   spring.redis.port=6379
   ```

   

3. 测试

注入redisTemplate



## Redis.conf详解

启动的时候就通过配置文件来启动 

1. 配置文件unit单位  对大小写不敏感

> 网络

```
bind 127.0.0.1    #绑定的IP
protected-mode yes  #保护模式
port 6379    #端口设置
```

> 通用

```
daemonize yes    #以守护进程的方式运行，默认是no，需要开启为yes
pidfile /var/run/redis_6379.pid   #如果以后台的方式运行，需要指定一个Pid文件
logfile ""  #日志的文件位置名
database 16  #数据库数量，默认是16个
always-show-logo  #是否总是显示logo
```



> 快照

持久化，在规定的时间内，执行了多少次操作。则会持久化到文件  .rdb  .aof

redis是内存数据库，如果没有持久化，那么数据断电即失

```bash
save 900 1  #如果900s内，至少有一个key进行了修改，就会进行持久化操作
stop-write-on-bgsave-error yes  #持久化如果出错，是否还需要继续工作
rdbcompression yes  #是否压缩rdb文件，需要消耗一些CPU资源
rdbchecksum yes  #保存rdb文件时，进行错误的检查校验
dir ./  #rdb文件保存的目录
```



> replication复制



> security

可以在这里设置redis密码，默认是没有密码

```bash
config get requirepass  #获取redis密码
config set requirepass "xxxxx"   #设置redis密码
```



> 限制clients

```bash
maxclients 10000    #设置能连接上redis的最大客户端数量
maxmemory <bytes>   #配置最大的内存容量
maxmemory-policy noeviction   #内存到达上限之后的处理策略
   1 volatile-lru    只对设置了过期时间的key进行LRU
   2 allkeys-lru     删除LRU算法的key
   3 volatile-random 随即删除即将过期的key
   4 allkeys-random   随即删除
   5 volatile-ttl    删除即将过期的
   6 noeviction      永不过期，返回错误
```



> append only模式  aof配置

```bash
appendonly no  #默认不开启aof模式，默认是使用rdb持久化，所以情况下，rdb完全够用
appendfsync always   #每次修改都会sync，速度慢，消耗性能
appendfsync everysec  #每秒执行一次sync，可能会丢失这一秒的数据
appendfsync no        #不执行sync，这时操作系统自己同步数据，速度最快

rewrite #重写
# 重写规则说明  aof默认就是文件的无限追加，只会越来越大
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size  64Mb    #如果aof文件大于64Mb，会fork一个新的进程将文件重写
```







## Redis持久化

### RDB(Redis DataBase)

> what is RDB

主从复制中，rdb就是备用的

![1616069339031](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1616069339031.png)

在指定时间间隔内将内存中的数据集快照写入磁盘，也就是Snapshot快照，它恢复是将快照文件直接读到内存里

Redis会单独创建(fork)一个子进程来进行持久化，先将数据写入到一个临时文件中，待持久化过程都结束了，在用这个临时文件替换上次持久化好的文件。

整个过程中，主进程不进行任何IO操作，这就确保了极高的性能，如果需要进行**大规模的数据恢复**，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加高效

在生产环境下，备份

**RDB的缺点是最后一次持久化后的数据可能丢失**，我们默认就是RDB，一般情况下不需要修改这个配置



> 触发机制

1. save的规则满足的情况下，会自动触发RDB规则
2. 执行flushall命令，也会触发RDB规则
3. 退出redis，也会产生rdb文件
4. 备份也会自动生成一个dump.rdb文件

> 如何恢复rdb文件

1. 只需要将rdb文件放在Reids启动目录即可，redis启动时会自动检查dump.rdb，并回复其中的数据
2. 查看需要存在的位置     `config get dir`

> 优点

1. 适合**大规模的数据恢复**
2. 对数据的完整性要求不高

> 缺点

1. 需要一定时间间隔进程操作！如果Redis意外宕机，最后一次修改的数据就没有了
2. fork进程时，会占用一定的内存空间





### AOF(Append Only File)

将所有命令都记录下来，恢复时就把这个文件全部再执行一遍

> 是什么

![1616069749312](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1616069749312.png)

以日志的形式记录每个写操作，将Redis执行过的所有指令记录下来(读操作不记录)，只可以追加文件但是不可以修改文件，Redis启动之初会读取该文件重新构建数据，换言之，**Redis重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复**。

**AOF保存的是appendonly.aof文件**

默认是不开启的，如果开启的话需要手动配置！AOF只需要将配置文件中appendonly改为yes，就开启了AOF，重启redis生效（如果AOF文件有错误，Redis无法启动，redis有一个修复指令：`redis-check-aof --fix` aof文件名）



> 优缺点

优点：

1. 每一次修改都同步，文件完整性更好
2. 每秒同步一次，可能会丢失一秒的数据
3. 从不同步，效率最高

缺点：

1. 相对于数据文件来说，aof文件大小远大于rdb，修复的速度也比rdb慢
2. aof运行效率比rdb慢，所以redis默认配置就是rdb持久化！



**扩展**

![1616071312076](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1616071312076.png)

![1616071334912](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1616071334912.png)

## Redis发布订阅

Redis发布订阅(pub/sub)是一种消息通信模式：发送者（pub）发送消息，订阅者接收消息

![1616073406333](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1616073406333.png)



> 原理

Redis是用C实现的，通过分析Redis源码里面的publish.c文件，了解发布和订阅机制的底层实现。

Redis通过publish,  subscribe,  psubscribe等命令实现发布和订阅功能

通过subscribe命令订阅某频道后，redis-server里维护了一个字典，字典的键就是一个个的channel，而字典的值就是一个个链表，链表中保存了所有订阅这个channel的客户端，**subscribe命令的关键**，就是将客户端添加到给定channel的订阅链表中。

通过publish命令向订阅者发送消息，redis-server会使用给定的频道作为键，在他维护的字典中查找记录订阅这个频道的所有客户端的链表，遍历这个链表，将消息发布给所有订阅者

pub/sub从字面上理解就是发布和订阅，在redis中，你可以设定对某一个key值进行消息发布和消息订阅，当一个key值进行了消息发布之后，所有订阅他的客户端都会收到相应的消息。**这一功能最明显的用法**就是用作实时消息系统，如群聊，即时聊天等等

> 使用场景

1. 实时消息系统，实时聊天

2. 订阅，关注系统

   稍微复杂的场景要使用消息中间件(MQ,kafaka)



## Redis主从复制

> 概念

是指将一台Redis服务器的数据，复制到其他的Redis服务器，前者称为主节点(master/leader)，后者称为从节点(slave/follower),**数据的复制是单向的，只能从主节点到从节点**，master以写为主，slave以读为主。

默认情况下，每台redis服务器都是主节点，一个主节点可以有0个或多个从节点，但一个从节点只能有一个主节点。

> 主从复制的作用

1. 数据冗余：主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式
2. 故障恢复：当主节点出现问题时，可以由从节点提供服务，实现快速的故障恢复，实际上是一种服务的冗余
3. 负载均衡：在主从复制的基础上，配合读写分离，可以由主节点提供写服务，从节点提供读服务（即写redis数据时应用连接主节点，读redis数据时应用连接从节点）分担服务器负载，尤其是在写少读多的情况下，通过多个从节点分担读负载，可以大大提高redis服务器的并发量
4. 高可用（集群）基石：主从复制还是哨兵和集群能够实施的基础，因此是redis高可用的基石

![1616075453181](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1616075453181.png)

![1616075507379](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1616075507379.png)

主从复制，读写分离  80%情况下都是在进行读操作，减缓服务器压力，架构中常用



### 环境配置

只配置从库，不配置主库

复制配置文件，然后修改对应信息

1. 端口
2. pid名字
3. log文件名字
4. dump.rdb名字

修改完毕之后，启动所有的redis服务器，可以通过进程信息查看



> 主从复制原理

slave启动成功连接到master后会发送一个sync同步命令

master接到命令。启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令，在后台进程执行完毕之后，**master将传送整个数据文件到slave，并完成一次完全同步**。

**全量复制**：slave服务在接受到数据库文件数据后，将其存盘并加载到内存中

**增量复制**：master继续将新的所有收集到的修改命令依次传给slave，完成同步

但只要是重新连接到master，一次完全同步（全量复制）将被自动执行。数据一定可以在从机中看到



如果主机断开了连接，可以使用`slaveof no one`让自己变成主机，其他的节点都可以手动连接到这个主节点



### 哨兵模式

> 哨兵模式（自动选举老大的模式）

概述

![1616136786415](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1616136786415.png)

哨兵模式是一种特殊的模式，首先Redis提供了哨兵的命令，哨兵是一个独立的进程，作为进程，他会独立运行，**其原理是哨兵通过发送命令，等待redis服务器响应，从而监控运行多个redis实例**

![1616136960414](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1616136960414.png)

这里哨兵有两个作用

- 通过发送命令让redis服务器返回监控其运行状态，包括主服务器和从服务器
- 当哨兵监测到master宕机，会自动将slave切换成master·，然后通过发布订阅模式通知其他的从服务器，修改配置文件，让他们切换主机。

然而一个哨兵进程对redis服务器进行监控，可能会出现问题，为此可以使用多个哨兵进行监控，各个哨兵之间还会进行监控，形成**多哨兵模式**

![1616137383219](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1616137383219.png)

![1616137413430](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1616137413430.png)

> 测试

目前的状态是一主二从！

1. 配置哨兵配置文件 sentinel.conf

   ```bash
   sentinel monitor myredis 127.0.01 6379 1    #sentinel monitor 被监控的名称 host port 1   后面的数字1，代表主机挂了，slave投票看让谁接替成为主机，票数最多的成为主机
   ```

2. 启动哨兵。如果master节点断开了，这时就会从从机中随机选择一个服务器（里面有个投票算法）

3. 如果主机回来了，只能归并到新的主机下，当作从机，这就是哨兵模式规则

4. **优点**

   1. 哨兵集群，基于主从复制模式，所有的主从配置优点都有
   2. 主从可以切换，故障可以转移，系统可用性会更好
   3. 哨兵模式就是主从模式的升级，手动到自动，更加健壮

5. **缺点**

   1. redis不好在线扩容，集群容量一旦达到上限，在线扩容十分麻烦
   2. 实现哨兵模式的配置很麻烦，里面有很多选择

> 哨兵模式的全部配置

![1616144251226](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1616144251226.png)

![1616144340621](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1616144340621.png)

![1616144375443](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1616144375443.png)

![1616144457456](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1616144457456.png)



## Redis缓存穿透与雪崩（面试高频，工作常用）

### 缓存穿透

> 概念

用户想查询一个数据，发现redis内存数据库没有，也就是缓存没有命中，于是向持久层数据库(MySQL或其他)查询，发现也没有，所以本次查询失败。

当用户很多时，缓存都没有命中，于是都去请求了持久层数据库，会给持久层数据库造成很大的压力，这是相当于出现了**缓存穿透(查不到导致的)**。

> 解决方案

**布隆过滤器**：是一种数据结构，对所有可能查询的参数以hash形式存储，在控制层先进行校验，不符合则丢弃，从而避免了对底层存储系统的查询压力

![1616316882478](C:\Users\wjt\AppData\Roaming\Typora\typora-user-images\1616316882478.png)

**缓存空对象**：当存储层不命中后，即使返回的是空对象也将其缓存起来，同时会设置一个过期时间，之后再访问这个数据将会从缓存中获取，保护了后端数据源。

![1616317062134](C:\Users\wjt\AppData\Roaming\Typora\typora-user-images\1616317062134.png)

这种方法会存在两个问题：

1. 若空值能被缓存起来，意味着缓存需要更多的空间来缓存更多的键
2. 即使对空值设置了过期时间，还是会在缓存层和存储层的数据会有一段时间窗口的不一致，这对于需要保持一致性的业务会有影响

### 缓存击穿

> 概念

注意和缓存穿透的区别。**缓存击穿(查询太多)**，是指一个key非常热点，在不停地扛着大并发，大并发集中对着一个点进行访问，当这个key在失效的瞬间，持续的大并发就穿破缓存，直至请求数据库，就像在布上凿了一个洞。

当某个key在过期的瞬间，有大量的请求并发访问，这类数据一般是热点数据，由于缓存过期，会同时访问数据库来查询最新数据，并写回缓存，会导致数据库瞬间压力过大

> 解决方案

**设置热点数据永不过期**：从缓存层面来看，没有设置过期时间，所以不会出现热点key过期后产生的问题。

**加互斥锁**：使用分布式锁，保证对每个key在同一时间内只有一个线程去查询后端服务，其他线程没有获得分布式锁的权限，因此只需要等待即可。这种方式将高并发的压力转移到了分布式锁，因此对分布式锁的考验很大



### 缓存雪崩

> 概念

在某一时间段内，缓存集中过期失效。Redis宕机

产生雪崩的原因之一：比如双十一0点，很快迎来一波抢购，这波商品时间比较集中地放入了缓存，假设缓存一小时，那么到一点时，这批商品的缓存就过期了，而对这批商品的访问查询，都落到了数据库上。对数据库而言，就会产生周期性的压力波峰，于是所有的请求都会到达存储层，存储层的调用量暴增，使存储层也会挂掉。（所以双十一会停掉一些服务，保证主要的服务可用）

![1616318581058](C:\Users\wjt\AppData\Roaming\Typora\typora-user-images\1616318581058.png)



> 解决方案

**Redis高可用**：多增设几台redis，一台挂掉之后其他的还可以继续工作，其实就是是搭建的集群。（异地多活）

**限流降级**：在缓存失效后，通过加锁或队列来控制都数据库写缓存的线程数量。比如对某个key只允许一个线程查询数据和写缓存，其他线程等待

**数据预热**：在正式部署之前，先把可能的数据预先访问一遍，这样部分可能大量访问的数据就会加载到缓存中，在即将发生大并发访问之前手动触发加载缓存不同的key，设置不同的过期时间，让缓存时效的时间尽量均匀