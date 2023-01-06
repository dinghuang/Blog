---
title: Spring Data Redis与Redisson对比
date: 2019-03-26 20:47:00
tags:
    - Java
categories: Java
---

# Spring Data Redis与Redisson对比

## Spring Data Redis
[Spring Data Redis](https://spring.io/projects/spring-data-redis)是更大的Spring Data系列的一部分，可以从Spring应用程序轻松配置和访问Redis。它提供了与商店交互的低级和高级抽象，使用户免于基础设施问题。Spring Boot 从 2.0版本开始，将默认的Redis客户端Jedis替换问Lettuce。

### 特性
- 连接包作为多个Redis驱动程序/连接器的低级抽象（[Jedis](https://github.com/xetorthio/jedis)和[Lettuce](https://github.com/mp911de/lettuce)。不推荐支持[JRedis](https://github.com/alphazero/jredis)和[SRP](https://github.com/spullara/redis-protocol)。）
- [异常](https://docs.spring.io/spring-data/data-redis/docs/current/reference/html/#redis:connectors)转换到Spring的便携式数据访问异常层次结构Redis的驱动程序例外
- [RedisTemplate](https://docs.spring.io/spring-data/data-redis/docs/current/reference/html/#redis:template)，提供高级抽象，用于执行各种Redis操作，异常转换和序列化支持
- [Pubsub](https://docs.spring.io/spring-data/data-redis/docs/current/reference/html/#pubsub)支持（例如消息驱动的POJO的MessageListenerContainer）
- [Redis Sentinel](https://docs.spring.io/spring-data/data-redis/docs/current/reference/html/#redis:sentinel)和[Redis Cluster](https://docs.spring.io/spring-data/data-redis/docs/current/reference/html/#cluster)支持
- JDK，String，JSON和Spring Object / XML映射[序列化程序](https://docs.spring.io/spring-data/data-redis/docs/current/reference/html/#redis:serializer)
- 在Redis之上的JDK Collection实现
- 原子[计数器](https://docs.spring.io/spring-data/data-redis/docs/current/reference/html/#redis:support)支持classes
- 排序和流水线功能
- 专门支持SORT，SORT / GET模式和返回的批量值
- Redis [实现](https://docs.spring.io/spring-data/data-redis/docs/current/reference/html/#redis:support:cache-abstraction)了Spring 3.1缓存抽象
- 自动实现Repository接口，包括支持自定义查找程序方法@EnableRedisRepositories
- CDI对存储库的支持

### 使用
在``pom.xml``中加入
```
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-redis</artifactId>
    <version>2.1.5.RELEASE</version>
</dependency>
```
在``application.yml``中加入
```
spring:
  redis:
    database: 6  #Redis索引0~15，默认为0
    host: 127.0.0.1
    port: 6379
    password:  #密码（默认为空）
    pool:
      max-active: 8   #连接池最大连接数（使用负值表示没有限制）
      max-wait: -1ms  #连接池最大阻塞等待时间（使用负值表示没有限制）
      max-idle: 5     #连接池中的最大空闲连接
      min-idle: 0     #连接池中的最小空闲连接
    timeout: 10000ms    #连接超时时间（毫秒）
```
加入配置类
```
@Configuration
@EnableCaching
public class RedisConfiguration extends CachingConfigurerSupport {

    /**
     * RedisTemplate配置
     *
     * @param redisConnectionFactory redisConnectionFactory
     * @return RedisTemplate
     */
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        // 配置redisTemplate
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory);
        //key序列化
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        //value序列化
        redisTemplate.setValueSerializer(new GenericJackson2JsonRedisSerializer());
        redisTemplate.afterPropertiesSet();
        return redisTemplate;
    }

}

```
代码使用
```

    @Autowired
    private RedisTemplate redisTemplate;

    @Autowired
    private RedisConnectionFactory redisConnectionFactory;

    @SuppressWarnings("unchecked")
    public void test(){
        //设置键值对
        redisTemplate.opsForValue().set("test:set1", "testValue1");
        //设置键值对数组
        redisTemplate.opsForSet().add("test:set2", "asdf");
        //数据hash存入
        redisTemplate.opsForHash().put("hash1", "name1", "lms1");
        redisTemplate.opsForHash().put("hash1", "name2", "lms2");
        redisTemplate.opsForHash().put("hash1", "name3", "lms3");
        //获取
        System.out.println(redisTemplate.opsForValue().get("test:set"));
        //hash获取
        System.out.println(redisTemplate.opsForHash().get("hash1", "name1"));
        //发布、订阅消息（更多请参考 https://docs.spring.io/spring-data/data-redis/docs/current/reference/html/
        String message = "dinghuang123@gmail.com";
        byte[] msg = message.getBytes();
        byte[] channel = message.getBytes();
        redisConnectionFactory.getConnection().publish(msg, channel);
        redisTemplate.convertAndSend("hello!", "world");
    }
```
如图所示
![](https://minioapi.frp.strongsickcat.com/file/dinghuang-blog-picture/9bc4cb9fgy1g1fxturm52j219e0k2ac6.jpg)


## Redisson
[Redisson](https://redisson.org)是一个在Redis的基础上实现的Java驻内存数据网格（In-Memory Data Grid）。它不仅提供了一系列的分布式的Java常用对象，还提供了许多分布式服务。其中包括(BitSet, Set, Multimap, SortedSet, Map, List, Queue, BlockingQueue, Deque, BlockingDeque, Semaphore, Lock, AtomicLong, CountDownLatch, Publish / Subscribe, Bloom filter, Remote service, Spring cache, Executor service, Live Object service, Scheduler service) Redisson提供了使用Redis的最简单和最便捷的方法。Redisson的宗旨是促进使用者对Redis的关注分离（Separation of Concern），从而让使用者能够将精力更集中地放在处理业务逻辑上。能够完美的在云计算环境里使用，并且支持AWS ElastiCache主备版，AWS ElastiCache集群版，Azure Redis Cache和阿里云（Aliyun）的云数据库Redis版。Redisson底层采用的是Netty 框架。支持Redis 2.8以上版本，支持Java1.6+以上版本。

Redisson作为独立节点 可以用于独立执行其他节点发布到分布式执行服务 和 分布式调度任务服务 里的远程任务。
![](https://minioapi.frp.strongsickcat.com/file/dinghuang-blog-picture/9bc4cb9fgy1g1f1ntysocj21b60ycgn7.jpg)

### 特性
- 复制的Redis服务器模式（还支持[AWS ElastiCache](http://docs.aws.amazon.com/AmazonElastiCache/latest/UserGuide/Replication.html)和[Azure Redis](https://azure.microsoft.com/en-us/services/cache/)缓存）：
  -  自动主服务器更改发现
- 群集Redis服务器模式（还支持[AWS ElastiCache](http://docs.aws.amazon.com/AmazonElastiCache/latest/UserGuide/Replication.html)和[Azure Redis](https://azure.microsoft.com/en-us/services/cache/)缓存：
  - 自动主从服务器发现
  - 自动状态和拓扑更新
  - 自动插槽更改发现
- Sentinel Redis服务器模式：
  - 自动主，从和服务器发现
  - 自动状态和拓扑更新
- 掌握Slave Redis服务器模式
- 单Redis服务器模式
- 线程安全的实现
- [Reactive Streams](https://github.com/redisson/redisson/wiki/3.-operations-execution#32-reactive-way) API
- [异步](https://github.com/redisson/redisson/wiki/3.-operations-execution#31-async-way) API
- 异步连接池
- Lua脚本
- [分布式Java对象](https://github.com/redisson/redisson/wiki/6.-Distributed-objects)
Object holder，Binary stream holder，Geospatial holder，BitSet，AtomicLong，AtomicDouble，PublishSubscribe，Bloom filter，HyperLogLog
- [分布式Java集合](https://github.com/redisson/redisson/wiki/7.-Distributed-collections)
Map，Multimap，Set，List，SortedSet，ScoredSortedSet，LexSortedSet，Queue，Deque，Blocking Queue，Bounded Blocking Queue，Blocking Deque，Delayed Queue，Priority Queue，Priority Deque
- [分布式Java锁和同步器](https://github.com/redisson/redisson/wiki/8.-Distributed-locks-and-synchronizers)
Lock，FairLock，MultiLock，RedLock，ReadWriteLock，Semaphore，PermitExpirableSemaphore，CountDownLatch
- [分布式服务](https://github.com/redisson/redisson/wiki/9.-distributed-services)
远程服务，Live Object服务，Executor服务，Scheduler服务，MapReduce服务
- [Spring框架](https://github.com/redisson/redisson/wiki/14.-Integration%20with%20frameworks#141-spring-framework)
- [Spring Cache](https://github.com/redisson/redisson/wiki/14.-Integration%20with%20frameworks/#142-spring-cache)实现
-[ Spring Transaction API](https://github.com/redisson/redisson/wiki/14.-Integration-with-frameworks/#147-spring-transaction-manager)实现
- [Spring Data Redis](https://github.com/redisson/redisson/wiki/14.-Integration-with-frameworks/#148-spring-data-redis)集成
- [Spring Boot Starter](https://github.com/redisson/redisson/wiki/14.-Integration-with-frameworks/#149-spring-boot-starter)实现
- [Hibernate Cache](https://github.com/redisson/redisson/wiki/14.-Integration%20with%20frameworks/#143-hibernate-cache)实现
- [Transactions API](https://github.com/redisson/redisson/wiki/10.-Additional-features#104-transactions)
-[ XA Transaction API](https://github.com/redisson/redisson/wiki/10.-additional-features/#105-xa-transactions)实现
- [JCache API（JSR-107）](https://github.com/redisson/redisson/wiki/14.-Integration%20with%20frameworks/#144-jcache-api-jsr-107-implementation)实现
- [Tomcat会话管理器](https://github.com/redisson/redisson/wiki/14.-Integration%20with%20frameworks#145-tomcat-redis-session-manager)实现
- [Spring Session](https://github.com/redisson/redisson/wiki/14.-Integration%20with%20frameworks/#146-spring-session)实现
- [Redis流水线](https://github.com/redisson/redisson/wiki/10.-additional-features#102-execution-batches-of-commands)（命令批处理）
- 支持Android平台
- 支持自动重新连接
- 支持无法发送命令自动重试
- 支持OSGi
- 支持SSL
- 支持许多流行的编解码器（[Jackson JSON](https://github.com/FasterXML/jackson)，[Avro](http://avro.apache.org/)，[Smile](http://wiki.fasterxml.com/SmileFormatSpec)，[CBOR](http://cbor.io/)，[MsgPack](http://msgpack.org/)，[Kryo](https://github.com/EsotericSoftware/kryo)，[Amazon Ion](https://amzn.github.io/ion-docs/)，[FST](https://github.com/RuedigerMoeller/fast-serialization)，[LZ4](https://github.com/jpountz/lz4-java)，[Snappy](https://github.com/xerial/snappy-java)和JDK Serialization）
- 超过1800个单元测试

### 与spring-data-redis结合使用
``pom.xml``加入依赖
```
 <dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson-spring-data-21</artifactId>
    <version>3.10.5</version>
</dependency>
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-redis</artifactId>
    <version>2.1.5.RELEASE</version>
</dependency>
```
在``resources``文件夹添加配置文件``redisson.yml``
```
#Redisson配置
singleServerConfig:
  address: "redis://127.0.0.1:6379"
  password: null
  clientName: null
  database: 7 #选择使用哪个数据库0~15
  idleConnectionTimeout: 10000
  pingTimeout: 1000
  connectTimeout: 10000
  timeout: 3000
  retryAttempts: 3
  retryInterval: 1500
  reconnectionTimeout: 3000
  failedAttempts: 3
  subscriptionsPerConnection: 5
  subscriptionConnectionMinimumIdleSize: 1
  subscriptionConnectionPoolSize: 50
  connectionMinimumIdleSize: 32
  connectionPoolSize: 64
  dnsMonitoringInterval: 5000
  #dnsMonitoring: false

threads: 0
nettyThreads: 0
codec:
  class: "org.redisson.codec.JsonJacksonCodec"
transportMode: "NIO"
```
注册``RedissonConnectionFactory``
```
 @Configuration
 public class RedissonSpringDataConfig {
    
    @Bean
    public RedissonConnectionFactory redissonConnectionFactory(RedissonClient redisson) {
        return new RedissonConnectionFactory(redisson);
    }
    
    @Bean(destroyMethod = "shutdown")
    public RedissonClient redisson(@Value("classpath:/redisson.yml") Resource configFile) throws IOException {
        Config config = Config.fromYAML(configFile.getInputStream());
        return Redisson.create(config);
    }
    
 }
```
代码使用
```
    @Autowired
    private RedissonClient redissonClient;

    @SuppressWarnings("unchecked")
    public void test(){
        //设置键值对
        RBucket<String> keyObj = redissonClient.getBucket("k1");
        keyObj.set("v1236");
    }
```

## 结论
spring-data-redis 支持的基本能够满足对redis的操作，提供了2种客户端连接，也支持redis集群的模式。如果涉及到利用redis做分布式锁的话，redisson封装了更多的工具和基础原子对象进行操作，redisson是优先选择，其次redisson兼容了很多的框架，那么多star不是没有道理的= =。同时也可以通过redisson与RxJava结合，实现线程安全的异步任务等等。