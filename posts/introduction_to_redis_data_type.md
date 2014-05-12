---
date: 2011-01-30
title: Redis的数据类型
---

[Redis](http://redis.io/) 是一个开源的高级key-value数据库. 由 [Salvatore
Sanfilippo](http://twitter.com/antirez) (就算是老外,名字也够奇怪) 和
[Pieter Noordhuis](http://twitter.com/pnoordhuis) 使用C开发.
如果你使用过 [memcached](http://memcached.org/) ,就会发现两者十分相似,
但是 [Redis](http://redis.io/) 并非memcached的++,
Redis不仅支持更丰富的数据结构,并且数据
[可持久化](http://redis.io/topics/persistence) .
Redis的两个最大优点:

**超级快:** 使用Redis自带的redis-benchmark工具测试, 在Linux 2.6,Xeon
X3320 2.5G的机器上可以达到每秒110000次SET 81000次GET. 见 [How fast is
Redis?](http://redis.io/topics/benchmarks)

**丰富的数据类型** Redis目前支持的

[数据类型](http://redis.io/topics/data-types) ,

* [Binary-safe](http://en.wikipedia.org/wiki/Binary-safe) String
* List
* Set
* Sorted set
* Hashe

Redis会在启动后将所有的数据加载到内存,这也是为什么Redis会如此快的原因之一.
但是有时候我们并不需要用到所有数据,
这样加载所有数据对内存是种巨大的浪费. Redis 2.0引进了 [Virtual
Memory](http://redis.io/topics/virtual-memory) 技术来避免这种情况.

闲话休提, 回归正传. String就不用说它了, 说说其它四种类型

**List:** Redis中的list是使用 [Linkedlist](http://en.wikipedia.org/wiki/Linked_list) 实现的,
也就是说向list中添加新的数据复杂度为O(1).
list主要 [操作](http://redis.io/commands#list) :

* [LPUSH](http://redis.io/commands/lpush) 向list的左侧(头部)插入数据.
* [RPUSH](http://redis.io/commands/rpush) 向list的右侧(尾部)插入数据.
* [LRANGE](http://redis.io/commands/lrange) 取得范围内的数据

```
5.times do |i|
  redis.lpush('list1', "lpush-#{i}")
end

5.times do |i|
  redis.rpush('list1', "rpush-#{i}")
end

puts redis.lrange('list1', 0, -1)

#lpush-4
#lpush-3
#lpush-2
#lpush-1
#lpush-0
#rpush-0
#rpush-1
#rpush-2
#rpush-3
#rpush-4
```


**Set:** Set是未排序的集合.

set的主要 [操作](http://redis.io/commands#set) :

* [SADD](http://redis.io/commands/sadd) 向集合中添加数据.
* [SISMEMBER](http://redis.io/commands/sismember)
判断数据是否是集合的元素.
* [SINTER](http://redis.io/commands/sinter) 对多个集合取交集.
* [SMEMBERS](http://redis.io/commands/smembers) 取得集合的所有元素.

```
5.times do |i|
  redis.sadd('set1', "set-#{i}")
end

puts redis.sismember('set1', 'set-2')
#true

puts redis.smembers('set1')
#set-2
#set-3
#set-4
#set-0
#set-1

5.times do |i|
  redis.sadd('set2', "set-#{i + 3}")
end

puts redis.sinter('set1', 'set2')
#set-3
#set-4
```


**Sorted set** 与Set十分相似, 但是它是排序的集合.

sorted set的主要 [操作](http://redis.io/commands#sorted_set) :

* [ZDD](http://redis.io/commands/zadd) 类似SADD, 但是有个score参数用作排序.
* [ZRANGE](http://redis.io/commands/zrange) 取得范围内的数据, 注意结果已按照score排序
* [ZREVRANGE](http://redis.io/commands/zrevrange) 与ZRANGE一样, 但是结果顺序与ZRANGE相反
* [ZRANGEBYSCORE](http://redis.io/commands/zrangebyscore) 用score取得范围内的数据

```
#5.times do |i|
#  redis.zadd('sortedset1', rand(100), "sortedset-#{i}")
#end

puts redis.zrange('sortedset1', 0, -1)
#sortedset-1
#sortedset-3
#sortedset-0
#sortedset-2
#sortedset-4
puts redis.zrevrange('sortedset1', 0, -1, {:with_scores => true})
#sortedset-4
#79
#sortedset-2
#62
#sortedset-0
#38
#sortedset-3
#11
#sortedset-1
#4
puts redis.zrangebyscore('sortedset1', 10, 90, :limit => [0, 2], :with_scores => true)
#sortedset-3
#11
#sortedset-0
#38
```

**Hashe** hash表结构

Hashe的主要 [操作](http://redis.io/commands#hash) :

* [HSET](http://redis.io/commands/hset) 对hash的field设置值
* [HGET](http://redis.io/commands/hget) 取得hash的field的值
* [HDEL](http://redis.io/commands/hdel) 删除field的值
* [HKEYS](http://redis.io/commands/hkeys) 取得hash的所有field
* [HVALS](http://redis.io/commands/hvals) 取得hash的所有的值
* [HMGET](http://redis.io/commands/hmget) 一次取得多个field的值

```
5.times do |i|
  redis.hset("hash1", "field#{i}", "hash-value#{i}")
end

puts redis.hkeys("hash1")
puts '*' * 10
redis.hdel("hash1", "field1")
puts '*' * 10
puts redis.hvals("hash1")
puts '*' * 10
puts redis.hgetall("hash1")
puts '*' * 10
puts redis.hget("hash1", "field2")
puts '*' * 10
puts redis.hmget("hash1", "field0", "field1", "field3", "field4")
#field0
#field1
#field2
#field3
#field4
#**********
#**********
#hash-value0
#hash-value2
#hash-value3
#hash-value4
#**********
#{"field0"=>"hash-value0", "field2"=>"hash-value2", "field3"=>"hash-value3", "field4"=>"hash-value4"}
#**********
#hash-value2
#**********
#hash-value0
#
#hash-value3
#hash-value4
```