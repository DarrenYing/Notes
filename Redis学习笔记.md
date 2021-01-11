# Redis学习笔记

## 一、Redis介绍

### 1.1、引言

> 1. 由于用户量增大，请求数量也随之增大，数据压力过大
> 2. 多台服务器之间，数据不同步
> 3. 多台服务器之间的锁， 已经不存在互斥性了

![image-20201224092558107](https://i.loli.net/2021/01/11/idh63Lbknmj2qfy.png)

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

![image-20201224164949920](https://i.loli.net/2021/01/11/NylaXLubBzjTF9h.png)

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



### 4.3、Jedis如何以Json String的形式存储一个对象到Redis

> 创建对象后转为json字符串，再存到Redis即可，取出时再由json转为对象

![image-20201228094625249](https://i.loli.net/2021/01/11/hHTdDFJiS3nVeN2.png)

### 4.4、使用Jedis连接池

> **jedis连接资源的创建与销毁是很消耗程序性能**，所以jedis为我们提供了jedis的**池化技术**，
>
> jedisPool在创建时初始化一些连接资源存储到连接池中，使用jedis连接资源时不需要创建，
>
> 而是从连接池中获取一个资源进行redis的操作，使用完毕后，不需要销毁该jedis连接资源，
>
> 而是将该资源归还给连接池，供其他请求使用。

> 1、简单方式

![image-20201228094853644](https://i.loli.net/2021/01/11/Adr3VNDLJupiZeG.png)

> 设置配置信息，再创建连接处
>
> maxIdle 连接池中最多可空闲maxIdle个连接 ，若取值为20，表示即使没有数据库连接时依然可以保持20空闲的连接，而不被清除，随时处于待命状态。设 0 为没有限制。
>
> maxIdle通常和maxTotal设置为同一个值

![image-20201228095634165](https://i.loli.net/2021/01/11/7edR2cTolMJxgCv.png)

其他步骤同上。

### 4.5、Redis的管道操作

>在操作Redis时，每执行一条命令需要先发送请求到Redis服务器，这个过程要经过网络延迟，Redis还需要给客户端一个响应。
>
>如果需要一次性执行多个命令，上述方式效率低，可以通过Redis的管道，先将命令全部放到客户端的一个pipeline中，再一次性将这些命令发送到Redis服务器，Redis也可以一次性返回全部命令执行后的响应结果

![image-20201228100704492](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20201228100704492.png)



## 五、Redis其他配置及集群

> 修改.yml文件，做数据集映射，方便后期修改Redis配置信息

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
    volumes:
      - ./conf/redis.conf:/usr/local/redis/redis.conf
    command: ["redis-server","/usr/local/redis/redis.conf"]
```

### 5.1、Redis的AUTH

> 方式一：通过修改Redis的配置文件，实现Redis的密码校验

```
# redis.conf
requirepass 密码
```

> 三种客户端的连接方式
>
> 1. redis-cli：在输入正常命令之前，先输入auth密码即可
>
> 2. 图形化界面，在连接Redis的信息中添加上密码
>
> 3. Jedis客户端：
>
>    第一种：jedis.auth(password);
>
>    第二种：使用JedisPool的方式，选择带密码参数的构造函数即可

---

> 方式二：在不修改redis.conf文件的前提下，在第一次连接Redis时，输入命令
>
> Config set requirepass 密码
>
> 后续执行操作时，就需要输入auth校验。

### 5.2、Redis的事务

> Redis的事务：一次事务操作，该成功的成功，该失败的失败
>
> 先开启事务，执行一系列的命令，但是命令不会立即执行，会被放在一个队列中，如果你执行事务，那么这个队列中的命令全部执行，如果取消了事务，一个队列中的命令全部作废。
>
> 1. 开启事务：multi
> 2. 输入要执行的命令：被放入到一个队列中
> 3. 执行事务：exec
> 4. 取消事务：discard
>
> Redis的事务想发挥功能，需要配置watch监听机制
>
> 1. watch key [key ...]
> 2. unwatch key [key ...]
>
> 在开启事务之前，先通过watch命令去监听一个或多个key，在开启事务之后，如果有其他客户端修改了我监听的key，事务会自动取消
>
> 如果执行了事务，或者取消了事务，watch监听自动消除，一般不需要手动执行unwatch

### 5.3、Redis持久化机制

> RDB是Redis默认的持久化机制
>
> 1. RDB持久化文件，速度较快，以二进制文件存储，传输方便
> 2. RDB无法保证数据的绝对完全

```sh
save 900 1       # 在900秒内，有1个key改变了，就执行RDB持久化
save 300 10		 # 在300秒内，有10个key改变了，就执行RDB持久化
save 60 10000    # 在60秒s，有10000个key改变了，就执行RDB持久化

rdbcompression yes	# 开启持久化压缩

dbfilename dump.rdb # 持久化文件名称
```

---

> AOF持久化机制默认是关闭的，Redis官方推荐同时开启RDB和AOF，更安全
>
> 1. AOF持久化的速度，相对RDB较慢，存储一个文本文件，后期文件体积可能较大，传输困难
> 2. AOF持久化时机。（下方代码块后三行）
> 3. AOF相对RDB更安全，推荐两者同时开启。

```sh
appendonly no         # no表示关闭aof，yes表示开启aof
appendfilename "appendonly.aof"  # aof持久化文件名称
appendfsync always    # 每执行一个写操作，立即持久化到AOF文件中，性能比较低
appendfsync everysec  # 每秒执行一次持久化
appendfsync no        # 根据系统环境不同，在一定时间内执行一次持久化
```

---

> 同时开启AOF和RDB的注意事项（建议操作之前两者都开）
>
> 如果同时开启了AOF和RDB持久化，那么Redis宕机重启后，会优先加载AOF的持久化文件。
>
> 如果先开启了RDB，再次开启AOF，如果RDB执行了持久化，那么RDB文件中的内容会被AOF覆盖掉。

### 5.4、Redis的主从架构

> 单个Redis存在读写瓶颈的问题

![image-20201229102735599](https://i.loli.net/2021/01/11/OJj9CDglnbN3X6t.png)

```yml
version: '3.1'
services:
  redis1:
    image: daocloud.io/library/redis:5.0.7
    restart: always
    container_name: redis1
    environment:
      - TZ=Asia/Shanghai
    ports:
      - 7001:6379
    volumes:
      - ./conf/redis1.conf:/usr/local/redis/redis.conf
    command: ["redis-server","/usr/local/redis/redis.conf"]
  redis2:
    image: daocloud.io/library/redis:5.0.7
    restart: always
    container_name: redis2
    environment:
      - TZ=Asia/Shanghai
    ports:
      - 7002:6379
    volumes:
      - ./conf/redis2.conf:/usr/local/redis/redis.conf
    links:
      - redis1:master
    command: ["redis-server","/usr/local/redis/redis.conf"]
  redis3:
    image: daocloud.io/library/redis:5.0.7
    restart: always
    container_name: redis3
    environment:
      - TZ=Asia/Shanghai
    ports:
      - 7003:6379
    volumes:
      - ./conf/redis3.conf:/usr/local/redis/redis.conf
    links:
      - redis1:master
    command: ["redis-server","/usr/local/redis/redis.conf"]
```

---

```
# 从节点配置 replica of
replicaof <masterip> <masterport> # 对于本例，即为 replicaof master 6379
```



### 5.5、哨兵

> 哨兵可以帮助我们解决主从架构中的单点故障问题

> 修改主从架构配置文件

```yml
version: '3.1'
services:
  redis1:
    image: daocloud.io/library/redis:5.0.7
    restart: always
    container_name: redis1
    environment:
      - TZ=Asia/Shanghai
    ports:
      - 7001:6379
    volumes:
      - ./conf/redis1.conf:/usr/local/redis/redis.conf
      - ./conf/sentinel1.conf:/data/sentinel.conf    # 修改处
    command: ["redis-server","/usr/local/redis/redis.conf"]
  redis2:
    image: daocloud.io/library/redis:5.0.7
    restart: always
    container_name: redis2
    environment:
      - TZ=Asia/Shanghai
    ports:
      - 7002:6379
    volumes:
      - ./conf/redis2.conf:/usr/local/redis/redis.conf
      - ./conf/sentinel2.conf:/data/sentinel.conf    # 修改处
    links:
      - redis1:master
    command: ["redis-server","/usr/local/redis/redis.conf"]
  redis3:
    image: daocloud.io/library/redis:5.0.7
    restart: always
    container_name: redis3
    environment:
      - TZ=Asia/Shanghai
    ports:
      - 7003:6379
    volumes:
      - ./conf/redis3.conf:/usr/local/redis/redis.conf
      - ./conf/sentinel3.conf:/data/sentinel.conf    # 修改处
    links:
      - redis1:master
    command: ["redis-server","/usr/local/redis/redis.conf"]
```

> 准备哨兵的配置文件，并且在容器内部手动启动哨兵即可

```sh
# 哨兵需要后台启动
daemonize yes
# 指定Master节点的ip和端口(主)
sentinel monitor master localhost 6379 2
# 指定Master节点的ip和端口(从)
sentinel monitor master ip地址 6379 2
# 哨兵每隔多久监听一次redis架构
sentinel down-after-milliseconds master 10000
```

> 在Redis容器内部启动sentinel即可

```
redis-sentinel sentinel.conf
```

### 5.6、Redis集群

> Redis集群在保证主从加哨兵的基本功能之外，还能够提升Redis存储数据的能力

>Redis集群的特点：
>
>1. Redis集群是无中心的
>2. Redis集群有一个ping-pong的机制
>3. 投票机制，Redis集群节点的数量必须是2n+1
>4. Redis集群中默认分配了16384个hash槽，在存储数据时，就会将key进行crc16的算法，并且对16384取余，根据最终的结果，将key-value存放到执行Redis节点中，而且每一个Redis集群都在维护着相应的hash槽
>5. 为了保证数据的安全性，集群中的每一个节点，至少都要跟着一个从节点
>6. 可以单独地针对Redis集群中的某一个节点搭建主从
>7. 当Redis集群中超过半数的节点宕机之后，Redis集群就瘫痪了

![image-20201231094420587](https://i.loli.net/2021/01/11/gbXpVMsNjJP7TFR.png)

```yml
# docker-compose.yml
version: "3.1"
services:
  redis1:
    image: daocloud.io/library/redis:5.0.7
    restart: always
    container_name: redis1
    environment:
      - TZ=Asia/Shanghai
    ports:
      - 7001:7001
      - 17001:17001
    volumes:
      - ./conf/redis1.conf:/usr/local/redis/redis.conf
    command: ["redis-server","/usr/local/redis/redis.conf"]
  redis2:
    image: daocloud.io/library/redis:5.0.7
    restart: always
    container_name: redis2
    environment:
      - TZ=Asia/Shanghai
    ports:
      - 7002:7002
      - 17002:17002
    volumes:
      - ./conf/redis2.conf:/usr/local/redis/redis.conf
    command: ["redis-server","/usr/local/redis/redis.conf"]
  redis3:
    image: daocloud.io/library/redis:5.0.7
    restart: always
    container_name: redis3
    environment:
      - TZ=Asia/Shanghai
    ports:
      - 7003:7003
      - 17003:17003
    volumes:
      - ./conf/redis1.conf:/usr/local/redis/redis.conf
    command: ["redis-server","/usr/local/redis/redis.conf"]
  redis4:
    image: daocloud.io/library/redis:5.0.7
    restart: always
    container_name: redis4
    environment:
      - TZ=Asia/Shanghai
    ports:
      - 7004:7004
      - 17004:17004
    volumes:
      - ./conf/redis4.conf:/usr/local/redis/redis.conf
    command: ["redis-server","/usr/local/redis/redis.conf"]
  redis5:
    image: daocloud.io/library/redis:5.0.7
    restart: always
    container_name: redis5
    environment:
      - TZ=Asia/Shanghai
    ports:
      - 7005:7005
      - 17005:17005
    volumes:
      - ./conf/redis5.conf:/usr/local/redis/redis.conf
    command: ["redis-server","/usr/local/redis/redis.conf"]
  redis6:
    image: daocloud.io/library/redis:5.0.7
    restart: always
    container_name: redis6
    environment:
      - TZ=Asia/Shanghai
    ports:
      - 7006:7006
      - 17006:17006
    volumes:
      - ./conf/redis6.conf:/usr/local/redis/redis.conf
    command: ["redis-server","/usr/local/redis/redis.conf"]
```

---

```sh
# redis.conf
# 指定redis的端口号
port 7001
# 开启Redis集群
cluster-enbaled yes
# 集群信息的文件
cluster-config-file nodes-7001.conf
# 集群的对外ip地址
cluster-announce-ip 192.168.199.109
# 集群对外的port
cluster-announce-port 7001
# 集群的总线端口
cluster-announce-bus-port 17001
```

---

> 启动了6个Redis的节点
>
> 随便跳转到其中一个容器内部，使用redis-cli管理集群
>
> cluster-replicas参数指定每个主节点跟几个从节点

```sh
redis-cli --cluster create 47.106.189.246:7001 47.106.189.246:7002 47.106.189.246:7003 47.106.189.246:7004 47.106.189.246:7005 47.106.189.246:7006 --cluster-replicas 1
```

### 5.7、Java连接Redis集群

> 使用JedisCluster对象连接Redis集群

```java
@Test
public void test(){
	// 创建Set<HostAndPort> nodes
    Set<HostAndPort> nodes = new HashSet<>();
    nodes.add(new HostAndPort("47.106.189.246", "7001"));
    nodes.add(new HostAndPort("47.106.189.246", "7002"));
    nodes.add(new HostAndPort("47.106.189.246", "7003"));
    nodes.add(new HostAndPort("47.106.189.246", "7004"));
    nodes.add(new HostAndPort("47.106.189.246", "7005"));
    nodes.add(new HostAndPort("47.106.189.246", "7006"));
    
    // 创建JedisCluster对象
    JedisCluster jedisCluster = new JedisCluster(nodes);
    
    // 操作
    String value = jedisCluster.get('a');
    System.out.println(value);
}
```



## 六、Redis常见问题

### 6.1、Redis的删除策略

> key的生存时间到了，Redis就会立即删除吗？

> 不会立即删除。
>
> 1. 定期删除：
>
>    Redis每隔一段时间就会去查看Redis设置了过期时间的key，会在100ms的间隔中默认查看3个key
>
> 2. 惰性删除：
>
>    如果当你去查询一个设置了生存时间的key时，Redis会先查看当前key的生存时间是否依旧到了，到了的话就直接删除当前key，并返回给用户一个空值

### 6.2、Redis的淘汰机制

> 在Redis的内存满时，若添加新的数据，会触发淘汰机制。

> 1. volatile-lru：
>
>    在内存不足时，Redis会在设置了生存时间的key中淘汰一个最近最少使用的key
>
> 2. allkeys-lru：
>
>    在内存不足时，Redis会在全部的key中淘汰一个最近最少使用的key
>
> 3. volatile-lfu：
>
>    在内存不足时，Redis会在设置了生存时间的key中淘汰一个最近最少频次使用的key
>
> 4. allkeys-lfu：
>
>    在内存不足时，Redis会在全部的key中淘汰一个最近最少频次使用的key
>
> 5. volatile-random：
>
>    在内存不足时，Redis会在设置了生存时间的key中随机淘汰一个key
>
> 6. allkeys-random：
>
>    在内存不足时，Redis会在全部的key中随机淘汰一个key
>
> 7. volatile-ttl：
>
>    在内存不足时，Redis会在设置了生存时间的key中随机淘汰一个剩余生存时间最少的key
>
> 8. noeviction：(eviction 驱逐)
>
>    当内存不足时，直接报错

> 指定淘汰机制的方式：maxmemory-policy 具体策略名称
>
> 设置Redis的最大内存：maxmemory <bytes>

### 6.3、缓存的常见问题

> 缓存穿透问题：
>
> 原因：客户端查询的数据，在Redis中没有，在数据库中也没有
>
> 解决方案：
>
> 1. 根据id查询时，如果id是自增的，就将id的最大值放到Redis中，如果查询的id过大，就不用再查询数据库
> 2. 如果id不是整型，可以将全部的id放到set中，在用户查询时，在set中先查看是否有该id即可
> 3. 获取客户端的ip地址，可以对ip添加访问限制 (比如限制1min内最多访问60次)

![image-20201231102704386](https://i.loli.net/2021/01/11/gbXpVMsNjJP7TFR.png)

---

> 缓存击穿问题：
>
> 原因：缓存中的热点数据，在Redis中的缓存突然到期了，造成了大量请求都去访问数据库，造成数据库宕机
>
> 解决方案：
>
> 1. 添加一个锁，在访问数据时，如果缓存中不存在，就只能允许有限个请求去访问数据库，避免数据库宕机（推荐，后面还有分布式锁）
> 2. 对于热点数据不设置生存时间（不推荐）

---

> 缓存雪崩问题：
>
> 原因：当大量缓存同时到期时，如果有大量的请求同时访问数据，会导致数据库宕机
>
> 解决方案：
>
> 1. 将缓存中数据的生存时间，设置为30min-60min的随机时间

---

> 缓存倾斜问题：
>
> 原因：大量热点数据放在了一个Redis节点上，导致Redis节点无法承受住大量的请求而宕机
>
> 解决方案：
>
> 1. 扩展主从架构，搭建大量的从节点，缓解Redis的压力（比较昂贵）
> 2. 可以在Tomcat中加JVM缓存，在查询Redis之前，先查询Tomcat中的缓存