---
title: Redis的使用小结
date: 2016-11-26 14:24:00
categories:
- 技术
tags:
- 实习
---

> 前言：之前的系统需要将数据库中的数据，作为List放到缓存里，然后由两台服务器取访问缓存。这个操作用Memcached非常之不方便，因为Memcached没有支持那么多数据结构。故上周五将redis加入到系统中，今天做个小结~

<!-- more -->

#### Redis的介绍

> 以下两段介绍 摘自 百度百科，将redis的优点说的比较清晰。

> redis是一个key-value存储系统。和Memcached类似，它支持存储的**value类型相对更多**，包括string(字符串)、list(链表)、set(集合)、zset(sorted set --有序集合)和hash（哈希类型）。这些数据类型都支持push/pop、add/remove及取交集并集和差集及更丰富的操作，而且这些操作都是原子性的。在此基础上，redis支持各种不同方式的排序。与memcached一样，为了保证效率，数据都是缓存在内存中。区别的是**redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件**，并且在此基础上实现了master-slave(主从)同步。

>Redis 是一个高性能的key-value数据库。 redis的出现，很大程度补偿了memcached这类key/value存储的不足，在部 分场合可以对关系数据库起到很好的补充作用。它提供了Java，C/C++，C#，PHP，JavaScript，Perl，Object-C，Python，Ruby，Erlang等客户端，使用很**方便**。


#### Redis的下载

>下载地址：https://github.com/MSOpenTech/redis/releases。

#### Redis 服务的安装开启

(参考 自 菜鸟教程)

Windows下

>1. cmd 进入Redis的安装目录
>2. 在指令上输入 redis-server.exe redis.windows.conf
>3. 另启一个cmd窗口，原来的不要关闭，不然就无法访问服务端了。
切换到redis目录下运行 redis-cli.exe -h 127.0.0.1 -p 6379 。

Linux 下安装
>下载地址：http://redis.io/download，下载最新文档版本。

> $ wget http://download.redis.io/releases/redis-2.8.17.tar.gz

> $ tar xzf redis-2.8.17.tar.gz

> $ cd redis-2.8.17

> $ make

>make完后 redis-2.8.17目录下会出现编译后的redis服务程序redis-server,还有用于测试的客户端程序redis-cli,两个程序位于安装目录 src 目录下：

>下面启动redis服务.

>$ cd src

>$ ./redis-server

>注意这种方式启动redis 使用的是默认配置。也可以通过启动参数告诉redis使用指定配置文件使用下面命令启动。

>$ cd src

>$ ./redis-server redis.conf

>redis.conf是一个默认的配置文件。我们可以根据需要使用自己的配置文件。
启动redis服务进程后，就可以使用测试客户端程序redis-cli和redis服务交互了。 比如：

>$ cd src

>$ ./redis-cli

>redis> set foo bar

>redis> get foo

#### 如何将Redis融入Java(Springmvc)

##### Maven依赖(新增)

    <properties>
        <spring-data-redis.version>1.5.0.RELEASE</spring-data-redis.version>
    </properties>
    
    <dependency>
		<groupId>org.springframework.data</groupId>
		<artifactId>spring-data-redis</artifactId>
		<version>${spring-data-redis.version}</version>
	</dependency>

	<dependency>
		<groupId>redis.clients</groupId>
		<artifactId>jedis</artifactId>
		<version>2.7.3</version>
	</dependency>
	
##### 新增配置文件
    
    redis.properties 用于存放redis的相关参数
    applicationContext-redis-context.xml 用于实现redis相关的注入

[redis.properties]

- redis.host=127.0.0.1
- redis.port=6379
- redis.pass=
- redis.timeout=2000
- redis.maxTotal=100
- redis.maxIdle=8
- redis.minIdle=8
- redis.masterName=
- redis.sentinels=
    
[redis.properties]


```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
      http://www.springframework.org/schema/tx
      http://www.springframework.org/schema/tx/spring-tx.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd">

       <!-- scanner redis properties  -->
    <context:property-placeholder location="redis.properties"/>

    <bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig">
        <property name="maxTotal" value="${redis.maxTotal}"/> <!-- 控制一个pool可分配多少个jedis实例 -->
        <property name="maxIdle" value="${redis.maxIdle}"/><!-- 最大能够保持idel状态的对象数 -->
        <property name="minIdle" value="${redis.minIdle}"/><!-- 最大能够保持idel状态的对象数 -->
    </bean>

    <bean id="jedisConnectionFactory"
          class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
        <constructor-arg ref="jedisPoolConfig" />
        <property name="hostName" value="${redis.host}" />
        <property name="port" value="${redis.port}" />
        <property name="password" value ="${redis.pass}" />
        <property name="timeout" value ="${redis.timeout}" />
    </bean>

    <!--<bean id="redisTemplate" class="org.springframework.data.redis.core.StringRedisTemplate">-->
    <bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
              <property name="connectionFactory"   ref="jedisConnectionFactory" />
    </bean>

</beans>
```

##### 代码结构

> 建立Service

> 注：Coupon是我的实体类 需要 implements Serializable

> RedisService是Redis服务公共接口

> AbstractRedisService 是抽象的公共实现

> RedisCouponServiceImpl 是具体实现

层次结构如下：
```
@Service("redisCouponService")
public class RedisCouponServiceImpl<String,Coupon> extends AbstractRedisService<String,Coupon> implements RedisService<String,Coupon> {


}
```

具体代码如下：

[RedisService]


```
public interface RedisService<K,V> {

    public void setKeyAndValue(K key, V value);

    public void setKeyAndValueWithTime(K key, V value, Long time, TimeUnit unit);

    public V getValue(K key);

    public void rPushList(K key, V value);

    public V lPopList(K key);

    public void delKey(K key);
}
```

[AbstractRedisService]


```
public abstract class AbstractRedisService<K, V> {

    @Autowired
    protected RedisTemplate redisTemplate;

    // 设置redisTemplate
    public void setRedisTemplate(RedisTemplate<K, V> redisTemplate) {
        this.redisTemplate = redisTemplate;
    }

    public RedisTemplate<K, V> getRedisTemplate() {
        return redisTemplate;
    }

    public void setKeyAndValue(K key, V value) {
        getRedisTemplate().opsForValue().set(key, value);
    }

     //有过期时间的set
    public void setKeyAndValueWithTime(K key, V value, Long time, TimeUnit unit) {
        getRedisTemplate().opsForValue().set(key, value, time, unit);
    }

    public V getValue(K key) {
        return getRedisTemplate().opsForValue().get(key);
    }

    public void rPushList(K key, V value) {
        getRedisTemplate().opsForList().rightPush(key, value);
    }

    public V lPopList(K key) {
        return getRedisTemplate().opsForList().leftPop(key);
    }

    public void delKey(K key) {
        getRedisTemplate().delete(key);
    }
}
```

##### 具体使用

使用带过期时间的使用，这里过期时间是20秒

```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration({ "/applicationContext-dao.xml",
    "/applicationContext-service.xml",
    "/applicationContext-service-lmswitch.xml",
    "/applicationContext-redis-context.xml"}
)
public class RedisServiceTest {


    @Autowired
    private RedisService<String,Coupon> redisCouponService;

    @Test
    public void testRedisListTime(){

        Coupon coupon = new Coupon();
        coupon.setId(1);
        coupon.setCodeType(1);
        coupon.setCouponCode("FD1111");

        redisCouponService.setKeyAndValueWithTime("test", coupon, 20L, TimeUnit.SECONDS);



    }

    @Test
    public void testRedisListTimeGet(){
        Coupon coupon = (Coupon) redisCouponService.getValue("test");
        if (coupon != null) {
            System.out.println(coupon.getId());
        }
    }
}

```

#### 备注

>目前，我自己测试通过。以后具体会遇到哪些坑，我会边填边走~~

>Go 且行且尝试






    