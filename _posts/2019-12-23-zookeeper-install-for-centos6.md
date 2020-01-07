---
layout: post
title:  "Docker 安装 Zookeeper"
date:   2019-12-23 00:00:00
categories: Zookeeper
tags: Zookeeper Docker Install
author: poazy
---

* content
{:toc}

# 安装环境

## OS 版本

* CentOS release 6.5 (Final)

## Docker 版本

* Docker version 1.7.1, build 786b29d



# zookeeper 集群安装

## 查找 zookeeper 镜像

```url
https://hub.docker.com/_/zookeeper?tab=tags
```

## 拉取 zookeeper 镜像

* 选择 `3.4.13` 版本
* `3.4.14` 或 `3.5.x` 版本容器运行会报 `FATAL: kernel too old` 错误启动不了！

```bash
# 192.168.0.79、192.168.0.82、192.168.0.95 三台服务器均拉取
docker pull zookeeper:3.4.13
```

## 创建目录创建文件

### 创建目录

```bash
# 【没事没执行 rm 命令】注意这是删除目录！
rm -rf /boazy/data/dockerdata/zookeeper/*
# 创建目录
# 192.168.0.79、192.168.0.82、192.168.0.95 三台服务器均创建目录
mkdir -p /boazy/data/dockerdata/zookeeper/{data,datalog,conf}
```

### 创建文件

* 因为有对 `zoo.cfg` 文件的挂载，如果不事先创建好，容器运行时会报 `/docker-entrypoint.sh: line 15: /conf/zoo.cfg: Is a directory` 错误

```bash
# 192.168.0.79、192.168.0.82、192.168.0.95 三台服务器均创建文件
vi /boazy/data/dockerdata/zookeeper/conf/zoo.cfg
```

```zoo.cfg
clientPort=2181
dataDir=/data
dataLogDir=/datalog
tickTime=2000
initLimit=5
syncLimit=2
autopurge.snapRetainCount=3
autopurge.purgeInterval=0
maxClientCnxns=60
server.1=192.168.0.79:2888:3888
server.2=192.168.0.82:2888:3888
server.3=192.168.0.95:2888:3888
```

## 安装运行 zookeeper

* 采用 `--net=host` 网络模式，以避免 `3888` 端口不通问题

```bash
### 192.168.0.79 服务器创建运行容器
docker run -d --restart always --name zookeeper \
--net=host \
-v /boazy/data/dockerdata/zookeeper/data:/data \
-v /boazy/data/dockerdata/zookeeper/datalog:/datalog \
-v /boazy/data/dockerdata/zookeeper/conf/zoo.cfg:/conf/zoo.cfg \
-e "TZ=Asia/Shanghai" \
-e "ZOO_MY_ID=1" \
-e "ZOO_SERVERS=server.1=192.168.0.79:2888:3888 server.2=192.168.0.82:2888:3888 server.3=192.168.0.95:2888:3888" \
zookeeper:3.4.13

### 192.168.0.82 服务器创建运行容器
docker run -d --restart always --name zookeeper \
--net=host \
-v /boazy/data/dockerdata/zookeeper/data:/data \
-v /boazy/data/dockerdata/zookeeper/datalog:/datalog \
-v /boazy/data/dockerdata/zookeeper/conf/zoo.cfg:/conf/zoo.cfg \
-e "TZ=Asia/Shanghai" \
-e "ZOO_MY_ID=2" \
-e "ZOO_SERVERS=server.1=192.168.0.79:2888:3888 server.2=192.168.0.82:2888:3888 server.3=192.168.0.95:2888:3888" \
zookeeper:3.4.13

### 192.168.0.95 服务器创建运行容器
docker run -d --restart always --name zookeeper \
--net=host \
-v /boazy/data/dockerdata/zookeeper/data:/data \
-v /boazy/data/dockerdata/zookeeper/datalog:/datalog \
-v /boazy/data/dockerdata/zookeeper/conf/zoo.cfg:/conf/zoo.cfg \
-e "TZ=Asia/Shanghai" \
-e "ZOO_MY_ID=3" \
-e "ZOO_SERVERS=server.1=192.168.0.79:2888:3888 server.2=192.168.0.82:2888:3888 server.3=192.168.0.95:2888:3888" \
zookeeper:3.4.13
```









