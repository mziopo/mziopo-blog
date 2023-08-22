---
title: Windows环境VM中安装Ubuntu 23.04
comments: true
toc: true
categories:
- Ubuntu
- VM
tags:
- Ubuntu
- VM
---

---

## 准备镜像

> https://mirrors.tuna.tsinghua.edu.cn/
>
> ubuntu-23.04-desktop-amd64.iso

---

## 安装VM17

> MC60H-DWHD5-H80U9-6V85M-8280D

---

## VM中安装Ubuntu

> https://blog.csdn.net/qyfx123456/article/details/130190155

```bash
#（vm中复制粘贴用）
sudo apt-get install open-vm-tools-desktop -y
```

---

## Ubuntu中安装Docker

> https://docs.docker.com/engine/install/ubuntu/

---

## Ubuntu中安装JDK8

```bash
sudo apt-get install openjdk-8-jdk
```

---

## Ubuntu中安装Redis

1. 安装

> https://developer.aliyun.com/article/764565

```bash
sudo apt install redis-server
```

2. 注释bind

```bash
vim /etc/redis/redis.conf
# bind 127.0.0.1
```

3. 设置密码

```bash
vim /etc/redis/redis.conf
# requierpass 123
```

4. 通过防火墙

```bash
sudo ufw allow proto tcp from 192.168.58.1/24 to any port 6379
```

5. 开机自启动

```bash
sudo systemctl enable redis-server
```

---

## Ubuntu中安装MySQL

```bash
sudo apt install mysql-server
# 防火墙 allow
sudo ufw allow mysql
sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
bind-address 0.0.0.0
# mysql
mysql
use mysql;
set global validate_password.policy=0;
set global validate_password.length=3;
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456';
update user set host = '%' where user='root';
FLUSH PRIVILEGES;
# restart
sudo systemctl restart mysql
```

---

## Docker中安装Nacos

```bash
vim Dokcerfile
# Dockerfile
FROM nacos/nacos-server:latest
EXPOSE 8848
ENV MODE standalone
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US.UTF-8
ENV LC_ALL en_US.UTF-8
CMD ["nacos"]
```

```bash
# build
docker build -t nacos .
```

```bash
# run
docker run --name nacos --restart=always -d -p 8848:8848 nacos
```

```bash
# ufw
sudo ufw allow proto tcp from 192.168.58.1/24 to any port 8848
```

```bash
# curl
curl localhost:8848/nacos
```