---
layout: post
title:  "Docker 安装 Sonatype Nexus 3"
date:   2019-12-12 00:00:00
categories: Nexus
tags: Nexus Docker Install
author: poazy
---

* content
{:toc}
> 基于 `docker 19.03.5` 版本创建并运行 `sonatype/nexus3 3.19.1` 容器。



# 安装环境

## OS 版本

* CentOS Linux release 7.5.1804 (Core)

## Docker 版本

* Docker version 19.03.5, build 633a0ea



# sonatype/nexus3 安装 

## 查找镜像版本

```
https://hub.docker.com/r/sonatype/nexus3/tags
```

* 选择 `sonatype/nexus3:3.19.1` 版本

## 创建并运行容器

```bash
# 宿主机创建数据目录
mkdir -p /boazy/data/dockerdata/nexus-data
# 将宿主机数据目录给容器权限（不权限会启动不了：Permission denied）
chown -R 200 /boazy/data/dockerdata/nexus-data/

# 拉取镜像（可跳过）
docker pull sonatype/nexus3:3.19.1
# 创建并运行镜像容器
docker run -d \
	-P -p 8081:8081 -p 22881:22 \
	--name nexus3 \
	--restart=always \
	--privileged=true \
	-v /boazy/data/dockerdata/nexus-data:/nexus-data \
	-e INSTALL4J_ADD_VM_PARAMS="-Duser.timezone=Asia/Shanghai" \
	sonatype/nexus3:3.19.1
```

## 运行问题

* 查看日志

```bash
docker logs nexus3
```

* 日志片段（Permission denied）

```bash
Error creating bundle cache.
Unable to update instance pid: Unable to create directory /nexus-data/instances
mkdir: cannot create directory '../sonatype-work/nexus3/log': Permission denied
mkdir: cannot create directory '../sonatype-work/nexus3/tmp': Permission denied
OpenJDK 64-Bit Server VM warning: Cannot open file ../sonatype-work/nexus3/log/jvm.log due to No such file or directory

Warning:  Cannot open log file: ../sonatype-work/nexus3/log/jvm.log
Warning:  Forcing option -XX:LogFile=/tmp/jvm.log
java.io.FileNotFoundException: ../sonatype-work/nexus3/tmp/i4j_ZTDnGON8hezynsMX2ZCYAVDtQog=.lock (No such file or directory)
        at java.io.RandomAccessFile.open0(Native Method)
        at java.io.RandomAccessFile.open(RandomAccessFile.java:316)
        at java.io.RandomAccessFile.<init>(RandomAccessFile.java:243)
        at com.install4j.runtime.launcher.util.SingleInstance.check(SingleInstance.java:72)
        at com.install4j.runtime.launcher.util.SingleInstance.checkForCurrentLauncher(SingleInstance.java:31)
        at com.install4j.runtime.launcher.UnixLauncher.checkSingleInstance(UnixLauncher.java:88)
        at com.install4j.runtime.launcher.UnixLauncher.main(UnixLauncher.java:67)
java.io.FileNotFoundException: /nexus-data/karaf.pid (Permission denied)
```

* 将宿主机数据目录给容器权限（解决方法）

```bash
chown -R 200 /boazy/data/dockerdata/nexus-data/
```


