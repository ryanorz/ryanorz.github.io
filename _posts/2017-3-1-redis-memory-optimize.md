---
layout:     post
title:      "Redis内存优化"
date:       2017-3-1
author:     "Ryan"
header-img: "img/post-bg-2017.jpg"
tags:
    - Redis
---

[redis-memory-usage-optimization-storage](http://www.infoq.com/cn/articles/tq-redis-memory-usage-optimization-storage)

[官方文档 memory-optimization](https://redis.io/topics/memory-optimization)

# 2011年的Redis文章总结

这是一篇2011年的文章，有些东西有时效性，如`VM`,现在版本的`Redis`已经没有`VM`了

作者根据`Redis`实际存储数据的手段来谈常用的内存优化手段．简单的总结起来有如下几点：

１. 设置`redis.conf`中`maxmemory`

２. 设置不同数据类型在不同状态下的存储策略．(list和hash在数据量小的时候采用线性存储，节省空间，量大的时候影响查询性能，需要进行权衡)

```
hash-max-zipmap-entries 512 (value这个Map内部不超过512个成员时使用线性紧凑存储)
hash-max-zipmap-value   64 (value这个Map内部的每个成员值长度不超过64字节采用线性紧凑存储)

zset-max-ziplist-entries 128
zset-max-ziplist-value 64

list-max-ziplist-size -2

set-max-intset-entries 512 (set内部数据全是数值型，且包含多少节点以下使用紧凑格式存储)
```

３. `Redis`内部对数值型数据采用`shared integer`方式节省内存开销，可以根据实际情况修改源码中的宏定义`REDIS_SHARED_INTEGERS`(默认值1000)

４. Redis持久化磁盘IO方式带来的问题.当你的Redis物理内存使用超过内存总容量的3/5时就会开始比较危险了。

# 官方文档总结

1. 对于不同数据类型的特殊编码优化与这篇文章基本一致．

```
hash-max-zipmap-entries 512 (hash-max-ziplist-entries for Redis >= 2.6)
hash-max-zipmap-value 64  (hash-max-ziplist-value for Redis >= 2.6)
list-max-ziplist-entries 512
list-max-ziplist-value 64
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
set-max-intset-entries 512
```

2. 编译32位版本的Redis．减少每个key占用的内存（但是这样使用的内存限制为４G）

3. Bit与Byte级别的操作,`GETRANGE`, `SETRANGE`, `GETBIT`, `SETBIT`

4. 尽可能使用hash, 使用hash抽象需要内存需求高的业务.

5. maxmemory（但这个可能会有额外的内存分配），设置这个需要评估`peak memory usage`.因为当Redis删除数据时，并不会立即将内存还给操作系统，所以不设置这个参数会导致无限申请内存.
