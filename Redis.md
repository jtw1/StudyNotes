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

p27  14.20