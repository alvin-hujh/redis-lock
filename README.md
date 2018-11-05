# 分布式锁的实现标准

> 首先，为了确保分布式可用，我们至少要确保锁的实现同时满足以下四个条件

* 互斥性。在任意时刻，只有一个客户端能持有锁
* 不会发生死锁。即使只有一个客户端在持有锁的期间崩溃而没有主动解锁，也能保证后续其它客户端能加锁
* 具有容错性。只要大部分的 Redis 节点正常运行，客户端就可以加锁和解锁
* 解铃还须系铃人。加锁和解锁必须是同一个客户端，客户端不能把别人加的锁给解了

> 注意几个点
- jedis 的 setnx()方法作用就是 SET IF NOT EXIST, expire()方法就是给锁加一个过期时间。
乍一看好像和前面的 set() 方法结果一样，然而由于这是两条命令，不具有原子性，
如果程序在执行完 setnx() 方法之后突然崩溃，导致锁没有设置过期时间，那么将会发生死锁。
网上之所以有人这样实现，是因为低版本的 jedis 并不支持多参数的set()方法。

-需要考虑操作的原子性，使用 script lua 脚本代替多步客户端方法的调用

使用Spring Boot 2.x版本需要自行根据 lettuce api 修改相关实现
考虑广大同学习惯使用 spring-boot-starter-data-redis，故基于此进行封装。

注意：
 1. 由于 jesi 已经很久没有更新（2016.07起），SpringBoot2.x 版本已经放弃使用 jedis 作为底层 redis 客户端实现
 2. 在 SpringBoot2.x 中，spring-boot-starter-data-redis 默认实现已经改为 lettuce，如果使用该版本 SpringBoot，需要做对应修改 
