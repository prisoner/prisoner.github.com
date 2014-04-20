---
date: 2011-02-06
title: Memcached的Consistent Hashing
---

在有N台 [Memcached](http://memcached.org/) 的情况下,
如何有效的将数据分布到每台server上,
在添加或移除server时如何减少对已有数据的影响.

Memcached使用了Consistent Hashing算法来解决这些问题.

### Consistent Hashing算法

详见 [Consistent hashing and random
trees](http://portal.acm.org/citation.cfm?id=258660) 和 [Consistent
Hashing](http://en.wikipedia.org/wiki/Consistent_hashing)

Memcached的Consistent Hashing算法:

1.  生成一个0~2^32-1的环
2.  求出memcached server(node)的哈希值, 并将其配置到环上
3.  求出数据的哈希值, 并将其配置到环上
4.  将数据映射到node上. 从数据配置到的位置开始顺时针查找,将数据保存到找到的第一个node上, 如果超过2^32-1还找不到node，就保存到第0个节点

<img src="{{urls.media}}/2011-01-30_01.png" alt="" width="600">

*图片来源 http://tech.idv2.com/2008/07/24/memcached-004/*

注意,Memcached的Consistent Hashing算法是在客户端实现的. 另外,
有些Consistent
Hashing算法的实现还使用了虚拟节点的方法来最大抑制数据的分布不均匀,
来减小服务器增减时的缓存重新分布.

[memcached分布测试报告](http://www.javaeye.com/topic/346682)

[ruby实现](https://github.com/superfeedr/consistent_hashr/blob/master/lib/consistent_hashr.rb)

<script src="https://gist.github.com/1112126.js"></script>

