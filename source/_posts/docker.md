---
title: Docker 笔记
comments: true
toc: true
categories:
- Docker
tags:
- Docker
typora-root-url: ..
---

---

## README

本篇主要记录Docker学习笔记

---

## docker pull $image

```bash
docker pull mysql
# Using default tag: latest
# latest: Pulling from library/mysql
# b193354265ba: Pull complete # 分层下载 联合文件系统
# 14a15c0bb358: Already exists # 分层下载 联合文件系统 可以和其他镜像共用
# 02da291ad1e4: Pull complete 
# 9a89a1d664ee: Pull complete 
# a24ae6513051: Pull complete 
# b85424247193: Pull complete 
# 9a240a3b3d51: Pull complete 
# 8bf57120f71f: Pull complete 
# c64090e82a0b: Pull complete 
# af7c7515d542: Pull complete 
# Digest: sha256:c0455ac041844b5e65cd08571387fa5b50ab2a6179557fd938298cab13acf0dd # 签名
# Status: Downloaded newer image for mysql:latest
# docker.io/library/mysql:latest #真实地址 等价 docker pull mysql
```

---

## 两种进入容器的方法

### 进入容器内部 开启一个新终端

```bash
docker exec -it $container /bin/bash
```

### 进入容器内部 进入正在执行的容器终端 不开启新终端

```bash
docker attach $container
```

---

## 拷贝文件

```bash
docker cp $container:$path $path
```

拷贝容器内文件到主机



---

## 运行挂载容器卷

```bash
docker run -it -v $name:/etc/nginx:ro nginx --volumes-from mysql
```

具名挂载：默认目录 /var/lib/docker/volumes/$name/_data

匿名挂载：不填`$name`:

`:ro` readonly 只能容器外部修改

`:rw` readwrite

`--volumes-from` 跟随挂载 例子可以实现多个mysql容器数据同步



---

## docker inspect $container

信息



---

## docker history $image

查看镜像历史构建步骤



---

## Dockerfile

| 命令       | 说明                                   |
| ---------- | -------------------------------------- |
| ADD        | 添加文件进去                           |
| COPY       | 添加文件进去                           |
| RUN        | 镜像构建时运行的命令                   |
| CMD        | 容器启动时运行的命令，会被外部参数替换 |
| ENTRYPOINT | 容器启动时运行的命令，可以追加         |
| ENV        | 构建时设置环境变量                     |

---

### CMD 和 ENTRYPOINT 区别

两者都是设置某容器启动时要执行的命令

举例如下：

构建 centos 镜像，打印当前目录

```bash
# Dockerfile
FROM centos
CMD ["ls","-a"]
```

```bash
docker build .
docker run $container -l
```

<u>CMD</u>

会报错，因为 `ls -a` 被替换成了 `-l`

<u>ENTRYPOINT</u>

无报错，可以正常打印当前目录信息，因为 `ls -a` 被追加成了 `ls -la`



---

### Dockerfile 例子

```bash
FROM python:3.7-alpine
WORKDIR /code
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
EXPOSE 5000
COPY . .
CMD ["flask", "run"]
```

---

## 网络

![image-20230821160035004](/images/docker-net.png)



---

### 容器互联

`--link`

只是在host中添加配置，不推荐使用。

```bash
docker run -d -P --name centos01 --link centos02 centos
docker run -d -P --name centos02 centos
# Ping 成功
docker exec -it centos01 ping centos02
# Ping 失败 因为02 host 中没配置01的信息
docker exec -it centos02 ping centos01
```

---

### 自定义网络

```bash
docker network create --driver=bridge --subnet=192.168.0.0/16 --gateway=192.168.0.1 my-net
docker run -d -P --name=centos01 --net=mynet centos
docker run -d -P --name=centos02 --net=mynet centos
# 自定义网络可以直接ping名字，不需要--link
docker exec -it centos01 ping centos02
```

自定义网络中每个节点关系已经自动维护好了。

不同的集群，比如`redis`集群或者`mysql`集群，使用不同的自定义网络，互相隔离开来。



---

### 网络联通

```bash
# 将centos01放到my-net下
# 一个容器两个ip
docker network connect my-net centos01
```

---

## Compose

### 安装

```bash
# 下载
curl -L https://get.daocloud.io/docker/compose/releases/download/1.26.2/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
# 授权
chmod +x /usr/local/bin/docker-compose
# v
docker-compose version
```

### 命令

```bash
# up
docker-compose up -d
# down
docker-compose down
```

### 样例

```yml
version: "3"

services:
  db:
    image: mysql:5.7
    volumes: # 挂载
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: somewordpress # root用户 的 密码
      MYSQL_DATABASE: wordpress # 数据库名
      MYSQL_USER: wordpress # 数据库用户
      MYSQL_PASSWORD: wordpress # 数据库密码

  wordpress:
    depends_on: # 依赖项
      - db
    image: wordpress:latest
    volumes:
      - wordpress_data:/var/www/html
    ports:
      - "8000:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
volumes:
  db_data: { }
  wordpress_data: { }
```

---

## Swarm

准备四台服务器

简化版K8S

---

### 设置主节点

```bash
docker swarm init --advertise-addr 192.168.64.134
```

---

### 生成令牌

```bash
# 生成管理节点的令牌
docker swarm join-token manager
# 生成工作节点的令牌
docker swarm join-token worker
```

---

### 设置工作节点

```bash
# 135
docker swarm join --token SWMTKN-1-56fs1oaww5jkhsq7f3v2f3fazgm1jgxrvjc1n4cttab8v6bmr8-04efiil88xkppw186vk82y5f1 192.168.64.135:2377
# 136
docker swarm join --token SWMTKN-1-56fs1oaww5jkhsq7f3v2f3fazgm1jgxrvjc1n4cttab8v6bmr8-04efiil88xkppw186vk82y5f1 192.168.64.136:2377
```

---

### 设置管理节点

```bash
# 137
docker swarm join --token SWMTKN-1-56fs1oaww5jkhsq7f3v2f3fazgm1jgxrvjc1n4cttab8v6bmr8-4bocoqdhfypuppql3yqxt0u2p 192.168.64.134:2377
```

---

### 查看所有节点

```bash
[root@192 ~]# docker node ls
# ID                            HOSTNAME         STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
# n2ws0fm7fvsiniuo9wjmq3dfh *   192.168.64.134   Ready     Active         Leader           20.10.12
# y1clyylemoiqbrquha4hpgdg7     192.168.64.135   Ready     Active                          20.10.12
# 29g0dvdvrtajsxr5d6lgutddr     192.168.64.136   Ready     Active                          20.10.12
# mbtufs92q1q1kbsrdkfh3gx7s     192.168.64.137   Ready     Active         Reachable        20.10.12
```

---

### Raft协议

Raft协议：保证大多数节点存活才可用

三个管理器的集群最多可以容忍一个节点的丢失。
五个管理器的集群可以容忍最大同时丢失两个节点。
N个管理器集群最多可以容忍丢失(N-1)/2个节点。
Docker 建议一个集群有七个节点。

---

### 创建服务

```bash
docker service create -p 80:80 nginx
docker service ls
# ID             NAME       MODE         REPLICAS   IMAGE          PORTS
# cccvbtrm38lt   nginx      replicated   1/1        nginx:latest   *:8888->80/tcp
```

---

### 动态扩容

```bash
# docker service scale nginx=3 
docker service update --replicas 3 nginx
# nginx
# overall progress: 3 out of 3 tasks 
# 1/3: running   
# 2/3: running   
# 3/3: running   
# verify: Service converged 
```

---

### 动态缩容

```bash
docker service scale nginx=1
```