
Redis 是基于内存的 Key Value 的 NoSql 数据库，由于其高性能，高可用，支持分布式集群的优点被广泛应用于缓存的业务场景。本篇文章就来详细了解下 Redis 缓存机制及内存淘汰策略。

## 如何使用缓存？

我们先来插入一个最简单的 key

```
127.0.0.1:6379> set name aaa
OK
127.0.0.1:6379>
```

OK, 插入成功。我们再来设置一下 key 的过期时间, redis 有 4 个命令来设置过期时间：

```
expire <key> <ttl>：            // 将 key 的生存时间设置为 ttl 秒
pexpire <key> <ttl>：           // 将 key 的生存时间设置为 ttl 毫秒
expireat <key> <timestamp>：    // 将 key 的过期时间设置为 timestamp 所指定的秒数时间戳
pexpireat <key> <ttl>：         // 将 key 的过期时间设置为 timestamp 所指定的毫秒数时间戳
```

过期之前我们可以通过命令查看剩余的时间

```
127.0.0.1:6379> ttl name
(integer) 936
127.0.0.1:6379>
```

如果自己不小心设置错了过期时间，那么我们可以删除先前的过期时间

```
persist <key>       // 命令可以移除一个键的过期时间
```

有的时候，我们可以将 set 和 expire 命令合并一个命令使用

```
127.0.0.1:6379> setex name 1000 aaa
OK
127.0.0.1:6379>
```

> setex 命令只能对字符串类型的数据进行 set 和 expire 操作。

## redis的过期策略

redis采用了 “定期删除+惰性删除”  的过期策略。

### 定期删除

原理：定期删除指的是redis默认每隔100ms就随机抽取一些设置了过期时间的key，检测这些key是否过期，如果过期了就将其删掉。

为什么会选择一部分，而不是全部：因为如果这是redis里面有大量的key都设置了过期时间，那么如果全部去检测一遍，CPU负载就会很高，会浪费大量的时间在检测上面，甚至直接导致redis挂掉。所有只会抽取一部分而不会全部检查。

出现问题：这样的话就会出现大量的已经过期的key并没有被删除，这就是 为什么有时候大量的key明明已经过了失效时间，但是redis的内存还是被大量占用的原因 ，为了解决这个问题，就需要 惰性删除 这个策略了。

### 惰性删除

原理：惰性删除不在是redis去主动删除，而是在你要获取某个key 的时候，redis会先去检测一下这个key是否已经过期，如果没有过期则返回给你，如果已经过期了，那么redis会删除这个key，不会返回给你。

这样两种策略就保证了 过期的key最终一定会被删除掉 ，但是这只是保证了最终一定会被删除，要是定时删除漏掉了大量过期的key，而且我们也没有及时的去访问这些key，那么这些key不就不会被删除了吗？不就会一直占着我们的内存吗?这样不还是会导致redis内存耗尽吗？

由于存在这样的问题，所以redis引入了内存淘汰机制来解决。

## 内存淘汰机制

内存淘汰机制就保证了在redis的内存占用过多的时候，去进行内存淘汰，也就是删除一部分key，保证redis的内存占用率不会过高，那么它会删除那些key呢？
redis提供了6中内存淘汰策略，我们可以去进行选择，六中策略如下:

* noeviction：当内存不足以容纳新写入数据时，新写入操作会报错，无法写入新数据，一般不采用。
* allkeys-lru：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的key，这个是最常用的。
* allkeys-random：当内存不足以容纳新写入的数据时，在键空间中，随机移除key，一般也不使用。
* volatile-lru：volatile-lru：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，移除最近最少使用的key（这个一般不太合适）  。
* volatile-random：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，随机移除某个key  。
* volatile-ttl：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，有更早过期时间的key优先移除。

volatile 和 allkeys 规定了是对已设置过期的数据集还是从全部的数据集淘汰数据。具体的使用应根据不同的业务场景来配置不同的策略。一般来说，使用 volatile-lru 或者 allkeys-lru 是比较合理的。删除最少使用的 key。如果使用 redis 作为缓存，就使用 volatile-lru 。如果除了缓存还使用存储，就使用 allkeys-lru。

注意：ttl 和 random 的实现方法比较简单，lru 的实现方法是 redis 会随机挑选几个键值对，然后取出lru 最大的键值对淘汰。所以并不是严格的就会淘汰整个数据集中最少使用的 key，而是随机数据集中的最少使用的 key。
