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

1. `keys *`：查看当前库所有的key
2. `exists [key...]`：判断某个key是否存在，返回的数字为存在的key数量
3. `type key`：查看key是什么类型
4. `del [key...]`：删除指定的key数据，返回的数字为删除的key数量（原子性，若key很大，则在高并发情况下会产生阻塞队列）
5. `unlink [key...]`：非阻塞删除，会在异步中操作（非阻塞，异步操作，不会产生阻塞队列）
6. `ttl key`：查看还有多少秒过期，-1表示永不过期，-2表示已过期
7. `expire key`：为给定的key设置过期时间
8. `move key dbindex[0-15]`：将当前数据库的key移动到给定的数据库db当中，dbindex直接输入数字即可
9. `select dbindex`：切换数据库[0-15]，默认为0
10. `dbsize`：查看当前数据库key的数量
11. `flushdb`：清空当前库（很危险，尽量不要使用）
12. `flushall`：通杀全部库（更危险，最好不要使用）

备注：

> 命令不区分大小写，而key区分大小写
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

