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

***\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*基数统计，底层是String\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\****

## 1、pfadd命令

1. `pfadd key [element [element ...]]`：往 *key* 里面插入 *element* 元素

## 2、pfcount命令

1. `pfcount key [key ...]`：去掉重复的元素后，统计剩下的元素个数，多个 *key* ，会统计总的去掉重复元素之后的个数

## 3、pfmerge命令

1. `pfmerge destkey [sourcekey [sourcekey ...]]`：合并多个 *sourcekey* 到 *destkey* 中，并去掉重复元素

# GEO类型

***\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*底层是ZSet\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\****

***\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*中文乱码，在redis-cli后面加上--raw\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\****

## 1、geoadd命令

1. `geoadd key [NX|XX] [CH] longitude latitude member [longitude latitude member ...]`：*longitude* 经度，*latitude* 纬度，*member* 位置名称 

## 2、geopos命令

1. `geopos key [member [member ...]]`：返回经纬度

## 3、geohash命令

1. `geohash key [member [member ...]]`：返回经纬度坐标的geohash表示

## 4、geodist命令

1. `geodist key member1 member2 [M|KM|FT|MI]`：两个位置之间的距离，后面跟单位 *KM* 千米，*M* 米，*FT* 英尺，*MI* 英里

## 5、georadius命令

1. `georadius key longitude latitude radius M|KM|FT|MI [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count [ANY]] [ASC|DESC] [STORE key|STOREDIST key]`：以给定的经纬度为中心，返回键包含的位置元素当中，与中心的距离不超过给定最大距离(***radius***)的所有位置元素，***withdist***：返回位置元素的同时，将位置元素与中心之间的距离也一并返回，***withcoord***：将位置元素的经度与纬度一并返回，***withhash***：以52位有符号整数的形式，返回位置元素经过原始***geohash***编码的有序集合分值，***count***：限定返回的记录数

## 6、georadiusbymember命令

1. `georadiusbymember key member radius M|KM|FT|MI [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count [ANY]] [ASC|DESC] [STORE key|STOREDIST key]`：根据位置名称查找，和 ***georadius*** 的用法差不多

# Stream类型

## 队列相关指令

### 1、xadd命令

1. `xadd key [NOMKSTREAM] [MAXLEN|MINID [=|~] threshold [LIMIT count]] *|id field value [field value ...]`：添加消息到队列末尾，* 表示自动生成 ***MessageID***，若不加 \*，则要自己手动加 ***ID***，格式为***“时间戳-递增数字”***后面顺序跟着一堆业务 ***field***，***value***，***NOMKSTREAM*** 表示如果队列不存在，是否自动创建队列，默认自动创建

### 2、xtrim命令

1. `xtrim key MAXLEN|MINID [=|~] threshold [LIMIT count]`：对 ***Stream*** 的长度进行截取，***maxlen*** 允许的最大长度，***minid*** 允许的最小 ***id***

### 3、xdel命令

1. `xdel key id [id ...]`：删除 ***key*** 对应的 ***id***  

### 4、xlen命令

1. `xlen key`：获取 ***key*** 里面成员的数量

### 5、xrange命令

1. `xrange key start end [COUNT count]`：***start*** 表示开始值，`-`代表最小值，***end*** 表示结束值，`+` 代表最大值，***count*** 代表最多可以获取多少个值

### 6、xrevrange命令

1. `xrevrange key end start [COUNT count]`：***end*** 取 `+`，***start*** 取 `-`

### 7、xread命令

1. `xread [COUNT count] [BLOCK milliseconds] STREAMS key [key ...] id [id ...]`：***count*** 最多读取多少条消息，***block*** 是否已阻塞的方式读取消息，默认不阻塞，如果 `milliseconds` 设置为 0，表示永远阻塞，***$*** 表示从尾，***0*** 表示从头

## 消费组相关指令

### 1、xgroup create指令

1. `xgroup create key group id|$ [MKSTREAM] [ENTRIESREAD entries-read]`：用于创建消费者组

### 2、xreadgroup命令

1. `GROUP group consumer [COUNT count] [BLOCK milliseconds] [NOACK] STREAMS key [key ...] id [id ...]`：`>` 表示从第一条尚未被消费的消息开始读取，消费组 *group* 内的消费者 *consumer* 从 *key* 消息队列中读取所有消息

### 3、xpending命令

1. `xpending key group [[IDLE min-idle-time] start end count [consumer]]`：查询每个消费组内所有消费者(已读取，但尚未确认)的消息，查看某个消费者具体读了哪些数据

### 4、xack命令

1. `xack key group id [id ...]`：向消息队列确认消息处理已完成

### 5、xinfo stream命令

1. `xinfo stream key [FULL [COUNT count]]`：用于打印 Stream\Consumer\Group的详细信息

# redis客户端

## 1、Jedis

### 使用方法

1. 在springboot中引入依赖

   ```xml
   <!--   使用 jedis 作为 redis客户端    -->
   <dependency>
       <groupId>redis.clients</groupId>
       <artifactId>jedis</artifactId>
       <version>3.7.0</version>
   </dependency>
   ```

2. 测试jedis

   ```java
   @SpringBootTest
   public class RedisTestApplicationTests {
   
       private Jedis jedis;
   
       @BeforeEach
       public void setUp() {
           // 建立连接
           jedis = new Jedis("192.168.65.128", 6379);
           // 设置密码
           jedis.auth("111111");
           // 选择库
           jedis.select(0);
       }
   
       @Test
       public void testJedis() {
           Set<String> keys = jedis.keys("*");
           keys.forEach(System.out::println);
   
       }
   
       @AfterEach
       public void setDown() {
           // 关闭连接
           if (jedis != null) {
               jedis.close();
           }
       }
   }
   ```


### jedis连接池

Jedis本身是线程不安全的，并且频繁的创建和销毁连接会有性能损耗，因此推荐使用Jedis连接池代替Jedis直连方式

### 连接池配置

```java
public class JedisConnectFactory {
    private static final JedisPool jedisPool;

    static {
        // 配置连接池
        JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
        // 最大连接数
        jedisPoolConfig.setMaxTotal(8);
        // 最大空闲连接数
        jedisPoolConfig.setMaxIdle(8);
        // 最小空闲连接数
        jedisPoolConfig.setMinIdle(0);
        // 等待时长
        jedisPoolConfig.setMaxWait(Duration.ofMillis(1000));
        // 配置连接池对象
        jedisPool = new JedisPool(jedisPoolConfig,
                "192.168.65.128", 6379, 1000, "111111");
    }

    public static Jedis getJedis() {
        return jedisPool.getResource();
    }
}
```

在测试方法处，直接调用即可

```java
@BeforeEach
public void setUp() {
    // 建立连接
    // jedis = new Jedis("192.168.65.128", 6379);
    // 连接处创建 Jedis
    jedis = JedisConnectFactory.getJedis();
    // 设置密码
    jedis.auth("111111");
    // 选择库
    jedis.select(0);
}
```

## 2、SpringDataRedis

### 使用方法

1. 引入依赖

   ```xml
   <!-- 引入 redis -->
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-data-redis</artifactId>
   </dependency>
   <!--   连接池依赖(commons-pool)    -->
   <dependency>
       <groupId>org.apache.commons</groupId>
       <artifactId>commons-pool2</artifactId>
   </dependency>
   ```

2. 写入配置

   ```yaml
   spring:
     redis:
       host: 192.168.65.128
       port: 6379
       password: 111111
       lettuce:
         pool:
           max-active: 8
           max-idle: 8
           min-idle: 0
           max-wait: 100ms
   ```

3. 测试方法

   ```java
   @SpringBootTest
   public class SpringDataRedisTests {
       @Autowired
       private RedisTemplate<String, Object> redisTemplate;
   
       @Test
       public void testString() {
           redisTemplate.opsForValue().set("name", "zhangsan");
           Object name = redisTemplate.opsForValue().get("name");
           System.out.println(name);
       }
   }
   ```

### 配置redis

对redis配置之后，存入对象就不会以字节的形式表示出来了，实现了序列化和反序列化的重新配置

```java
@Configuration
public class RedisConfig {

    @Resource
    private RedisConnectionFactory redisConnectionFactory;

    @Bean
    public RedisTemplate<String, Object> redisTemplate() {
        // 创建 redisTemplate 对象
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        // 设置连接工厂
        redisTemplate.setConnectionFactory(redisConnectionFactory);
        // 创建 JSON 序列化工具
        GenericJackson2JsonRedisSerializer jsonRedisSerializer = new GenericJackson2JsonRedisSerializer();
        // 设置 key 的序列化
        redisTemplate.setKeySerializer(RedisSerializer.string());
        redisTemplate.setHashKeySerializer(RedisSerializer.string());
        // 设置 value 的序列化
        redisTemplate.setValueSerializer(jsonRedisSerializer);
        redisTemplate.setHashValueSerializer(jsonRedisSerializer);
        // 返回
        return redisTemplate;
    }
}
```

### 使用 StringRedisTemplate

StringRedisTemplate只能使用<String, String>的泛型，但是可以手动序列化和反序列化，更为简单

```java
@SpringBootTest
public class SpringDataRedisTests {

    @Resource
    private StringRedisTemplate stringRedisTemplate;
    
    private static final ObjectMapper objectMapper = new ObjectMapper();

    @Test
    public void testStringRedisTemplate() throws JsonProcessingException {
        User user = new User("zhangsan", 18);
        String userJson = objectMapper.writeValueAsString(user);
        stringRedisTemplate.opsForValue().set("user:1", userJson);
        String s = stringRedisTemplate.opsForValue().get("user:1");
        if (!Objects.isNull(s)) {
            User user1 = objectMapper.readValue(s, User.class);
            System.out.println(user1);
        }
    }
}
```

