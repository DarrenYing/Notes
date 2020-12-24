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









## 四、Java连接Redis







## 五、Redis其他配置及集群





## 六、Redis常见问题





