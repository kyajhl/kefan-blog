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

1. keys *：查看当前库所有的key
2. exists [key...]：判断某个key是否存在，返回的数字为存在的key数量
3. type key：查看key是什么类型
4. del [key...]：删除指定的key数据，返回的数字为删除的key数量（原子性，若key很大，则在高并发情况下会产生阻塞队列）
5. unlink [key...]：非阻塞删除，会在异步中操作（非阻塞，异步操作，不会产生阻塞队列）
6. ttl key：查看还有多少秒过期，-1表示永不过期，-2表示已过期
7. expire key：为给定的key设置过期时间
8. move key dbindex[0-15]：将当前数据库的key移动到给定的数据库db当中，dbindex直接输入数字即可
9. select dbindex：切换数据库[0-15]，默认为0
10. dbsize：查看当前数据库key的数量
11. flushdb：清空当前库（很危险，尽量不要使用）
12. flushall：通杀全部库（更危险，最好不要使用）

备注：

> 命令不区分大小写，而key区分大小写
>
> 永远的帮助命令，help @类型，如help @String，help @List，help @Hash

# String类型

