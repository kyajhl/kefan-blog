---
title: redis
tags: redis
categories: 学习笔记
abbrlink: 44296
date: 2023-09-03 10:39
---

# Redis 10大数据类型

10大数据类型包括：**String**(字符串)，**Bitmap**(位图)，**Bitfield**(位域)，**Hash**(哈希表)，**List**(集合)，**Set**(列表)，**Sorted set**(有序集合 ZSet)，**Geospatial**(地理空间)，**Hyperlog**(基数统计)，**Stream**(流)

# 常用命令

1. `keys *`：查看当前库所有的 *key* 
2. `exists [key...]`：判断某个 *key* 是否存在，返回的数字为存在的 *key* 数量
3. `type key`：查看 *key* 是什么类型
4. `del [key...]`：删除指定的 *key* 数据，返回的数字为删除的 *key* 数量（原子性，若key很大，则在高并发情况下会产生阻塞队列）
5. `unlink [key...]`：非阻塞删除，会在异步中操作（非阻塞，异步操作，不会产生阻塞队列）
6. `ttl key`：查看还有多少秒过期，-1表示永不过期，-2表示已过期
7. `expire key`：为给定的 *key* 设置过期时间
8. `move key dbindex[0-15]`：将当前数据库的 *key* 移动到给定的数据库 *db* 当中，*dbindex* 直接输入数字即可
9. `select dbindex`：切换数据库[0-15]，默认为 0
10. `dbsize`：查看当前数据库 *key* 的数量
11. `flushdb`：清空当前库（很危险，尽量不要使用）
12. `flushall`：通杀全部库（更危险，最好不要使用）

备注：

> 命令不区分大小写，而 *key* 区分大小写
>
> 永远的帮助命令，help @类型，如help @String，help @List，help @Hash

# String类型

## 1、set命令

1. `set key value [nx|xx]`：*`nx`*表示 *key* 不存在时 *set* ，*`xx`*表示 *key* 存在时 *set*，若不符合情况，均返回 *null*
2. `set key value [get]`：*`get`*表示会返回**老值**，同时设置**新值**，若 *key* 不存在则返回 *null*
3. `set key value [ex seconds|px milliseconds] `：*`ex seconds`* 以秒为单位设置过期时间，*`px milliseconds`* 以毫秒为单位设置过期时间
4. `set key value [exat unix-time-seconds|pxat unix-time-milliseconds]`：*`exat unix-time-seconds`* 设置以秒为单位的*unix* 时间戳所对应的时间为过期时间，*`pxat unix-time-milliseconds`* 设置以毫秒为单位的 *unix* 时间戳所对应的时间为过期时间
5. `set key value [keepttl]`：*`keepttl`* 保留设置前指定键的生存时间，这样在修改了 *key* 值后也不会丢失原来的过期时间

## 2、mset、mget和msetnx命令

1. `mset key value [key value ...]`：同时设置多个键值
2. `mget key [key ...]`：同时获取多个键值
3. `msetnx key value [key value ...]`：只有不存在键，才能设置值，并且是同时设置，即使只有一个不存在，也不能设置值

## 3、getrange和setrange命令

1. `getrange key start end`：相当于 *Java* 中的 `subString` ，可以截取范围内的字符串
2. `setrange key offset value`：从 *offset* 开始，用 *value* 覆盖原来的值，*value* 有多少位，就覆盖多少位

## 4、incr、incrby、decr和decrby命令

1. `incr key`：递增数字
2. `incrby key increment`：增加指定的整数
3. `decr key`：递减数字
4. `decrby key decrement`：减少指定的整数

## 5、strlen和append命令

1. `strlen key`：获取字符串长度
2. `append key value`：往键值追加指定内容 *value*

## 6、setex和setnx命令(分布式锁)

1. `setex key seconds value`：给 *key* 设置过期时间和值
2. `setnx key value`：只有当 *key* 不存在时，才会设置值

## 7、getset命令

1. `getset key value`：先 *get* ，再 *set* ，和 `set key value get` 一致

# List类型

## 1、lpush、rpush和lrange命令

1. `lpush key element [element ...]`：给 *key* 设置一系列值，并且是从左插入，和链表类似
2. `rpush key element [element ...]`：给 *key* 设置一系列值，并且是从右插入，和链表类似
3. `lrange key start stop`：*list* 类型独有的遍历方式，*start* 和 *stop* 取下标，*stop* 为 -1 时表示遍历**全部**

## 2、lpop和rpop命令

1. `lpop key [count]`：从左开始弹出 *key* 对应 *count* 个值，返回的是弹出的值
2. `rpop key [count]`：从右开始弹出 *key* 对应 *count* 个值，返回的是弹出的值

## 3、lindex和llen命令

1. `lindex key index`：获取 *index* 对应的值
2. `llen key`：获取列表中元素的个数

## 4、lrem和ltrim命令

1. `lrem key count element`：删除 *count* 个值等于 *element* 的元素
2. `ltrim key start stop`：截取指定范围内的值后再赋值给 *key* 

## 5、rpoplpush命令

1. `rpoplpush source destination`：对 *surce* 执行 `rpop` 操作后，把 **弹出的元素** 再 `lpush` 到 *destination*

## 6、lset和linsert命令

1. `lset key index element`：设置对应 *index* 位置的元素值为 *element*
2. `linsert key before|after pivot element`：在对应 *pivot* 值的位置前|后，插入 *element* 值

# Hash类型

## 1、hset、hget、hmset、hmget、hgetall和hdel命令

1. `hset key field value [field value ...]`：设置 *key* 对应的键值对，可以设置多个
2. `hget key field`：获取 *key* 对应的键的值
3. `hmset key field value [field value ...]`：和 `hset` 用法差不多
4. `hmget key field [field ...]`：可以获取 *key* 多个键的值
5. `hgetall key`：可以获取 *key* 对应的多个键值对
6. `hdel key field [field ...]`：删除 *key* 多个键

## 2、hlen命令

1. `hlen key`：获取 *key* 全部键的个数

## 3、hexists 命令

1. `hexists key field`：判断 *key* 对应的键是否存在，若存在则返回 1，不存在返回 0

## 4、hkeys和hvals命令

1. `hkeys key`：获取 *key* 所有的键
2. `hvals key`：获取 *key* 多有的值

## 5、hincrby和hincrbyfloat命令

1. `hincrby key field increment`：给 *key* 对应的键值加上 *increment* ，但是键对应的值只能为数字，否则会报错
2. `hincrbyfloat key field increment`：给 *key* 对应的键值加上 *increment* ，*increment* 可以为浮点数

## 6、hsetnx命令

1. `hsetnx key field value`：给 *key* 设置键值，但是这个键必须不存在，否则会报错

# Set类型

## 基本命令

### 1、sadd命令

1. `sadd key member [member ...]`：往 *key* 里面添加成员，可以添加多个，但不能重复

### 2、smembers命令

1. `smembers key`：获取 *key* 全部的成员

### 3、sismember命令

1. `sismember key member`：查看 *key* 是否有成员 *member* ，若没有则返回 0，有则返回 1

### 4、srem命令

1. `srem key member [member ...]`：删除 *key* 里面的成员，可以删除多个

### 5、scard命令

1. `scard key`：获取 *key* 里面的成员个数

### 6、srandmember命令

1. `srandmember key [count]`：随机获取 *key* 里面的 *count* 个成员，成员不删除

### 7、spop命令

1. `spop key [count]`：随机弹出 *key* 里面 *count* 个值，成员会删除

### 8、smove命令

1. `smove source destination member`：从 `source` 中移动成员到 `destination` 中

## 集合运算

### 1、差集运算(sdiff)

1. `sdiff key [key ...]`：获取两个 *key* 之间的差集，如 A-B，获取 A 中有的，B 中没有的

### 2、并集运算(sunion)

1. `sunion key [key ...]`：获取两个 *key* 之间的并集

### 3、交集运算(sinter和sintercard)

1. `sinter key [key ...]`：获取两个 *key* 之间的交集
2. `sintercard numkeys key [key ...] [LIMIT limit]`：获取 *numkeys* 个 *key* 的交集元素个数，后面的 *limit* 表示显示几个

# ZSet类型

## 1、zadd命令

1. `zadd key [NX|XX] [GT|LT] [CH] [INCR] score member [score member ...]`：往 *key* 里面添加成员和分数

## 2、zrange命令

1. `zrange key start stop [byscore|bylex] [rev] [LIMIT offset count] [WITHSCORES]`：遍历 *key* ，*withscores* 表示可以带着分数

## 3、zrevrange命令

1. `zrevrange key start stop [WITHSCORES]`：反向遍历 *key* ，其他的和 `zrange` 一样

## 4、zrangebyscore命令

1. `zrangebyscore key min max [WITHSCORES] [LIMIT offset count]`：遍历 *key* ，从规定的最小值到最大值，即 `>=,<=` ，可以用 *offset* 指定返回的数量，如果 *min* 和 *max* 的左边带有`(`，如 `(min` 或者 `(max` ，表示 `>` 或者 `<`

## 5、zscore命令

1. `zscore key member`：获取 *key* 对应成员的分数

## 6、zcard命令

1. `zcard key`：获取 *key* 的成员个数

## 7、zrem命令

1. `zrem key member [member ...]`：删除 *key* 的成员，可以删除多个

## 8、zincrby命令

1. `zincrby key increment member`：给 *key* 对应的成员分数增加 *increment*

## 9、zcount命令

1. `zcount key min max`：获得 *key* 在 *min* 到 *max* 范围内的成员个数

## 10、zmpop命令

1. `zmpop numkeys key [key ...] MIN|MAX [COUNT count]`：从 *numkeys* 个 *key* 中弹出最小(MIN)或最大(MAX)分数的成员，弹出 *count* 个

## 11、zrank命令

1. `zrank key member [WITHSCORES]`：正序获取 *key* 对应成员的下标值

## 12、zrevrank命令

1. `zrevrank key member [WITHSCORES]`：逆序获取 *key* 对应成员的下标值

# Bitmap类型

***\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*底层是 String\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\****

## 1、setbit命令

1. `setbit key offset value`：设置 *key* 的偏移量的值为多少，*offset* 从 0 开始，***value* 只能为 0 或 1**

## 2、getbit命令

1. `getbit key offset`：获得 *key* 偏移量 *offset* 的值

## 3、strlen命令

1. `strlen key`：获得 *key* 占据几个字节，不是获得字符串的长度，超过 8 位后自己按照 8 位一组一byte再扩容

## 4、bitcount命令

1. `bitcount key [start end [BYTE|BIT]]`：获取  *key* 里面从 *start* 位到 *end* 位含有 1 的位数，默认是 *BYTE* 单位，最好是用 *BIT* 作为单位

## 5、bitop命令

1. `BITOP AND|OR|XOR|NOT destkey key [key ...]`：让多个 *key* 可以进行 ***位运算*** (与  或  异或  非)，结果给 *destkey*

# HyperLogLog类型

***\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*基数统计\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\****

## 1、pfadd命令

1. `pfadd key [element [element ...]]`：往 *key* 里面插入 *element* 元素

## 2、pfcount命令

1. `pfcount key [key ...]`：去掉重复的元素后，统计剩下的元素个数，多个 *key* ，会统计总的去掉重复元素之后的个数

## 3、pfmerge命令

1. `pfmerge destkey [sourcekey [sourcekey ...]]`：合并多个 *sourcekey* 到 *destkey* 中，并去掉重复元素

# GEO类型









