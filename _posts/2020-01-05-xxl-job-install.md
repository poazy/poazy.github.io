---
layout: post
title:  "Docker 安装 xxl-job-admin"
date:   2020-01-05 00:00:00
categories: 调度任务
tags: XXL-JOB Docker Install
author: poazy
---

* content
{:toc}
> 基于 `docker 19.03.5` 版本创建并运行 `xuxueli/xxl-job-admin 2.1.2` 容器。



# 安装环境

## OS 版本

```bash
[root@centos7-qscft ~]# cat /etc/centos-release
CentOS Linux release 7.6.1810 (Core)
[root@centos7-qscft ~]#
```

## Docker 版本

```bash
[root@centos7-qscft ~]# docker -v
Docker version 19.03.5, build 633a0ea
[root@centos7-qscft ~]#
```



# xuxueli/xxl-job-admin 安装

## 配置数据库表

* 本次 `mysql 数据库` 采用 docker 方式搞了一个给使用
* 在此前  `mysql 数据库` 已安装准备好了

```bash
[root@centos7-qscft ~]# docker ps -a | grep mysql
6bc169b80e0a  mysql:8.0.18  "docker-entrypoint.s…"  2 hours ago  Up 2 hours  0.0.0.0:3306->3306/tcp, 33060/tcp  mysql
[root@centos7-qscft ~]#
```

### 下载 `tables_xxl_job.sql` 文件

*  通过 https://github.com/xuxueli/xxl-job/blob/master/doc/db/tables_xxl_job.sql 中 `Raw` 获取下载地址

* 下载地址：https://raw.githubusercontent.com/xuxueli/xxl-job/master/doc/db/tables_xxl_job.sql 

```bash
### （临时）设置 hosts 访问 raw.githubusercontent.com 教程：https://www.ioiox.com/archives/62.html
### 查询域名IP：https://www.ipaddress.com/
### 设置 hosts（不设置 hosts 的话 raw.githubusercontent.com 是访问不了的）
### 修改 hosts 文件添加一行内容：199.232.4.133 raw.githubusercontent.com
### 保存退出，就可以下了
vi /etc/hosts
#
### 下载 sql 脚本文件
wget https://raw.githubusercontent.com/xuxueli/xxl-job/master/doc/db/tables_xxl_job.sql
```

```bash
[root@centos7-qscft ~]# vi /etc/hosts
[root@centos7-qscft ~]# cat /etc/hosts
127.0.0.1       centos7-qscft   centos7-qscft
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
199.232.4.133 raw.githubusercontent.com
[root@centos7-qscft ~]# pwd
/root
[root@centos7-qscft ~]# ll -h
total 16K
-rw-------. 1 root root 5.5K Jun  2  2019 anaconda-ks.cfg
-rw-------. 1 root root 5.2K Jun  2  2019 original-ks.cfg
[root@centos7-qscft ~]# wget https://raw.githubusercontent.com/xuxueli/xxl-job/master/doc/db/tables_xxl_job.sql
--2020-01-05 15:10:48--  https://raw.githubusercontent.com/xuxueli/xxl-job/master/doc/db/tables_xxl_job.sql
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 199.232.4.133
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|199.232.4.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 6451 (6.3K) [text/plain]
Saving to: ‘tables_xxl_job.sql’

100%[==========================================================================================================================================================>] 6,451       --.-K/s   in 0.001s

2020-01-05 15:10:49 (8.55 MB/s) - ‘tables_xxl_job.sql’ saved [6451/6451]

[root@centos7-qscft ~]# ll -h
total 24K
-rw-------. 1 root root 5.5K Jun  2  2019 anaconda-ks.cfg
-rw-------. 1 root root 5.2K Jun  2  2019 original-ks.cfg
-rw-r--r--. 1 root root 6.3K Jan  5 15:10 tables_xxl_job.sql
[root@centos7-qscft ~]#
```

### 执行 `tables_xxl_job.sql` 文件

* 将下的 `tables_xxl_job.sql` 文件复制到 `mysql` 容器中目录

```bash
# 复制命令（复制到 mysql 容器的 /tmp 目录下）
docker cp tables_xxl_job.sql mysql:/tmp
```

```bash
[root@centos7-qscft ~]# pwd
/root
[root@centos7-qscft ~]# ll -h
total 24K
-rw-------. 1 root root 5.5K Jun  2  2019 anaconda-ks.cfg
-rw-------. 1 root root 5.2K Jun  2  2019 original-ks.cfg
-rw-r--r--. 1 root root 6.3K Jan  5 15:10 tables_xxl_job.sql
[root@centos7-qscft ~]# docker cp tables_xxl_job.sql mysql:/tmp
[root@centos7-qscft ~]# docker exec -it mysql ls -h /tmp
tables_xxl_job.sql
[root@centos7-qscft ~]#
```

* 执行 `tables_xxl_job.sql` 脚本创建库表

```bash
### 1、连接数据库（这里使用的是 docker 容器中的 mysql）
### 命令中第一个 mysql 为 docker 容器名，第二个 mysql 为 mysql 命令
docker exec -it mysql mysql -u root -p --default-character-set=utf8
### 输入正确密码，成功连接 mysql
#
### 2、source 命令创建 xxl-job 数据库表
source /tmp/tables_xxl_job.sql
```

```bash
[root@centos7-qscft ~]# docker exec -it mysql mysql -u root -p --default-character-set=utf8
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 18
Server version: 8.0.18 MySQL Community Server - GPL

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> source /tmp/tables_xxl_job.sql
Query OK, 1 row affected (0.01 sec)

Database changed
Query OK, 0 rows affected, 7 warnings (0.01 sec)

Query OK, 0 rows affected, 7 warnings (0.02 sec)

Query OK, 0 rows affected, 4 warnings (0.01 sec)

Query OK, 0 rows affected, 2 warnings (0.01 sec)

Query OK, 0 rows affected, 1 warning (0.02 sec)

Query OK, 0 rows affected, 3 warnings (0.01 sec)

Query OK, 0 rows affected, 2 warnings (0.02 sec)

Query OK, 0 rows affected (0.01 sec)

Query OK, 1 row affected (0.01 sec)

Query OK, 1 row affected (0.00 sec)

Query OK, 1 row affected (0.00 sec)

Query OK, 1 row affected (0.00 sec)

Query OK, 0 rows affected (0.00 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| xxl_job            |
+--------------------+
5 rows in set (0.00 sec)

mysql> show tables;
+--------------------+
| Tables_in_xxl_job  |
+--------------------+
| xxl_job_group      |
| xxl_job_info       |
| xxl_job_lock       |
| xxl_job_log        |
| xxl_job_log_report |
| xxl_job_logglue    |
| xxl_job_registry   |
| xxl_job_user       |
+--------------------+
8 rows in set (0.00 sec)

mysql> exit
Bye
[root@centos7-qscft ~]#
```

## 查找镜像

```
https://hub.docker.com/r/xuxueli/xxl-job-admin/tags
```

## 拉取镜像

```bash
### 拉取镜像，选择 2.1.2 版本
docker pull xuxueli/xxl-job-admin:2.1.2
```

## 运行镜像

```bash
docker run -d --restart=always --name xxl-job-admin \
	--net=host \
	-e PARAMS="--server.port=8180 --spring.datasource.url=jdbc:mysql://192.168.9.241:3306/xxl_job?Unicode=true&characterEncoding=UTF-8&allowMultiQueries=true&serverTimezone=Asia/Shanghai --spring.datasource.username=root --spring.datasource.password=mysql123 --spring.mail.host=smtp.126.com --spring.mail.username=boazy@126.com --spring.mail.password=admin123" \
	-e "TZ=Asia/Shanghai" \
	xuxueli/xxl-job-admin:2.1.2
#
### PAPAMS 属性信息可以通过 https://github.com/xuxueli/xxl-job/blob/master/xxl-job-admin/src/main/resources/application.properties 查看
```

```bash
[root@centos7-qscft ~]# docker ps -al | grep xxl-job-admin
fb811fcbebcf        xuxueli/xxl-job-admin:2.1.2   "sh -c 'java -jar /a…"   4 minutes ago       
Up 4 minutes        0.0.0.0:8180->8080/tcp   xxl-job-admin
[root@centos7-qscft ~]#
```

## 访问 xxl-job-admin

* 访问地址：http://192.168.9.241:8180/xxl-job-admin/ 
* （默认）用户名/密码：admin/123456



# xxl-job-executor-sample-springboot 安装

* 官方没有 xxl-job-executor-sample-xxx 镜像
* 自己搞吧

## 查询镜像

```

```

## 拉取镜像

```bash

```

## 运行镜像

```bash

```

```bash

```

