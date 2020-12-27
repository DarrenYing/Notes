# Redis学习笔记

## 一、Redis介绍

### 1.1、引言

> 1. 由于用户量增大，请求数量也随之增大，数据压力过大
> 2. 多台服务器之间，数据不同步
> 3. 多台服务器之间的锁， 已经不存在互斥性了

![image-20201224092558107](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20201224092558107.png)

### 1.2、NoSQL

> Redis就是一款NoSQL
>
> NoSQL-->非关系型数据库-->Not Only SQL
>
> 1. Key-Value：Redis
> 2. 文档型：ElasticSearch, Solr, Mongodb
> 3. 面向列：Hbase，Cassandra
> 4. 图形化：Neo4j
>
> 除了关系型数据库都是非关系型数据库
>
> NoSQL只是一种概念，泛指非关系型数据库

### 1.3、Redis介绍

> 意大利人，在开发款LLOOGG的统计页面，因为MySQL性能不好，自己研发了一款非关系型数据库，并命名为Redis。
>
> Redis（Remote Dictionary Server）即远程字典服务，Redis是由C语言编写，是一款基于Key-Value的NoSQL，其基于内存存储数据，提供多种持久化机制，性能：读取数据可达110000次/s，写入数据可达81000次/s，Redis还提供了主从、哨兵以及集群的搭建方式，可以更方便地横向扩展以及垂直扩展。

## 二、Redis安装

### 2.1、安装Redis

```yml
version: '3.1'
services:
  redis:
    image: daocloud.io/library/redis:5.0.7
    restart: always
    container_name: redis
    environment:
      - TZ=Asia/Shanghai
    ports:
      - 6379:6379
```

### 2.2、使用redis-cli连接Redis

> 进去redis容器内部：docker exec -it 容器id bash
>
> 在容器内部，使用redis-cli连接

### 2.3、使用图形化界面连接Redis

> 下载地址：[Releases · lework/RedisDesktopManager-Windows (github.com)](https://github.com/lework/RedisDesktopManager-Windows/releases)
>
> 安装后，连接时IP改为云服务器IP即可

## 三、Redis常用命令

### 3.1、Redis存储数据的结构

> 常用的5种数据结构
>
> - key-string：一个key对应一个值
> - key-hash：一个key对应一个Map
> - key-list：一个key对应一个列表
> - key-set：一个key对应一个集合
> - key-zset：一个key对应一个有序的集合
>
> 另外三种数据结构
>
> HyperLogLog：计算近似值
>
> GEO：存储地理位置
>
> BIT：通过byte数组来存储

![image-20201224164949920](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20201224164949920.png)

> key-string：最常用，一般用于存储单个值
>
> key-hash：存储一个对象数据
>
> key-list：使用list结构实现栈和队列结构
>
> key-set：交集，差集和并集的操作
>
> key-zset：常用于排行榜，积分存储等操作

### 3.2、string常用命令

```sh
# 1. 添加值，如果key已经存在，就覆盖
set key value

# 2. 取值
get key

# 3. 批量操作
mset key value [key value ...]
mget key [key ...]
```

---

```sh
# 4. 自增命令
inc key

# 5. 自减命令
decr key

# 6. 自增或自减指定数量
incrby key increment
decrby key increment
```

---

```sh
# 7. 设置值的同时，指定其生存时间（推荐使用）
setex key second value

# 8. 设置值，如果key已存在，则跳过，否则和set命令一样
setnx key value

# 9. 在key对应的value后，追加内容
append key value

# 10. 查看value字符串的长度
strlen key
```

### 3.3、hash常用命令

```sh
# 1. 存储数据
hset key field value

# 2. 获取数据
hget key field 

# 3. 批量操作
hmset key field value [field value ...]
hmget key field [field ...]
```

---

```sh
# 4. 自增(increment可以为正，也可以为负)
hincrby key field increment
```

---

```sh
# 5. 设置值(如果key-field不存在，则正常添加，如果存在，就跳过)
hsetnx key field value

# 6. 检查field是否存在
hexists key field

# 7. 删除key对应的field，可以删除多个
hdel key field [field ...]
```

---

```sh
# 8. 获取当前hash结构中的全部field和value
hgetall key # py: dict.items()

# 9. 获取当前hash结构中的全部field
hkeys key # py: dict.keys()

# 10. 获取当前hash结构中的全部value
hvals key # py: dict.vals()

# 11. 获取当前hash结构中field的数量
hlen key
```

### 3.4、list常用命令

```sh
# 1. 存储数据(从左侧插入，从右侧插入)
lpush key value [value ...]
rpush key value [value ...]

# 2. 存储数据(如果key不存在或者key不是list结构，则跳过;无法批量操作)
lpushx key value
rpushx key value

# 3. 存储数据(修改指定位置的数据)
lset key index value
```

---

```sh
# 4. 弹栈方式获取数据
lpop key
rpop key

# 5. 获取指定索引范围的数据(start从0开始，stop支持负数，如-1；两边都是闭)
lrange key start stop

# 6. 获取指定索引位置的数据
lindex key index

# 7. 获取整个列表的长度
llen key
```

---

```sh
# 8. 删除列表中的数据(删除count个相应value值，count<0时，从右向左删除，count=0时删除全部数据)
lrem key count value

# 9. 保留列表中的数据
ltrim key start stop

# 10. 将list1中的队尾(最右)，插入到list2头部(最左)
rpoplpush list1 list2
```

### 3.5、set常用命令

```sh
# 1. 存储数据
sadd key member [member ...]

# 2. 获取数据（获取全部数据）
smembers key

# 3. 随机获取一个数据（获取的同时，移除数据，count默认为1，代表弹出数据的数量）
spop key [count]
```

---

```sh
# 4. 交集
sinter set1 set2 ...

# 5. 并集
sunion set1 set2 ...

# 6. 差集(set1-set2-...)
sdiff set1 set2 ...
```

---

```sh
# 7. 删除数据
srem key member [member ...]

# 8. 查看当前的set集合中是否包含这个值
sismember key member
```

### 3.6、zset常用命令

```sh
# 1. 添加数据(score必须是数值，member不允许重复)
zadd key score member [score member ...]

# 2. 修改member的分数(如果member不存在，会先添加该member)
zincrby key score member
```

---

```sh
# 3. 查看指定的member的分数
zscore key member

# 4. 获取zset中member的数量
zcard key

# 5. 根基score的范围查询member数量
zcount key min max

# 6. 删除zset中的成员
zrem key member [member ...]
```

---

```sh
# 7. 根据分数从小到大排序，获取指定范围内的数据(withscore表示返回时，会返回member对应分数)
zrange key start stop [withscores]

# 8. 根据分数从大到小排序，获取指定范围内的数据
zrevrange key start stop [withscores]

# 9. 根据score的范围获取member(limit和MySQL中用法一致)
zrangebyscore key min max [withscores] [limit offset count]

# 10. 根据score的范围获取member
zrevrangebyscore key max min [withscores] [limit offset count]

# some examples
zrangebyscore salary 3000 10000  # 左右都闭
zrangebyscore salary (3000 10000 # 左开右闭
zrangebyscore salary -inf +inf	 # 查询全部
```

### 3.7、key常用命令

```sh
# 1. 查看Redis中全部的key (pattern: *, xxx*, *xxx)
 keys pattern
 
 # 2. 查看某一个key是否存在(1-存在，0-不存在)
 exists key
 
 # 3. 删除key
 del key [key ...]
```

---

```sh
# 4. 设置key的生存时间，单位为秒或毫秒
expire key second
pexpire key milliseconds

# 5. 设置key的生存时刻，单位为秒或毫秒
expireat key timestamp
pexpireat key milliseconds-timestamp

# 6. 查看key的剩余生存时间，单位为秒或毫秒(-2:当前key不存在，-1:key未设置生存时间)
ttl key
pttl key

# 7. 移除key的生存时间(1-移除成功，0-key未设置生存时间或key不存在)
persist key
```

---

```sh
# 8. 选择操作的库(redis默认有16个库，编号0-15，默认选择0号库)
select 0-15

# 9. 移动key到另一个库中
move key db
```

### 3.8、库的常用命令

```sh
# 1. 清空当前所在的数据库
flushdb

# 2. 清空全部数据库
flushall

# 3. 查看当前数据库中有多少个key
dbsize

# 4. 查看最后一次操作的时间(准确说是最后一次操作磁盘的时间)
lastsave

# 5. 实时监控Redis服务接收到的操作命令
monitor
```

## 四、Java连接Redis

> 通过Jedis连接和操作，操作完记得释放资源

### 4.1、Jedis连接Redis



### 4.2、Jedis如何存储一个对象到Redis

> 以byte[]的形式存储，通过序列化和反序列化即可



## 五、Redis其他配置及集群





## 六、Redis常见问题





