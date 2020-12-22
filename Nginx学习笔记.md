# Nginx学习笔记

## 一、Nginx介绍

> Nginx的特点：
>
> * 稳定性极强， 7*24小时不间断运行
> * 提供了非常丰富的配置实例
> * 占用内存小，并发能力强

## 二、Nginx的安装

### 2.1、安装Nginx

[docker-compose.yml]()

```yml
version: '3.1'
services:
  nginx:
    restart: always
    image: daocloud.io/library/nginx:latest
    container_name: nginx
    ports:
      - 80:80
```

### 2.2、Nginx的配置文件

> 关于Nginx的核心配置文件nginx.conf

```json
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

# 以上统称为全局块
# worker_processes数值越大，Nginx的并发能力就越强
# error_log表示Nginx的错误日志存放位置

events {
    worker_connections  1024;
}
# events模块
# worker_connections数值越大，Nginx的并发能力就越强

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    
    /*
      conf.d下default.conf的内容，一个server块
    */
    server {
        listen       80;
        listen  [::]:80;
        server_name  localhost;

        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }
		# location块
		# root：将接收到的请求根据/usr/share/nginx/html去查找静态资源
		# index：默认去上述路径中找到index.html或index.htm
    }
	# server块
	# listen: 代表Nginx监听的端口号
	# localhost: 代表Nginx接收请求的ip

    
    include /etc/nginx/conf.d/*.conf; /*引入文件，用户可以自定义*/
}
# http块
# include表示引入一个外部的文件 --> /mime.types中存放着大量的媒体类型
# include /etc/nginx/conf.d/*.conf; --> 引入了conf.d目录下的以.conf为结尾的配置文件
```

### 2.3、修改docker-compose文件

```yml
version: '3.1'
services:
  nginx:
    restart: always
    image: daocloud.io/library/nginx:latest
    container_name: nginx
    ports:
      - 80:80
    volumes:
      - /opt/docker_nginx/conf.d/:/etc/nginx/conf.d  # 映射
```

> 数据卷映射之后，要在对应目录/opt/docker_nginx/conf.d下创建相应的.conf文件

## 三、Nginx的反向代理

### 3.1、正向代理和反向代理介绍

> 正向代理：
>
> 1. 正向代理服务是由客户端设立的
> 2. 客户端知道代理服务器和目标服务器是谁
> 3. 帮助突破访问权限，提高访问速度，对目标服务器隐藏客户端的ip地址

> 反向代理：
>
> 1. 反向代理服务器是配置在服务端的
> 2. 客户端不知道访问的是哪一台服务器
> 3. 能达到负载均衡，并且可以隐藏服务器真正的ip地址

### 3.2、基于Nginx实现反向代理

> 准备一个目标服务器
>
> 启动之前的Tomcat服务器
>
> 编写Nginx的配置文件， 通过Nginx访问到Tomcat服务器

```
server {
  listen 80;
  server_name localhost;
  location / {
    proxy_pass http://云服务器ip:8080/;
  }
}
```

### 3.3、关于Nginx的location路径映射

> 优先级关系：
>
> （location = ）>（location /xx/yy/zz）>（location ^~）>（location ~，~*）>（location /起始路径）>（location /）

```json
# 1. = 匹配
location = / {
	# 精准匹配，主机名后面不能带任何字符串
}
```

---

```json
# 2. 通用匹配
location /xxx {
	# 匹配所有以/xxx开头的路径
}
```

---

```json
# 3. 正则匹配
location ~ /xxx {
	# 匹配所有以/xxx开头的路径
}
```

---

```json
# 4. 匹配开头路径
location ^~ /images/ {
	# 匹配所有以/images开头的路径
}
```

---

```json
# 5. ~* \.(gif|jpg)$ {
	# 匹配以gif或jpg为结尾的路径
}
```



## 四、Nginx负载均衡

> Nginx默认提供三种负载均衡的策略：
>
> 1. 轮询：将客户端发起的请求，平均分配给每一台服务器
> 2. 权重：将客户端的请求，根据服务器的权重值不同，分配不同的数量
> 3. ip_hash：基于发起请求的客户端的ip地址不同，他始终会将请求发送到指定的服务器上

### 4.1、轮询

> 只需要在配置文件中添加如下内容

```
upstream 名字 {
  server ip:port;
  server ip:port;
  ...
}

server {
  listen: 80;
  server_name localhost;
  
  location / {
    proxy_pass http://upstream的名字/;
  }
}
```

### 4.2、权重

> 在之前的基础上，加weight

```
upstream 名字 {
  server ip:port weight=10;(数字即可)
  server ip:port weight=2;
  ...
}

server {
  listen: 80;
  server_name localhost;
  
  location / {
    proxy_pass http://upstream的名字/;
  }
}
```

### 4.3、ip_hash

> 第一行加上ip_hash即可

```
upstream 名字 {
  ip_hash;
  server ip:port;
  server ip:port;
  ...
}

server {
  listen: 80;
  server_name localhost;
  
  location / {
    proxy_pass http://upstream的名字/;
  }
}
```



## 五、Nginx动静分离

> Nginx的并发能力公式：
>
> ​	worker_processes * worker_connections / n = Nginx最终的并发能力
>
> 对于动态资源，n=4，因为客户端请求Nginx后，Nginx要再请求服务器；
>
> 对于静态资源，n=2，因为客户端请求Nginx后，Nginx在本地若能找到资源，可以直接返回。
>
> Nginx通过动静分离，来提升并发能力，为用户提供更快的响应。

### 5.1、动态资源代理

```
# 配置如下
location / {
  proxy_pass 路径;
}
```

### 5.2、静态资源代理

```json
# 配置如下
location / {
  root 静态资源的路径
  index 默认访问路径下的资源名称
  autoindex on; # 以列表形式展示静态资源路径下的全部内容
}

# 先修改docker，添加一个数据卷，映射到Nginx服务器的一个目录
# 在数据卷的路径下添加静态资源，如index.html和1.jpg
# 修改配置文件，再restart
```



## 六、Nginx集群

### 6.1、引言

> 服务器和nginx都可能出现单点故障，需要通过集群解决
>
> 为了避免由于nginx的宕机，而导致整个程序的崩溃
>
> 准备多台nginx
>
> 准备keepalived，监听nginx的健康情况
>
> 准备haproxy，提供一个虚拟的路径，统一去接收用户请求

[![rrAgxK.png](https://s3.ax1x.com/2020/12/22/rrAgxK.png)](https://imgchr.com/i/rrAgxK)

### 6.2、具体搭建

[![rrAr5R.png](https://s3.ax1x.com/2020/12/22/rrAr5R.png)](https://imgchr.com/i/rrAr5R)

