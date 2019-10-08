## Redis安装

windos操作系统

https://jingyan.baidu.com/article/0f5fb099045b056d8334ea97.html

记得在增加一行requirepass 后重启一下服务器不然可能会报错，密码未设置。

## Redis常用命令

- del key 删除一个key
- dump key 序列化一个key的值
- exists key 判断一个key是否存在
- expire key seconds 设置一个key的过期时间
- pexpire key milliseconds 以毫秒设置一个key的过期时间
- ttl key 返回一个key的剩余生存时间以秒为单位
- pttl key 以毫秒为单位，返回给定的key的剩余生存时间
- persist key 移除key的过期时间，将key持久保持
- random key 随机返回一个key
- rename key newkey修改key的名称
- move

###### 内存维护策略

设置过期时间涉及到了redis的内存维护策略

redis作为优秀的中间缓存件，时常会存储大量的数据即使采用了集群部署来动态扩容，也用该及时的整理内存，维持系统性能。

在redis中有两种解决方案：

1、为数据设置超时时间。

2、采用LRU(最近最久未使用)算法动态的将不用的数据删除。

- volatile-lru:设定超时时间的数据中，删除最不常使用的数据。
- allkeys-lru:查询所有的key中最近最不常使用的数据进行删除，这是应用最广泛的策略。
- voliatitle-random:在已经设定了超时的数据中随即删除。
- allkeys-random:查询所有的key，之后随即删除。
- volatile-ttl:查询全部设定超时时间的数据，之后排序，将马上将要过期的数据进行删除
- noeviction:如果设置为该属性，则不会进行删除操作，如果内存溢出则报错返回。

## Key的命名建议

- key不要太长，尽量不要超过1024字节，会消耗内存，降低查找效率
- key也不要太短，可读性会降低
- 在一个项目中，key最好采用统一的命名模式

# Redis数据类型

redis支持五种数据类型：string，hash，list，set，zset。

## string

一个key对应一个value。string是二进制安全的。可以包含任何数据，例：jpg图片或者序列化对象。

是redis最基本的数据类型，一个键最大能存储512MB。

二进制安全是指在传输数据时，保证二进制数据的信息安全。特点是：1、编码、解码在客户端完成，执行效率高；2、不需要频繁的编解码，不会出现乱码

#### string命令

getset key value 替换并返回旧值

incr key自增1

decr key 自减1

incrby key value 按值自加

setnx key value key不存在时设置，存在本语句则无效

#### 应用场景

- 用于保存单个字符串或json字符串数据
- 因string是二进制安全的，所以可以把一个图片文件的内容作为字符串保存
- 计数器（常规key-value缓存应用。常规计数：微博数、粉丝数）

incr等指令本身就具有原子操作的特性，所以我们完全可以利用redis的incr、incrby、decr、decrby等指令来实现原子计数的效果。

## hash（哈希）

redis hash 是一个string类型的field和value的映射表，特别适合用于存储对象。redis中每个hash可以存储2^32-1键值对。

#### hash命令

hset  key名 属性名 value

hmset  key名 属性名 value 多个field-value

#### 应用场景

常用于存储一个对象

底层为一个hashmap，并提供了直接存取这个map成员的接口。

# Jedis

首先要加载jedis驱动

```
 <!--jedis客户端需要依赖-->
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>2.4.2</version>
        </dependency>
```



```java
/**
     * 测试字符串string
     * Redis中有哪些命令，jedis就有哪些方法
     */
    @Test
    public void t1(){
        Jedis jedis=new Jedis("127.0.0.1",6379);
        jedis.auth("12345");
        jedis.set("strName","字符串的名称");
        String strName=jedis.get("strName");
        System.out.println("Redis中的数据："+strName);
        jedis.close();
    }
    /**
     * Redis string 作用：为了减轻数据库（MySQL）的访问压力
     * 需求：判断某key是否存在，如果存在，就从redis中查询，如果不存在
     * 就查数据库，且要将查询出的数据存入redis
     */
    @Test
    public void t2(){
        Jedis jedis=new Jedis("127.0.0.1",6379);
        jedis.auth("12345");
        String key="applicationName";
        if(jedis.exists(key)) {
            String result = jedis.get(key);
            System.out.println("Redis 数据库中查询得到：" + result);
        }else{
            String result="微信开发会议达人";
            jedis.set(key,result);
            System.out.println("MySQL数据库查询得到："+result);
        }
        jedis.close();
    }
```



