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

Redis(Remote Dictionary Server)，远程字典服务。是一个开源的使用ANSI C语言编写。支持网络，可基于内存亦可持久化的日志型，Key-value数据库，并提供多种语言的API。

> Redis能干嘛

1. 内存存储，持久化，内存中是断电即失，所以说持久化很重要
2. 效率高，可用于高速缓存
3. 发布订阅系统，地图信息分析
4. 计数器（浏览量），计时器

> 数据类型

五大基本数据类型：String   List   Set   Hash   Zset   三种特殊数据类型：geo   hyperloglog    bitMap