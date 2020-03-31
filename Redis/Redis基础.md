## 前言

Redis在互联网技术存储方面使用非常广泛，所有的后端技术都要了解edis的使用和原理方面。

## 为啥用Redis

因为传统的关系型数据库如Mysql已经不能适用所有的场景了，比如秒杀的库存扣减，APP首页的访问流量高峰等等，都很容易把数据库打崩，所以引入了缓存中间件，目前市面上比较常用的缓存中间件有Redis 和 Memcached 不过中和考虑了他们的优缺点，最后选择了Redis。

## Redis有哪些数据结构

字符串String、字典Hash、列表List、集合Set、有序集合SortedSet。

如果你是Redis中高级用户，而且你要在这次面试中突出你和其他候选人的不同，还需要加上下面几种数据结构HyperLogLog、Geo、Pub/Sub

如果你还想加分，那你说还玩过Redis Module，像BloomFilter，RedisSearch，Redis-ML，这个时候面试官得眼睛就开始发亮了，心想这个小伙子有点东西啊。

本人在面试回答到Redis相关的问题的时候，经常提到BloomFilter（布隆过滤器）这玩意的使用场景是真的多，而且用起来是真的香，原理也好理解



