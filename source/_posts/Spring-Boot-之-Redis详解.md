---
title: Spring Boot 之 Redis详解
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2017-08-01 21:30:48
categories: 框架
tags: SpringBoot
---

Redis是目前业界使用最广泛的内存数据存储。

Redis支持丰富的数据结构，同时支持数据持久化。

Redis还提供一些类数据库的特性，比如事务，HA，主从库。

REmote DIctionary Server(Redis) 是一个由Salvatore Sanfilippo写的key-value存储系统。

![](http://img3.imgtn.bdimg.com/it/u=247756168,1064472101&fm=26&gp=0.jpg)
<!--more-->

# Redis特点

Redis支持数据的持久化，可以将内存中的数据保存到磁盘中，重启的时候可以再次加载使用

Redis不仅仅支持简单的key-value类型的数据，同时还支持list，set，zset，hash等数据结构的存储

Redis支持数据的备份

支持主从同步，数据存在内存中，性能卓越。

***

# Redis数据结构

## String（字符串）

String是redis最基本的类型，一个key对应一个value

```sql
127.0.0.1:6379> set name "runoob"
OK
127.0.0.1:6379> get name
"runoob"
```

set 和 get 命令，key为name，value为runoob。



## List（列表）

双向列表，适用于最新列表，关注列表

列表是最简单的字符串列表，按照插入顺序排序。可以添加一个元素到列表的头部或者尾部

```sql
127.0.0.1:6379> lpush runoob redis
(integer) 1
127.0.0.1:6379> lpush runoob mongodb
(integer) 2
127.0.0.1:6379> lpush runoob damin
(integer) 3
127.0.0.1:6379> lrange runoob 0 10
1) "damin"
2) "mongodb"
3) "redis"
```

lpush存入链表，lrange列出链表



## Set（集合）

适用于无顺序的集合，点赞点踩，抽奖，已读，共同好友

Redis的Set是String类型的无序集合。

集合是通过哈希表实现的，所以添加，删除，查找，都是$O(1)$

$asdd$命令：添加一个string元素命令到key对应的set集合中，成功返回1，如果元素已经存在，返回0，key对应的set不存在返回错误。

```sql
127.0.0.1:6379> SADD damin redis
(integer) 1
127.0.0.1:6379> SADD damin mysql
(integer) 1
127.0.0.1:6379> SADD damin mongodb
(integer) 1
127.0.0.1:6379> SADD damin mysql
(integer) 0
127.0.0.1:6379> SMEMBERS damin
1) "mongodb"
2) "mysql"
3) "redis"
```



## SortedSet（有序集合）

排行榜，优先队列

和set一样，zadd也是不允许重复的成员

## Hash（哈希）

对象属性，不定长属性数

是一个键名对集合

hash特别适合用于存储对象

```sql
127.0.0.1:6379> HMSET user:1 username runoob password runoob points 200
OK
127.0.0.1:6379> HGETALL user:1
1) "username"
2) "runoob"
3) "password"
4) "runoob"
5) "points"
6) "200"
```

Redis HMSET , HGETALL 命令，user:1为键值



KV：单一数值，验证码，PV，缓存

***

# 启动Redis

```reStructuredText
1、用cmd命令转到redis根目录下
2、执行cmd命令：redis-server redis.windows.conf
```



***

# Redis基础

基本指令功能


```java
/**
 * Created by nowcoder on 2016/7/30.
 */
@Service
public class JedisAdapter implements InitializingBean {
    private static final Logger logger = LoggerFactory.getLogger(JedisAdapter.class);
    private JedisPool pool;

    public static void print(int index, Object obj) {
        System.out.println(String.format("%d, %s", index, obj.toString()));
    }

    public static void main(String[] argv) {
        Jedis jedis = new Jedis("redis://localhost:6379/9");
        jedis.flushDB();

        // get set
        jedis.set("hello", "world");
        print(1, jedis.get("hello")); //输出hello对应的world
        jedis.rename("hello", "newhello"); //newhello替代hello
        print(1, jedis.get("newhello"));  //指向输出world
        jedis.setex("hello2", 1800, "world");
        print(1,jedis.get("hello2"));

        jedis.set("pv","100");    //点踩功能
        jedis.incr("pv");    //加1
        jedis.incrBy("pv",5); //加5
        print(2,jedis.get("pv"));
        jedis.decrBy("pv",2);  //减2
        print(2,jedis.get("pv"));

        print(3,jedis.keys("*"));   //输出所有表

        String listName ="list";
        jedis.del(listName);
        for(int i=0;i<10;i++){
            jedis.lpush(listName,"a"+String.valueOf(i));   //添加进表
        }
      
        print(4,jedis.lrange(listName,0,12)); //输出范围0-12的内容
        print(4,jedis.lrange(listName,0,2));  //输出范围0-2的内容
        print(5,jedis.llen(listName));  //输出长度
        print(6,jedis.lpop(listName)); //弹出最前一个的内容
        print(10,jedis.linsert(listName,BinaryClient.LIST_POSITION.AFTER,"a4","dd"));
        print(11,jedis.lrange(listName,0,11));

        String userKey = "userxx";
        jedis.hset(userKey,"name","jim");
        jedis.hset(userKey,"age","12");
        jedis.hset(userKey,"phone","1562225555");
        print(12,jedis.hget(userKey,"name")); //取出对应信息
        print(13,jedis.hgetAll(userKey));  //全部取出
        jedis.hdel(userKey,"phone");
        print(14,jedis.hgetAll(userKey));  //取出全部信息
        print(15,jedis.hexists(userKey,"email")); //有没有email
        print(16,jedis.hexists(userKey,"age")); //有没有age
        print(17,jedis.hkeys(userKey)); //输出key
        print(18,jedis.hvals(userKey));// 输出值
        jedis.hsetnx(userKey,"school","zju"); //添加学校属性
        jedis.hsetnx(userKey,"name","yxy");   //如果存在则不改写
        print(19,jedis.hgetAll(userKey));

        //set
        String likeKey1="commentLike1";
        String likeKey2="commentLike2";
        for(int i=0;i<10 ; ++i){
            jedis.sadd(likeKey1,String.valueOf(i));
            jedis.sadd(likeKey2,String.valueOf(i*i));
        }
        print(20,jedis.smembers(likeKey1));
        print(21,jedis.smembers(likeKey2));
        print(22,jedis.sunion(likeKey1,likeKey2));//求并
        print(23,jedis.sdiff(likeKey1,likeKey2));//我有你无
        print(24,jedis.sinter(likeKey1,likeKey2)); //求公共的
        print(25,jedis.sismember(likeKey1,"12"));
        print(25,jedis.sismember(likeKey2,"16")); //查询对象在不在里面
        jedis.srem(likeKey1,"5");
        print(27,jedis.smembers(likeKey1));

        User user = new User();
        user.setName("xx");
        user.setPassword("ppp");
        user.setHeadUrl("a.png");
        user.setSalt("salt");
        user.setId(1);
        print(46,JSONObject.toJSONString(user));
        jedis.set("user1",JSONObject.toJSONString(user));

        String value = jedis.get("user1");
        User user2=JSON.parseObject(value,User.class);
        print(47,user2);
        int k=2;
    }
    @Override
    public void afterPropertiesSet() throws Exception {
    }
}
```

---

# Redis事务

redis事务可以一次执行多个命令，并且有以下2个属性：

事务是一个单独的隔离操作，事务中所有命令都会序列化，按顺序地执行，事务在执行的过程中，不会被其他客户端发送过来的命令请求所中断

事务是一个原子操作，事务中的命令要不全部被执行，要不都不执行

一个事务从开始到执行有三个阶段 ：

开始事务

命令入队

执行事务

实例：

先以MULTI开始一个事务，然后将多个命令入队到事务中，最后由EXEC命令触发事务，一并执行事务中的所有命令

```sql
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> set book-name "Java Thinking in"
QUEUED
127.0.0.1:6379> GET book-name
QUEUED
127.0.0.1:6379> SADD tag "JAVA" "Coding"
QUEUED
127.0.0.1:6379> SMEMBERS tag
QUEUED
127.0.0.1:6379> EXEC
1) OK
2) "Java Thinking in"
3) (integer) 2
4) 1) "JAVA"
   2) "Coding"
```

事务相关命令：

DISCARD：取消事务discard，放弃执行事务中所有命令

EXEC：执行所有事务块内的命令 exec

MULTI：标记一个事务块的开始multi

UNWATCH：取消watch命令对所有key的监视

---

# Redis脚本

Redis脚本用Lua解释器来执行脚本，脚本执行的命令为EVAL

```sql
127.0.0.1:6379> EVAL "return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}" 2 key1 key2 first second
1) "key1"
2) "key2"
3) "first"
4) "second"
```

---

# Redis 连接

Redis连接服务器

```sql
127.0.0.1:6379> AUTH "password"
(error) ERR Client sent AUTH, but no password is set //没有密码，所以...
127.0.0.1:6379> ping
PONG
127.0.0.1:6379>
```

连接命令：

AUTH password：验证密码是否正确

PONG：查看服务器是否运行

ECHO message ： 打印字符串

QUIT：关闭当前连接

SELECT index ：切换到指定的数据库

---

# Java使用Redis

添加jedis.jar 包

然后就可以嗨起来了

## 连接服务器

```java
import redis.clients.jedis.Jedis;
public class RedisJava {
    public static void main(String [] args){
        //连接本地的Redis服务
        Jedis jedis =new Jedis("localhost");
        System.out.println("连接成功");
        //查看服务器是否运行
        System.out.println("服务正在运行："+jedis.ping());
    }
}

---
  连接成功
服务正在运行：PONG
```

## 字符串

```java
import redis.clients.jedis.Jedis;
public class RedisJava {
    public static void main(String [] args){
        //连接本地的Redis服务
        Jedis jedis =new Jedis("localhost");
        System.out.println("连接成功");
        jedis.set("damin","zhoudm.com");
        System.out.println("redis 存储的字符串为："+jedis.get("damin"));
    }
}

---
连接成功
redis 存储的字符串为：zhoudm.com
```

## Redis Java List

```java
import redis.clients.jedis.Jedis;
import java.util.List;
public class RedisJava {
    public static void main(String [] args){
        //连接本地的Redis服务
        Jedis jedis =new Jedis("localhost");
        System.out.println("连接成功");
        jedis.lpush("zhoudm","zhoudm.com");
        jedis.lpush("zhoudm","baidu.com");
        jedis.lpush("zhoudm","360.com");
        jedis.lpush("zhoudm","google.com");

        List<String> list=jedis.lrange("zhoudm",0,3);
        for(int i=0;i<list.size();i++) {
            System.out.println("列表为：" + list.get(i));
        }
    }
}

---
连接成功
列表为：google.com
列表为：360.com
列表为：baidu.com
列表为：zhoudm.com
```

## Redis Java Keys

```java
import redis.clients.jedis.Jedis;
import java.util.Set;
import java.util.Iterator;

public class RedisJava {
    public static void main(String [] args){
        //连接本地的Redis服务
        Jedis jedis =new Jedis("localhost");
        System.out.println("连接成功");

        Set<String> keys=jedis.keys("*");
        Iterator<String> it=keys.iterator();
        while(it.hasNext()){
            String key = it.next();
            System.out.println(key);
        }
    }
}

---
 连接成功
zhoudm
spring:session:sessions:7ff2fe7e-9454-48a1-b44c-9b5f6b7b1507
book-name
spring:session:sessions:9a02f60c-fd16-49b5-9977-08446f4ffff5
spring:session:sessions:f1312270-95cf-4d19-a88d-5af26b03a61c
spring:session:sessions:expires:9a02f60c-fd16-49b5-9977-08446f4ffff5
spring:session:expirations:1504881480000
user:1
spring:session:sessions:f31624a8-b014-4768-826d-8b5b484b63a7
collector
spring:session:sessions:expires:f1312270-95cf-4d19-a88d-5af26b03a61c
spring:session:sessions:expires:f31624a8-b014-4768-826d-8b5b484b63a7
name
spring:session:expirations:1504409460000
damin
runoob
tag
spring:session:expirations:1505295120000
spring:session:expirations:1504252440000
spring:session:sessions:expires:7ff2fe7e-9454-48a1-b44c-9b5f6b7b1507 
```





---





![](http://img5.imgtn.bdimg.com/it/u=2420256015,867012716&fm=26&gp=0.jpg)

![](http://img4.imgtn.bdimg.com/it/u=964798549,1007679285&fm=26&gp=0.jpg)