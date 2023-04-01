---
layout: post
title: 'Java中使用Redis-RedisUtil工具类'
date: 2023-03-29
description: 'Java中使用Redis-RedisUtil工具类用'
tags: '技术'
--- 
[知识补充](#jedisKnowledgeSupplement)

[Redis与Spring集成](#redisIntegratesWithSpring)

[查看RedisUtil源码](https://github.com/whvcse/RedisUtil/blob/master/RedisUtil.java)

首先讲一下Jedis，Jedis是Redis官方推荐的Java连接开发工具。要在Java开发中使用好Redis中间件，必须对Jedis熟悉才能写成漂亮的代码

## <a id=jedisKnowledgeSupplement>Jedis知识补充</a>
### 1、Redis知识补充
Redis可以存储键与5中不同数据结构类型之间的映射，这5中数据结构类型分别是：String(字符串)、List(列表)、Set(集合)、Hash(散列)和Zset(有序集合)。

|结构类型|结构存储的值|结构的读写能力|
| -- | -- | -- |
|String|可以是字符串、整数或者浮点数|对整个字符串或者字符串的其中一部分执行操作；对象和浮点数执行自增(increment)或者自减(decrement)|
|List|一个链表，链表上的每个节点都包含了一个字符串|从链表的两端推入或者弹出元素；根据偏移量对链表进行修剪(trim)；读取单个或者多个元素；根据值来查找或者移除元素|
|Set|包含字符串的无序收集器(unorderedcollection)，并且被包含的每个字符串都是独一无二的、各不相同|添加、获取、移除单个元素；检查一个元素是否存在于某个集合中；计算交集、并集、差集；从集合里卖弄随机获取元素|
|Hash|包含键值对的无序散列表|添加、获取、移除单个键值对；获取所有键值对|
|ZSet|字符串成员(member)与浮点数分值(score)之间的有序映射，元素的排列顺序由分值的大小决定|添加、获取、删除单个元素；根据分值范围(range)或者成员来获取元素|

### 2、RedisTemplate和StringRedisTemplate
两者主要区别是他们使用的序列化类不一样，RedisTemplate使用的是JdkSeriallizationRedisSerializer，StringRedisTemplate使用的是StringRedisSeriallizer，两者的数据是不共通的。

**1.RedisTemplate:**
RedisTemplate使用的是JDK的序列化策略，向Redis存入数据会将数据先序列化成字节数组然后再存入Redis数据库，这个时候打开Redis查看的时候，你会看到你的数据不是以可读的形式展示的，而是以字节数组显示，类似下面：`\xAC\xED\x00\\x05t\x05sr\x00`。所以使用RedisTemplate可以直接把一个java对象直接存储在redis里面。

**2.StringRedisTemplate:**
StringRedisTemplate默认采用的是String的序列化策略，保存的key和value都是采用此策略序列化保存的。
StringRedisTemplate是继承RedisTemplate的，这种对redis的操作方式更优雅，因为RedisTemplate以字节数组的形式存储不利于管理，也不通用。

### 3、<a id=redisIntegratesWithSpring>Redis与Spring集成</a>
1.集成配置
```java
<bean id="poolConfig" class="redis.clients.jedis.JedisPoolConfig">
    <property name="maxIdle" value="300" />
    <property name="maxTotal" value="600" />
    <property name="maxWaitMillis" value="1000" />
    <property name="testOnBorrow" value="true" />
</bean>

<bean id="jedisConnectionFactory"
    class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
    <property name="hostName" value="127.0.0.1" />
    <property name="password" value="WangFan01!" />
    <property name="port" value="6379" />
    <property name="poolConfig" ref="poolConfig" />
</bean>

<bean id="redisTemplate" class="org.springframework.data.redis.core.StringRedisTemplate">
    <property name="connectionFactory" ref="jedisConnectionFactory" />
</bean>

<!-- 这里可以配置多个redis -->
<bean id="redisUtil" class="com.wf.ew.core.utils.RedisUtil">
    <property name="redisTemplate" ref="redisTemplate" />
```

2.使用RedisUtil工具类方法如下：
```java
@Autowired
private RedisUtil redisUtil;
```


**Jedis基本使用**
Jedis的基本使用非常简单，只需要创建Jedis对象的时候指定host，port，password即可。当然，Jedis对象有很多构造方法，都大同小异，只是对应的Redis连接的socket的参数不一样而已。

```java
    public static void main(String[] args) {
        Jedis jedis = new Jedis("127.0.0.1", 6380);
        jedis.auth("root");
        Set<String> keys = jedis.keys("*");
        for (String key : keys) {
            System.out.println(key);
        }
        jedis.close();
    }
```'
