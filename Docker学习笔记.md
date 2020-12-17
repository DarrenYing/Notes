# Docker学习笔记

## 一、Docker的基本操作

### 1.1、Docker 安装

> 直接使用docker desktop即可

### 1.2、Docker中央仓库（镜像源）

> 1. Docker官方仓库，速度较慢
>
>    [https://hub.docker.com/]()
>
> 2. 国内常用镜像网站
>
>    https://registry.docker-cn.com
>
>    http://hub.daocloud.io/
>
> 3. 在公司内部会采用私服的方式拉取镜像
>
>    配置/etc/docker/daemon.json
>
>    ```json
>    {
>    	"registry-mirrors": ["https://registry.docker-cn.com"],
>    	"insecure-regristries": ["ip:port"]
>    }
>    
>    systemctl darmon-reload
>    systemctl restart docker
>    ```

### 1.3、镜像的操作

```sh
# 拉取镜像到本地
docker pull 镜像名称[:tag]
# 举例
docker pull daocloud.io/daocloud/dao-wordpress:v4.7.3
```

---

```sh
# 查看本地全部镜像
docker images
# 删除本地镜像
docker rmi 镜像的标识
```

---

```sh
# 镜像的导入导出
# 将本地的镜像导出
docker save -o 导出的本地路径 镜像id
# 导入本地的镜像
docker load -i 本地镜像路径
# 修改镜像名称
docker tag 镜像id 新镜像名称:版本
```

### 1.4、容器的操作

```sh
# 1. 运行容器
# 简单操作
docker run 镜像的标识|镜像名称[:tag]
# 常用的参数
docker run -d -p 宿主机端口:容器端口 --name 容器名称 镜像的标识|镜像名称[:tag]
# -d: 代表在后台运行容器 detached
# -p 宿主机端口:容器端口: 为了映射当前Linux的端口和容器的端口
# --name 容器名称: 指定容器的名称
```

---

```sh
# 2. 查看正在运行的容器
docker ps [-qa]
# -q: 只查看容器的标识
# -a: 查看全部的容器，包括未运行的
```

---

```sh
# 3. 查看容器的日志
docker logs -f 容器id
# -f: 可以滚动查看日志最后几行
```

---

```sh
# 4. 进入到容器内部
docker exec -it 容器id /bin/bash
```

---

```sh
# 5. 删除容器（删除容器前，需要先停止容器）
docker stop 容器id
docker stop $(docker ps -qa)
docker rm 镜像id
docker rm $(docker ps -qa) # 删除全部容器
```

---

```sh
# 6. 启动容器
docker start 容器id
```



## 二、Docker 应用

### 2.1、准备要部署的项目工程

```sh
# MySQL数据库的连接中用户名和密码改变，要修改
```

### 2.2、准备MySQL容器

```sh
# 运行MySQL容器
docker run -d -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=root daocloud.io/library/mysql:5.7.4
```

### 2.3、准备Tomcat容器

```sh
# 运行Tomcat容器，只需要将项目的war包部署到Tomcat容器内部即可(war包针对java项目)
# 可以通过命令将宿主机的内容复制到容器内部
docker cp 文件名称 容器id:容器内部路径
# 举例
docker cp ssm.war fe:/usr/local/tomcat/webapps/
```

### 2.4、数据卷

>为了部署项目工程，需要使用到cp的命令将宿主机内的文件复制到容器内部，但是容器内部是很难修改文件的，如果项目更新，比较麻烦
>
>数据卷：将宿主机的一个目录映射到容器的一个目录中。
>
>可以在宿主机中操作目录中的内容，那么容器内部映射的文件，也会跟着一起改变

```sh
# 1. 创建数据卷
docker volume create 数据卷名称
# 创建数据卷之后，默认存放目录为 /var/lib/docker/volumes/数据卷名称/_data
```

---

```sh
# 2. 查看数据卷的详细信息
docker volume inspect 数据卷名称
```

---

```sh
# 3. 查看全部数据卷
docker volume ls
```

---

```sh
# 4. 删除数据卷
docker volume rm 数据卷名称
```



## 三、Docker自定义镜像



## 四、Docker-Compose



## 五、Docker DI、CD