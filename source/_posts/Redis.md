---
title: Redis数据库
date: 2023.10.1
author: Aik
img: ../images/deepLearning.jpg
top: true
hide: false
cover: true
coverImg: /images/1.jpg
password: 
toc: true
mathjax: true
summary: 关于Redis的相关简介以及项目实战
categories: Java开发
tags:
  - 数据库
  - Java
---

#  Redis

**Re**mote **Di**ctionary **S**erver(远程词典服务器)，一种基于内存的键值型NoSQL数据库。

特点：

- 键值（Key-value）型，value支持多种不同数据结构，功能丰富
- 单线程，每个命令具备原子性
- 低延迟，速度快（基于内存、IO多路复用、良好的编码）
- 支持数据持久化（AOF和RDB）
- 支持主从集群、分片集群
- 支持多语言客户端

## 数据结构

key一般使用String类型，但value分为多种类型。

基本类型:

- String: hello world
- Hash: {name:"xwy",age:25}
- List: [A ->B ->C ->D]
- Set: {A,B,C}
- SortedSet: {A:1, B:2, C:3}

特殊类型:

- GEO: {A:(120.3, 30.5)}
- BitMap: 0110110101110101011
- HyperLog: 0110110101110101011

## 命令

### 通用

```bash
KEYS ${key} #查看指定key，*代表查看所有，*${key}进行模糊查询（一般不推荐，因为redis是单线程，这种查询方式最费时间，容易造成消息堵塞）
DEL ${key} #删除指定的key
EXISTS ${key} #判断key是否存在
EXPIRE ${key} ${seconds} #给一个key设置有效期为seconds，有效期到期时该key会被自动删除
TTL ${key} #查看key的剩余有效期 -1代表永久有效 -2代表已过期
```

```bash
FLUSHALL #清空数据库
```

### String

字符串类型，可分为三类：

- string：普通字符串
- int：整数类型，可以做自增、自减操作
- float：浮点类型，可以做自增、自减操作

底层都是以字节数组形式存储，只是编码方式不同，字符串类型的最大空间不能超过512m

**基础命令**

```bash
SET ${key} ${value}	#添加或修改String类型的键值对
GET ${key} #根据key获取String类型的value
MSET ${key} ${value} ${key1} ${value1}...	#添加或修改多个...
MGET ${key} ${key1}...	#根据多个key获取多个String类型的value
```

**数值命令**

```bash
INCR ${key} #让一个整型的key自增1
INCRBY ${key} ${increment} #让一个整型的key自增并指定步长
INCRBYFLOAT ${key} ${increment} #让一个浮点型的key自增并指定步长
```

**组合命令**

```bash
SETNX ${key} ${value} #添加一个String类型的键值对，前提是这个key不存在，否则不会执行(分布式锁)
SETEX ${key} ${seconds} ${value} #添加key，value并且设置有效期
```

**key的层级格式**

层级间以 : 分割，类似分成不同的包

```bash
SET user:1 '{"id":1,"name":"xwy","age":25}'
SET product:1 '{"id":1,"name":"iphone15","price":6999}'
```

可视化工具中效果如下



<img src="https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231006134153027.png" alt="image-20231006134153027" style="zoom:50%;" />

### Hash

散列，value是一个无序字典，类似Java中的HashMap结构，{key：{(key1，value1), (key2, value2)}

**基础命令**

```bash
HSET ${key} ${field} ${value}	#添加或修改hash类型key下指定field的值
HGET ${key} ${field} #根据key获取hash类型的value
HMSET ${key} ${field1} ${value1} ${field2} ${value2}...	#添加或修改多个...
HMGET ${key} ${field1} ${value1} ${field2} ${value2}...	#根据一个key多个field获取多个hash类型的value
HGETALL ${key} #获取key下的所有键
HKEYS ${key} #获取key下的所有值
HDEL ${key} ${field} #删除指定key下field的值
```

**数值命令**

```bash
HINCRBY ${key} ${field} ${increment} #让一个整型的key自增并指定步长
INCRBYFLOAT ${key} ${field} ${increment} #让一个浮点型的key自增并指定步长
```

**组合命令**

```bash
SETNX ${key} ${field} ${value} #添加一个hash类型的指定key下的field的value，前提是这个key和field不存在，否则不会执行
```

### List

与Java中的LinkedList类似，可以看作是一个**双向链表**结构，即可以支持正向检索也可以支持反向检索。

**特点**：

- 有序
- 元素可以重复
- 插入和删除快
- 查询速度一般

**使用场景**：

保存对顺序有要求的数据，例如朋友圈点赞评论，有一个点赞评论展示的先后顺序。

**基础命令**

```bash
LPUSH ${key} ${element} #从列表左侧插入一个或多个元素
LPOP ${key} #移除并返回列表左侧的第一个元素（默认为1，可以后面跟数字来指定移除的数量），没有则返回nil
RPUSH ${key} ${element} #从列表右侧插入一个或多个元素
RPOP ${key} #移除并返回列表右侧的第一个元素，没有则返回nil
LRANGE ${key} ${start} ${end} #返回一段角标范围内的所有元素
BPOP ${key} ${timeout} #移除并返回列表左侧的第一个元素，没有先等待指定的时间，时间结束还是没有就返回nil(阻塞式获取)
BRPOP ${key} ${timeout} #移除并返回列表右侧的第一个元素，没有先等待指定的时间，时间结束还是没有就返回nil(阻塞式获取)
```

利用List结构模拟一个**栈**：

入口和出口在同一边，LPUSH+LPOP、RPUSH+RPOP

利用List结构模拟一个**队列**：

入口和出口在不同边，LPUSH+RPOP、RPUSH+LPOP

利用List结构模拟一个**阻塞队列**：

入口和出口在不同边，出队时使用阻塞式，LPUSH+BRPOP、RPUSH+BLPOP

### Set

与Java中的HashSet类似，可以看作时一个value为null的HashMap，具备与HashSet类似的特征。

- 无序
- 元素不可重复
- 查找快
- **支持交集、并集、差集等功能**（好友列表，共同好友等）

**基础命令**

```bash
SAAD ${key} ${member} #向set中添加一个或多个元素
SREM ${key} ${member} #移除set中的指定元素
SCARD ${key} #返回set中元素的个数
SISMEMBER ${key} ${member} #判断一个元素是否存在于set中
SMEMBERS ${key} #获取set中的所有元素
```

**特殊命令**

```bash
SINTER ${key1} ${key2} #求key1和key2的交集
SDIFF ${key1} ${key2} #求key1和key2的差集,不在key2里但在key1里的元素
SUNION ${key1} ${key2} #求key1和key2的并集，由于是不重复的所以交集部分只会记录一次
```

### SortedSet

一种可排序集合，与Java中的TreeSet类似，但底层数据结构却差别很大。Java中是红黑树实现的，SortedSet中的每一个元素都带有一个score属性，基于score对元素进行排序，底层的实现是一个跳表（SkipList）+hash表。

**特点**：

- 可排序
- 元素不重复
- 查询速度快

**使用场景**：

例如排行榜的功能。

**常见命令**

```bash
ZADD ${key} ${score} ${member} #添加一个或多个元素到sorted set,如果已经存在则更新其score值
ZREM ${key} ${member} #删除sorted set中的一个指定元素
ZSCORE ${key} ${member} #获取sorted set中的指定元素的score值
ZRANK ${key} ${member} #获取sorted set中的指定元素的排名，升序
ZREVRANK ${key} ${member} #获取sorted set中的指定元素的排名，降序
ZCARD ${key} #获取sorted set中的元素个数
ZCOUNT ${key} ${min} ${max} #统计score值在给定范围内的所有元素的个数
ZINCRBY ${key} ${increment} ${member} #让sorted set中的指定元素自增，步长为指定的increment值
ZRANGE ${key} ${min} ${max} #按照score排序后，获取指定下标范围内的元素，升序
ZREVRANGE ${key} ${min} ${max} #按照score排序后，获取指定下标范围内的元素，降序
ZRANGEBYSCORE ${key} ${min} ${max} #按照score排序后，获取指定score范围内的元素
ZINTER ${key1} ${key2} #求key1和key2的交集
ZDIFF ${key1} ${key2} #求key1和key2的差集,不在key2里但在key1里的元素
ZUNION ${key1} ${key2} #求key1和key2的并集，由于是不重复的所以交集部分只会记录一次
```

## Redis的Java客户端

- **Jedis**：以Redis命令作为方法名称，学习成本低，简单实用。但是]edis实例是线程不安全的，多线程环境下需要基于连接池来使用。

- **Lettuce**：基于Netty实现的，支持同步、异步和响应式编程方式，并且是线程安全的。支持Redis的哨兵模式、集群模式和管道模式。

- Redisson：基于Redis实现的分布式、可伸缩的Java数据结构集合。包含了诸如Map、Queue、Lock、Semaphore、AtomicLong等强大功能。

Spring中的**Spring Data Redis** API兼容jedis和Lettuce.

### Jedis

[官方文档][https://github.com/redis/jedis]，jedis操作的封装命名与redis内的命令一致，所以很容易上手。

#### Java连接Redis

新建maven项目，添加依赖

```xml
<dependencies>
    <!--jedis-->
    <dependency>
        <groupId>redis.clients</groupId>
        <artifactId>jedis</artifactId>
        <version>5.0.0</version>
    </dependency>
    <!--junit单元测试-->
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>RELEASE</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

新建测试类

```java
public class JedisTest {
    private Jedis jedis;

    @BeforeEach
    void setUp() {
        jedis = new Jedis("127.0.0.1",6379);
        jedis.auth("123456");
        jedis.select(0);
    }

    @Test
    void testString() {
        String res = jedis.set("name","xwy");
        System.out.println(res);
        String name = jedis.get("name");
        System.out.println(name);
    }

    @Test
    void testHash() {
        jedis.hset("user:1","name","xwy");
        jedis.hset("user:1","age","25");

        Map<String, String> map = jedis.hgetAll("user:1");
        System.out.println(map);
    }

    @AfterEach
    void tearDown() {
        if (jedis!=null){
            jedis.close();
        }
    }
}
```

#### Jedis线程池

Jedis本身式线程不安全的，在多线程的情况下，并发的访问容易出现线程安全问题，在这种情况下需要给每个线程创建独立的Jedis对象，但频繁的创建和销毁对象会有性能损耗，因此通常会使用Jedis连接池代替Jedis的直连方式。

```java
public class JedisConnectionFactory {
    private static final JedisPool jedisPool;

    static{
        //配置连接池
        JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
        //最大为8个连接
        jedisPoolConfig.setMaxTotal(8);
        //最大空闲连接
        jedisPoolConfig.setMaxIdle(8);
        //最小空闲连接
        jedisPoolConfig.setMinIdle(0);
        //无空闲连接是的等待时长 默认为-1：无限等待
        jedisPoolConfig.setMaxWaitMillis(1000);

        jedisPool = new JedisPool(jedisPoolConfig,"127.0.0.1",6379,
                1000,"123456");
    }

    public static Jedis getJedis(){
        return jedisPool.getResource();
    }

}
```

```java
jedis = JedisConnectionFactory.getJedis(); #获取线程池中的Jedis对象
```

### SpringDataRedis

SpringData是Spring中数据操作的模块，包含对各种数据库的集成，其中对Redis的集成模块就叫做[SpringDataRedis][https://spring.io/projects/spring-data-redis].

- 提供了对不同Redis客户端的整合（Jedis和Lettuce）
- 提供了RedisTemplate统一API来操作Redis
- 支持Redis的发布订阅模型
- 支持Redis哨兵和Redis集群
- 支持基于Lettuce的响应式编程
- 支持基于JDK、JSON、字符串、Spring对象的数据序列化及反序列化
- 支持基于Redis的JDKCollection实现

SpringDataRedis中提供了RedisTemplate工具类，其中封装了各种对Redis的操作。并且将不同数据类型的操作API封装到了不同的类型中。

| API                         | 返回值类型      | 说明                   |
| --------------------------- | --------------- | ---------------------- |
| redisTemplate.opsForValue() | ValueOperations | 操作String类型数据     |
| redisTemplate.opsForHash()  | HashOperations  | 操作Hash类型数据       |
| redisTemplate.opsForList()  | ListOperations  | 操作List类型数据       |
| redisTemplate.opsForSet()   | SetOperations   | 操作Set类型数据        |
| redisTemplate.opsForzSet()  | ZSetOperations  | 操作SortedSet:类型数据 |
| redisTemplate               |                 | 通用的命令             |

SpirngBoot已经提供了对SpringDataRedis的支持

1.引入依赖

```xml
<!--Redis依赖-->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<!--连接池依赖-->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>
```

2.yml配置

```yaml
spring:
  redis:
    host: 127.0.0.1
    port: 6379
    password: 123456
    lettuce:
      pool:
        max-active: 8
        max-idle: 8
        min-idle: 0
        max-wait: 1000
```

3.单元测试

```java
@SpringBootTest
class RedisDemoApplicationTests {

    //注入RedisTemplate
    @Autowired
    private RedisTemplate redisTemplate;

    @Test
    void testString() {
        redisTemplate.opsForValue().set("name","xwySpring");
        Object object = redisTemplate.opsForValue().get("name");
        System.out.println(object);
    }

}
```

redisTemplate默认会将存进去的值当做object处理，底层默认会使用jdk的序列化工具进行序列化

![image-20231007101348539](https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231007101348539.png)

RedisTemplate.class

```java
if (this.defaultSerializer == null) {
    this.defaultSerializer = new JdkSerializationRedisSerializer(this.classLoader != null ? this.classLoader : this.getClass().getClassLoader());
}
```

断点调试可以发现最终调用的就是jdk的ObjectOutputStream序列化方法将object转换为了字节流输出

Serializer.class

```java
default byte[] serializeToByteArray(T object) throws IOException {
    ByteArrayOutputStream out = new ByteArrayOutputStream(1024);	//缓冲
    this.serialize(object, out);	//序列化
    return out.toByteArray();
}
```

DefaultSerializer.class

```java
public void serialize(Object object, OutputStream outputStream) throws IOException {
    if (!(object instanceof Serializable)) {
        throw new IllegalArgumentException(this.getClass().getSimpleName() + " requires a Serializable payload but received an object of type [" + object.getClass().getName() + "]");
    } else {
        ObjectOutputStream objectOutputStream = new ObjectOutputStream(outputStream);
        objectOutputStream.writeObject(object);
        objectOutputStream.flush();
    }
}
```

因此，这种默认的序列化方式一方面可读性很差，另一方面内存占用较大。

通过查看RedisSerializer序列化的实现方法可以发现，默认采用的是JdkSerializationRedisSerializer，StringRedisSerializer是专门用于处理字符串的，直接使用getBytes的方式 ，由于一般key都是字符串，所以一般使用这种方式，GenericJackson2JsonRedisSerializer是转json字符串的方式，所以一般用于value的序列化。

![image-20231007104456540](https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231007104456540.png)



```java
@Configuration
public class RedisConfig {

    @Bean
    public RedisTemplate<String,Object> redisTemplate(RedisConnectionFactory connectionFactory){
        // 创建RedisTemplate对象
        RedisTemplate<String,Object> template = new RedisTemplate<>();
        // 设置连接工厂
        template.setConnectionFactory(connectionFactory);
        // 创建JSON序列化工具
        GenericJackson2JsonRedisSerializer jsonRedisSerializer = new GenericJackson2JsonRedisSerializer();
        // 设置Key的序列化
        template.setKeySerializer(RedisSerializer.string());
        template.setHashKeySerializer(RedisSerializer.string());
        // 设置Value的序列化
        template.setValueSerializer(jsonRedisSerializer);
        template.setValueSerializer(jsonRedisSerializer);

        // 返回
        return template;
    }
}
```

需要引入依赖

```xml
<!--Jackson依赖-->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
</dependency>
```

这种方式可以完成自动反序列化操作，但由于自动反序列化需要存@class属性（反射机制），会占用一定的内存空间。

![image-20231007132335462](https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231007132335462.png)

为了节省内存空间，一般不会使用JSON序列化器来处理value，而是统一使用String序列化器，要求只能存储String类型的key和value。当需要存储Java对象时，手动完成对象的序列化和反序列化。

```java
@SpringBootTest
public class RedisStringTests {

    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    private static final ObjectMapper objectMapper = new ObjectMapper();

    @Test
    void testString() {
        stringRedisTemplate.opsForValue().set("name","xwy111");
        Object name = stringRedisTemplate.opsForValue().get("name");
        System.out.println(name);
    }

    @Test
    void testStringUser() throws JsonProcessingException {
        User user = new User("xwy",24);
        // 手动序列化
        String json = objectMapper.writeValueAsString(user);

        stringRedisTemplate.opsForValue().set("student",json);
        String o = stringRedisTemplate.opsForValue().get("student");

        //手动反序列化
        User user1 = objectMapper.readValue(o, User.class);
        System.out.println(user1);
        
    }
    
    @Test
    void testHash() throws JsonProcessingException {
        stringRedisTemplate.opsForHash().put("user:300","name","xwy");
        stringRedisTemplate.opsForHash().put("user:300","age","25");

        Map<Object, Object> entries = stringRedisTemplate.opsForHash().entries("user:300");
        System.out.println(entries);

        Map<String, String> map = new HashMap<>();
        User user = new User("xwy",24);
        String s = objectMapper.writeValueAsString(user);
        map.put("number:001",s);
        stringRedisTemplate.opsForHash().putAll("student:100",map);
        Map<Object, Object> entries1 = stringRedisTemplate.opsForHash().entries("student:100");

        System.out.println(entries1.get("number:001"));

    }
}
```



## 项目实战

![image-20231007150155102](https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231007150155102.png)





### 缓存机制

将数据库中的部分数据写入到缓存中，提高系统响应时间。



### 缓存更新策略

|          |                           内存淘汰                           |                           超时剔除                           |                   主动更新                   |
| :------: | :----------------------------------------------------------: | :----------------------------------------------------------: | :------------------------------------------: |
|   说明   | 不用自己维护，利用Redis的内存淘汰机制，当内存不足时自动淘汰部分数据，下次查询时更新缓存。 | 给缓存数据添加TTL时间，到期后自动删除缓存。下次查询时更新缓存。 | 编写业务逻辑，在修改数据库的同时，更新缓存。 |
|  一致性  |                              差                              |                             一般                             |                      好                      |
| 维护成本 |                              无                              |                              低                              |                      高                      |

根据业务场景采用不同策略：

- 低一致性：使用内存淘汰机制，如店铺类型的查询缓存。
- 高一致性：主动更新，并以超时剔除作为兜底方案，如店铺详情查询的缓存。

#### 主动更新策略

- Cache Aside Pattern：由缓存的调用者，在更新数据库的同时更新缓存。
- Read/Write Through Pattern：缓存与数据库整合为一个服务，由服务来维护一致性。调用者调用该服务，无需关心缓存一致性问题。
- Write Behind Caching Pattern：调用者只操作缓存，由其他线程异步的将缓存数据持久化到数据库，保证最终一致。



**Cache Aside Pattern**

操作缓存和数据库时有三个问题需要考虑：

1. 删除缓存还是更新缓存？
   更新缓存：每次更新数据库都更新缓存，无效写操作较多	X
   删除缓存：更新数据库时让缓存失效，查询时再更新缓存	√
2. 如何保证缓存与数据库的操作的同时成功或失败？
   单体系统，将缓存与数据库操作放在一个事务
   分布式系统，利用TCC等分布式事务方案
3. 先操作缓存还是先操作数据库？
   先删除缓存，再操作数据库。
   先操作数据库，再删除缓存。

![image-20231012132857961](https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231012132857961.png)

缓存更新策略的最佳实践方案：
1.低一致性需求：使用Redis自带的内存淘汰机制
2.高一致性需求：**主动更新，并以超时剔除作为兜底方案**
**读操作：**
缓存命中则直接返回
缓存未命中则查询数据库，并写入缓存，设定超时时间
**写操作：**
·先写数据库，然后再删除缓存
·要确保数据库与缓存操作的原子性



### 缓存穿透

缓存穿透是指客户端请求的数据在缓存中和数据库中都不存在，这样缓存永远不会生效，这些请求都会打到数据库。

常见的解决方案有两种：

- 缓存空对象
  - 优点：实现简单，维护方便
  - 缺点：
    - 额外的内存消耗
    - 可能造成短期的不一致（也可以新增的时候主动更新到缓存中）

<img src="https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231012152824728.png" alt="image-20231012152824728" style="zoom:50%;" />

- 布隆过滤
  - 优点：内存占用较少，没有多余key
  - 缺点：
    - 实现复杂
    - 存在误判可能（不放行的数据肯定不存在，放行的数据可能存在也可能不存在）

<img src="https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231012153341611.png" alt="image-20231012153341611" style="zoom:50%;" />

缓存穿透产生的原因是什么？

- 用户请求的数据在缓存中和数据库中都不存在，不断发起这样的请求给数据库带来巨大压力

缓存穿透的解决方案有哪些？

- 被动的方式

  - 缓存null值

  - 布隆过滤

- 主动的方式

  - 增强id的复杂度，避免被猜测id规律

  - 做好数据的基础格式校验

  - 加强用户权限校验

  - 做好热点参数的限流



### 缓存雪崩

**缓存雪崩**是指在同一时段大量的缓存key**同时失效**或者Redis服务**宕机**，导致大量请求到达数据库，带来巨大压力。

<img src="https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231012155838713.png" alt="image-20231012155838713" style="zoom: 50%;" />

针对大量key同时失效的情况

- 通过给不同的key设置随机TTL来解决。

针对服务宕机问题

- 利用Redis集群提高服务的高可用
- 给缓存业务添加降级限流策略 （微服务）
- 给业务添加多级缓存（微服务）



### 缓存击穿

**缓存击穿**问题也叫**热点Key**问题，就是一个被高并发访问并且**缓存重建业务较复杂**的key突然**失效了**，无数的请求访问会在瞬间给数据库带来巨大的冲击。**（注意：是key已经失效而带来的问题，没失效就说明不是缓存击穿问题）**

<img src="https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231012162121583.png" alt="image-20231012162121583" style="zoom:50%;" />

#### 互斥锁

<img src="https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231012162353902.png" alt="image-20231012162353902" style="zoom:50%;" />

性能会低，除了重建缓存数据的线程，别的线程都处于反复重试的过程中。

- 优点：
  - 没有额外的内存消耗（逻辑过期需要存入额外字段）
  - 保持一致性（互斥锁，不存在旧数据，别的线程只能等待新数据的写入）
  - 实现简单
- 缺点：
  - 线程需要等待，性能受影响
  - 可能有死锁风险（业务1获取缓存A的锁，接着获取缓存B的锁，但此时有业务2已经获取缓存B的锁，在获取缓存A的锁。由于获取互斥锁失败时会重试，也就产生死锁。）

#### 逻辑过期

<img src="https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231012163100851.png" alt="image-20231012163100851" style="zoom:50%;" />

不设置实际的TTL过期时间，通过写入一个新的字段来判断缓存中的数据是否真的过期了。

- 优点：
  - 线程无需等待，性能较好
- 缺点：
  - 不保证一致性（获取互斥锁失败时会直接返回旧的数据）
  - 有额外内存消耗（存储了代表过期状态的额外字段）
  - 实现复杂

<img src="https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231012215838095.png" alt="image-20231012215838095" style="zoom:50%;" />



#### CAP定理

一致性（C）：在分布式系统中的所有数据备份，在同一时刻是否同样的值。（等同于所有节点访问同一份最新的数据副本）

可用性（A）：保证每个请求不管成功或者失败都有响应。

分区容忍性（P）：系统中任意信息的丢失或失败不会影响系统的继续运作。

要么AP要么CP



### 优惠券秒杀

#### 全局ID生成器

当用户抢购时，会生成订单并保存到tb_voucher_order表中，如果使用传统的自增id会带来一些问题：

- id的规律性太明显
- 受单表数据量的限制

在分布式系统下生成全局唯一ID需要满足以下特性：

唯一性：订单id必须唯一

高可用：任何时候都能保证可以生成正确的id

高性能：速度要快

递增行：尽量单调递增，有利于数据库创建索引

安全性：在自增数值的基础上拼接一些额外信息



<img src="https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231014164010383.png" alt="image-20231014164010383" style="zoom:50%;" />

ID的组成部分：
符号位：1bit，永远为0
时间戳：31bit，以秒为单位，可以使用69年
序列号：32bit，秒内的计数器，支持每秒产生$$2^{32}$$个不同ID

全局唯一ID生成策略：

- UUID
- Redis自增
- snowflake算法

- 数据库自增

Redis自增ID策略：

- 每天一个key，方便统计订单量
- ID构造是时间戳+计数器

##### Redis自增

```java
public class RedisIdCreator {

    /**
     * 开始时间戳
     */
    private static final long BEGIN_TIME = 1672531200L;

    /**
     * 序列号的位数
     */
    private static final int COUNT_BITS = 32;

    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    public long nextId(String keyPrefix) {
        // 1.生成31位时间戳
        LocalDateTime now = LocalDateTime.now();
        long nowEpochSecond = now.toEpochSecond(ZoneOffset.UTC);
        long timestamp = nowEpochSecond - BEGIN_TIME;

        // 2.生成序列号
        // 以天为单位作为key，1.方便统计当天数据 2.防止超过存储上限
        String date = now.format(DateTimeFormatter.ofPattern("yyyy:MM:dd"));
        long increment = stringRedisTemplate.opsForValue().increment("icr:" + keyPrefix + ":" + date);

        // 3.返回全局id
        return timestamp << COUNT_BITS | increment;
    }
}
```



#### 优惠券秒杀下单

表关系如下：

- tb_voucher：优惠券的基本信息，优惠金额、使用规则等

- tb seckill voucher：优惠券的库存、开始抢购时间，结束抢购时间。特价优惠券才需要填写这些信息

下单时需要判断两点：

- 秒杀是否开始或结束，如果尚未开始或结束则无法下单
- 库存是否充足，不足则无法下单



#### 超卖问题

多线程下，会出现线程安全问题，在判断库存和实际修改库存之间，容易出现线程穿插执行的问题，导致最终的库存扣减过多的现象。

**悲观锁**
认为线程安全问题一定会发生，因此在操作数据之前先获取锁，确保线程**串行**执行。

- 例如Synchronized、Lock都属于悲观锁，以及之前用到的互斥锁。

**乐观锁**
认为线程安全问题不一定会发生，因此不加锁，只是在更新数据时去判断有没有其它线程对数据做了修改。

- 如果没有修改则认为是安全的，自己才更新数据。
- 如果**已经被其它线程修改**说明发生了安全问题，此时可以重试或异常。

#### 乐观锁的实现方法

乐观锁的关键是判断之前查询得到的数据是否有被修改过，常见的方式有两种：

**版本号法：**

|  id  | stock | version |
| :--: | :---: | :-----: |
|  10  |   1   |    1    |

新增一个版本号字段，修改的时候版本号+1，同时查询条件语句里需要判断一下版本号是否等于之前获取到的。相同则代表这段时间内没有进行过修改，不同则代表这段时间内有别的线程进行了修改。

update语句会对操作的数据加锁。

**CAS法（compare and set）：**

直接拿库存做判断，判断当前库存是否等于之前获取到的库存。

但这种判断的方式出现了少卖的现象，库存100 ，200个用户同时抢购，最终库存还剩余78。但继续增加并发量，可以保证库存始终>=0，确实保证了线程安全问题。

因此可以采用直接判断库存是否＞0来进行修改。



#### 一人一单

一个用户只能抢购一张秒杀优惠券。

```java
@Transactional
public Result createVoucherOrder(Long id, SeckillVoucher seckillVoucher) {
    // TODO 一人一单
    // 获取当前用户id
    Long uid = UserThreadLocal.getUser().getId();
    // toString()底层是new一个String对象，因此每次获取的锁都不一样。用intern返回与当前值相同的String对象
    // 这样就只会锁住当前用户，不会锁住其他的用户，
    synchronized (uid.toString().intern()) {
        // 查询库存订单
        int query = voucherOrderMapper.queryByIds(uid, seckillVoucher.getVoucherId());
        // 判断是否存在
        if (query > 0) {
            // 用户已经购买过了
            return Result.fail("用户已经购买过一次");
        }

        // 4.扣减优惠券库存
        Boolean success = seckillVoucherMapper.updateStockById(id, seckillVoucher.getStock());
        if (!success) {
            return Result.fail("库存不足");
        }
        // 5.创建订单
        long orderId = redisIdCreator.nextId(VOUCHER_ORDER_ID);
        UserVO user = UserThreadLocal.getUser();
        long userId = user.getId();
        voucherOrderMapper.insertOrder(orderId, userId, seckillVoucher.getVoucherId());
        // 6.返回订单id
        return Result.ok(orderId);
    }
}
```

锁加在方法内部会导致在事务提交之前就已经释放了，依然会存在线程安全问题，因此需要将锁的范围扩大到事务范围之外。

```java
// 获取当前用户id
Long uid = UserThreadLocal.getUser().getId();
// toString()底层是new一个String对象，因此每次获取的锁都不一样。用intern返回与当前值相同的String对象
// 这样就只会锁住当前用户，不会锁住其他的用户，
synchronized (uid.toString().intern()) {
    return this.createVoucherOrder(id, seckillVoucher);
}
```

这种方式调用createVoucherOrder方法时获取到的对象是当前的实现类对象，而不是代理对象。而事务要想生效，是Spring对当前的实现类做了动态代理，拿到的是代理对象，通过代理对象对事务进行处理。 

因此通过this.createVoucherOrder()调用时，其事务功能会失效。 

```java
@Resource
@Lazy
private IVoucherOrderService voucherOrderService;
synchronized (uid.toString().intern()) {
    
    return voucherOrderService.createVoucherOrder(uid, seckillVoucher);
}
```

可以通过注入自己来获取代理对象的createVoucherOrder方法，使事务生效。

```xml
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
</dependency>
```

```java
@EnableAspectJAutoProxy(exposeProxy = true)
```

```java
synchronized (uid.toString().intern()) {
    // 拿到当前类的代理对象
    IVoucherOrderService proxyObject = (IVoucherOrderService) AopContext.currentProxy();
    return proxyObject.createVoucherOrder(uid, seckillVoucher);
}
```

导包，然后在项目启动入口添加注解，暴露代理对象，然后通过获取这个代理对象来调用其方法。



然而，在**集群模式**下，依然会出现锁不住的问题，因为是在**两台JVM**中执行的操作，因此需要一个能够跨JVM的锁。

#### 分布式锁

满足分布式系统或集群模式下，多进程可见并且互斥的锁。

![image-20231016204456034](https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231016204456034.png)

#### 基于Redis的分布式锁

获取锁：SETNX+TTL

释放锁：手动DEL释放

获取锁和添加过期时间两个操作需要具有原子性。

**误删问题**：在存入锁的时候用UUID+线程id的方式来作为value，当作一个标识符，在删除前取出判断当前获取到的和最开始存进去的是否相同，不同则代表是别的进程的锁。



基于setnx实现的分布式锁存在下面的问题：

**不可重入：**同一个线程无法多次获取同一把锁

**不可重试：**获取锁只尝试一次就返回false，没有重试机制

**超时释放：**锁超时释放虽然可以避免死锁，但如果业务执行耗时较长，也会导致锁释放，存在安全隐患（如执行到中途，在更新库存前的判断已经通过，但遇到了阻塞，在此期间锁过期了，这时候会有另一个线程获取到锁开始执行业务，从而导致一人一单业务出现问题。）

**主从一致性：**主从复制+读写分离。如果redis提供了主从集群，主从同步存在延迟，当主宕机时， 



#### Redisson

Redisson是一个在Redis的基础上实现的Java驻内存数据网格(In-Memory Data Grid)。它不仅提供了一系列的分布式的ava常用对象，还提供了许多分布式服务，其中就包含了各种分布式锁的实现。

官网地址：https://redisson.org
GitHub地址：https://github.com/redisson/redisson



##### 锁的重入

方法1和2都需要获取锁，当方法1调方法2时就会出现方法2无法获取锁。

解决方法：采用hash结构额外存入一个count值，获取相同锁时count+1，释放锁时count-1，当count为0时代表业务结束了，就可以释放锁。

<img src="https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231017150344851.png" alt="image-20231017150344851" style="zoom:50%;" />

tryLock的lua脚本执行过程

```lua
local key = KEYS[1]; --锁的key
local threadId = ARGV[1]; --线程唯-标识
local releaseTime = ARGV[2]; --锁的自动释放时间
-- 判断是否存在
if(redis.call('exists', key)==0) then
    -- 不存在，获取锁
    redis.call('hset', key, threadId,'1');
    -- 设置有效期
    redis.call('expice', key, releaseTime);
    return 1; -- 返回结果
end;
-- 锁已经存在，判断threadId是否是自己
if(redis.call('hexists', key, threadId) == 1) then
    -- 存在，获取锁，重入次数+1
    redis.call('hincrby', key, threadId,'1');
    -- 设置有效期
    redis.call('expire', key, releaseTime);
    return 1; -- 返回结果
end
return 0；--代码走到这里，说明获取锁的不是自己，获取锁失败
```

unlock的lua脚本执行过程

```lua
local key = KEYS[1]; --锁的key
local threadId = ARGV[1]; --线程唯一标识
local releaseTime = ARGV[2]; --锁的自动释放时间
--一判断当前锁是否还是被自己持有
if (redis.call('HEXISTS',key,threadId)==0) then
	return nil; --如果已经不是自己，则直接返回
end;
-- 是自已的锁,则重入次数-1
local count redis.call('HINCRBY',key, threadId,-1);
-- 一判断是否重入次数是否已经为0
if (count >0) then
    --大于0说明不能释放锁，重置有效期然后返回
    redis.call('EXPIRE',key,releaseTime);
    return nil;
else
	--等于0说明可以释放锁，直接删除
    redis.call('DEL',key);
    return nil;
end;
```



##### 锁重试和看门狗（watchDog）机制

借助netty的定时任务进行实现。

 <img src="https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231017161523595.png" alt="image-20231017161523595" style="zoom:50%;" />

redisson分布式锁原理：

- 可重入：利用hash结构记录线程id和重入次数
- 可重试：利用信号量和PubSub功能实现等待、唤醒、获取锁失败的重试机制
- 超时续约：利用WatchDog，每隔一段时间(releaseTime/3)，重置超时时间TTL（解决了之前遇到的极端情况下由于线程阻塞导致一人可以通过不同进程抢购多单的问题。）



##### 主从一致性问题

锁失效问题

<img src="https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231017162429696.png" alt="image-20231017162429696" style="zoom: 33%;" />

**MultiLock联锁**：只有在所有节点处都获得锁才算获取成功，因此，宕机一个节点并不会导致主从不一致。

<img src="https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231017162706631.png" alt="image-20231017162706631" style="zoom:33%;" />



**不可重入Redis分布式锁：**

- 原理：利用setnx的互斥性；利用expire避免死锁；释放锁时判断线程标示
- 缺陷：不可重入、无法重试、锁超时失效

**可重入的Redis分布式锁：**

- 原理：利用hash结构，记录线程标示和重入次数；利用watchDog延续锁时间；利用信号量控制锁重试等待
- 缺陷：redis宕机引起锁失效问题

**Redisson multiLock**:

- 原理：多个独立的Redis节点，必须在所有节点都获取重入锁，才算获取锁成功
- 缺陷：运维成本高、实现复杂

#### Redis秒杀优化

<img src="https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231018121809436.png" alt="image-20231018121809436" style="zoom:50%;" />

需求：

- 新增秒杀优惠券的同时，将优惠券信息保存到Redis中
- 基于Lua脚本，判断秒杀库存、一人一单，决定用户是否抢购成功
- 如果抢购成功，将优惠券id和用户id封装后存入阻塞队列
- 开启线程任务，不断从阻塞队列中获取信息，实现异步下单功能

**秒杀优化思路：**

将线性执行改为异步执行，将抢单和下单业务分离，先利用redis完成库存余量、一人一单的判断，完成抢单业务；之后再将下单业务放入阻塞队列中，开启一个独立的线程异步完成下单操作，扣减数据库中的库存并且写入订单信息。

**基于阻塞队列的异步秒杀存在哪些问题：**

阻塞队列是基于JVM的，高并发情况下可能会导致内存溢出的问题。

由于是基于内存保存的抢单信息，如果突然宕机会导致数据丢失问题；以及从阻塞队列中取出数据后该条数据就无法再次从中获得，如果此时后续的执行出现了异常，如插入失败，库存扣减失败等会导致该订单永远丢失，无法再次进行处理。



#### Redis消息队列实现异步秒杀

消息队列(**M**essage **Q**ueue)，字面意思就是存放消息的队列。

最简单的消息队列模型包括3个角色：

- 消息队列：存储和管理消息，也被称为消息代理(Message Broker)

- 生产者：发送消息到消息队列

- 消费者：从消息队列获取消息并处理消息

Redis提供了三种不同的方式来实现消息队列：

- List结构：基于List结构模拟消息队列

- PubSub：基本的点对点消息模型

- Stream：比较完善的消息队列模型

##### 基于List结构模拟消息队列

<img src="https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231019125926854.png" alt="image-20231019125926854" style="zoom:50%;" />



##### 基于PubSub的消息队列

**PubSub（发布订阅）**是Redis2.0版本引入的消息传递模型。顾名思义，消费者可以订阅一个或多个channel，生产者向对应channel发送消息后，所有订阅者都能收到相关消息。

- SUBSCRIBE channel[channel]：订阅一个或多个频道

- PUBLISH channel msg：向一个频道发送消息

- PSUBSCRIBE pattern[pattern]：订阅与pattern格式匹配的所有频道消费者



<img src="https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231019132814648.png" alt="image-20231019132814648" style="zoom:50%;" />

**优点：**

- 采用发布订阅模型，支持多生产、多消费

**缺点：**

- 不支持数据持久化

- 无法避免消息丢失

- 消息堆积有上限，超出时数据丢失



##### 基于Stream的消息队列的XREAD特点

- 消息可回溯
- 一个消息可以被多个消费者读取
- 可以阻塞读取
- 有消息漏读的风险

##### 基于Stream的消息队列-消费者组

消费者组：将多个消费者划分到一个组中，监听同一个队列。具备下列特点：

- 消息分流：队列中的消息会分流给组内的不同消费者，而不是重复消费，从而加快消息处理的速度
- 消息标识：消费者组会维护一个标识，记录最后一个被处理的消息，哪怕消费者宕机重启，还会从标识之后读取消息。确保每一个消息都会被消费
- 消息确认：消费者获取消息后，消息处于pedding状态，并存入一个pending-list。当处理完成后需要通过XACK来确认消息，标记消息为已处理，才会从pending-list移除。

**创建消费者组**

```
XGROUP CREATE key groupNAME ID [MKSTREAM]
```

- key：队列名称
- groupName：消费者组名称
- ID：起始ID标示，$代表队列中最后一个消息，0则代表队列中第一个消息
- MKSTREAM：队列不存在时自动创建队列

```
XREADGROUP GROUP group consumer [Count count] [BLOCK milliseconds] [NOACK] STREAMS key [key ...] ID [ID ...]
```

- group：消费组名称

- consumer：消费者名称，如果消费者不存在，会自动创建一个消费者

- count：本次查询的最大数量

- BLOCK milliseconds：当没有消息时最长等待时间

- NOACK：无需手动ACK,获取到消息后自动确认

- STREAMS key：指定队列名称

- ID：获取消息的起始ID:

  - ">"：从下一个未消费的消息开始

  - 其它：根据指定id从pending-list中获取已消费但未确认的消息，例如0，是从pending-list中的第一个消息开始

**XREADGROUP命令的特点**：

- 消息可回溯
- 可以多消费者争抢消息，加快消费速度
- 可以阻塞读取
- 没有消息漏读的风险
- 有消息确认机制，保证消息至少被消费一次



##### 基于Redis的Stream结构作为消息队列，实现异步秒杀下单

1. 创建一个Stream类型的消息队列
2. 修改之前的秒杀下单Lua脚本，在认定有抢购资格后，直接向stream.orders中添加消息，消息包括voucheId、userId、orderId
3. 项目启动时，开启一个线程任务，尝试获取stream.orders中的消息，完成下单



#### 使用RabbitMQ作为消息队列，实现异步秒杀下单





### 博文发布

#### 发布探店笔记

根据前端接口，根据blog的id返回blog信息以及对应的用户信息。



#### 点赞

需求：

- 同一个用户只能点赞一次，再次点击则取消点赞

- 如果当前用户已经点赞，则点赞按钮高亮显示（前端已实现，判断字段Blog类的isLik属性)

实现步骤：

- 给Blog类中添加一个isLike字段，标示是否被当前用户点赞

- 修改点赞功能，利用Redis的set集合判断是否点赞过，未点赞过则点赞数+1，已点赞过则点赞数-1

- 修改根据id查询Blog的业务，判断当前登录用户是否点赞过，赋值给isLike字段

- 修改分页查询Blog业务，判断当前登录用户是否点赞过，赋值给isLike字段



可以将修改数据库的操作交给MQ。



#### 点赞排行榜

按照点赞时间先后排序，返回Top5的用户。

<img src="https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231022172321132.png" alt="image-20231022172321132" style="zoom:50%;" />

in加foreach返回UserList，但此时返回的List是按id在user表中的顺序返回的，因此需要再使用``field(id, ...)``函数按传入的idList进行顺序返回结果。

循环返回一个一个User再将其add到List\<User\>中不用考虑这个顺序问题，因为range取出来的数据本身就是排好序的，但循环需要不断的执行sql语句，效率会很差。

```xml
<select id="queryForUserList" resultType="com.xwy.entity.User">
    select id, phone, password, nick_name, icon, create_time, update_time from user
    where id in
    <foreach item="id" collection="list" open="(" separator="," close=")">
        #{id}
    </foreach>
    order by field(id,<foreach item="id" collection="list" separator=",">
    #{id}
</foreach>)
</select>
```



### 好友关注

#### 关注和取关

- 利用follow表存储用户id及关注该用户的用户id。通过followId和userId查询是否存在这样一条记录，来判断当前是未关注还是已关注，返回给前端相应的标识符。

- 用户自己不能关注自己，前端页面做好判断与页面展示的同时，后端也要做一次判断。

#### 共同关注

当前用户关注的和查看另一个用户所关注的，取交集。

```java
@Override
public Result commonFollow(Long currentId) {
    UserVO user = UserThreadLocal.getUser();
    if (user == null) {
        return Result.ok(false);
    }
    String myKey = "follows:"+user.getId();
    String userKey = "follows:"+currentId;
    Set<String> intersect = stringRedisTemplate.opsForSet().intersect(myKey, userKey);
    // 判空
    if (intersect == null||intersect.isEmpty()){
        return Result.ok();
    }
    List<Long> userList = intersect.stream().map(Long::valueOf).collect(Collectors.toList());
    List<UserVO> voList = userMapper.queryForUserList(userList)
        .stream()
        .map(user1 -> BeanUtil.copyProperties(user1, UserVO.class))
        .collect(Collectors.toList());
    return Result.ok(voList);
}
```

#### 消息推送Feed流

Feed流产品有两种常见模式：

- Timeline：不做内容筛选，简单的按照内容发布时间排序，常用于好友或关注。例如朋友圈

> 优点：信息全面，不会有缺失。并且实现也相对简单

> 缺点：信息噪音较多，用户不一定感兴趣，内容获取效率低

- 智能排序：利用智能算法屏蔽掉违规的、用户不感兴趣的内容。推送用户感兴趣信息来吸引用户

> 优点：投喂用户感兴趣信息，用户粘度很高，容易沉迷

> 缺点：如果算法不精准，可能起到反作用



本例中的个人页面，是基于关注的好友来做Feed流，因此采用Timeline的模式。该模式的实现方案有三种：

- 拉模式

- 推模式

- 推拉结合

##### 拉模式

也叫读扩散，用户读取订阅的用户的发件箱消息，按时间进行排序然后展示，由于不进行保存，因此每次拉取都会重新读一遍所有的数据。

<img src="https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231023160640750.png" alt="image-20231023160640750" style="zoom:50%;" />

##### 推模式

也叫写扩散，直接发送消息到所有的粉丝收件箱中，收件箱进行排序，延迟很低，但是消息会有重复保存的问题，存储负载很大，但是读取延迟很低。

<img src="C:\Users\Aik\AppData\Roaming\Typora\typora-user-images\image-20231023161042785.png" alt="image-20231023161042785" style="zoom:50%;" />

##### 推拉结合

也叫读写混合，兼具拉和推的两种模式的优势。

> 特殊关注，普通关注

<img src="https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231023161447200.png" alt="image-20231023161447200" style="zoom:50%;" />

大V发消息时只向自己的发件箱与所有的活跃粉丝发送消息，这样，活跃粉丝可以直接送自己的收件箱里获取消息，普通粉丝需要从大V的发件箱里拉取消息。



#### 滚动分页查询

<img src="https://hexo0.oss-cn-shanghai.aliyuncs.com/blog/img/image-20231023225155383.png" alt="image-20231023225155383" style="zoom:50%;" />
