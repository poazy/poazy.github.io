---
layout: post
title:  "Docker 安装 Flowable"
date:   2020-07-19 23:00:00
categories: Flowable
tags: Flowable
author: poazy
---

* content
{:toc}
> 根据官方 `flowable/flowable-rest` 和 `flowable/all-in-one` 镜像 `6.5.0` 版本安装 Flowable；
>
> `flowable/all-in-one` 中包含 flowable-idm、flowable-admin、flowable-task 和 flowable-modeler 四个应用；
>
> 使用 MySQL 8.0 数据库，解决 docker 安装下找不到 MySQL 驱动包问题。







# 前言

* 数据库这里采用 `MySQL 8.0` ，数据库信息如下：

  > 官方默认采用的是 H2 数据库，官方默认也没有集居 MySQL 驱动包，需要自己加载 MySQL 驱动包的。
  
  ```
  数据库地址端口：192.168.9.241:3306
  数据库库名：flowable
账号/密码：root/xxxx
  ```
  
* 本次在 192.168.9.241 服务器上的 `docker` 环境安装  `flowable/flowable-rest` 和 `flowable/all-in-one`  组件

* `flowable/flowable-rest` 组件的端口定义为 **`9900`**

  > 镜像地址

  ```bash
  https://hub.docker.com/r/flowable/flowable-rest
  ```

  > 安装后的访问地址：

  ```bash
  http://192.168.9.241:9900/flowable-rest/docs
  ```

* `flowable/all-in-one` 组件的端口定义为 **`9901`**

  > 镜像地址
  
  ```bash
  https://hub.docker.com/r/flowable/all-in-one
  ```
  
  > 安装后的访问地址：
  
  ```bash
  http://192.168.9.241:9901/flowable-idm
  http://192.168.9.241:9901/flowable-admin
  http://192.168.9.241:9901/flowable-task
  http://192.168.9.241:9901/flowable-modeler
  ```



# 创建数据库

## 创建 flowable 库

* 库中没有 flowable 库，则要先创建 flowable 库

```bash
CREATE DATABASE `flowable` CHARACTER SET 'utf8mb4' COLLATE 'utf8mb4_unicode_ci';
```

## 下载 MySQL 8.0 驱动

```bash
mkdir -p /boazy/data/dockerdata/flowable/lib && \
cd /boazy/data/dockerdata/flowable/lib && \
curl -L https://repo1.maven.org/maven2/mysql/mysql-connector-java/8.0.21/mysql-connector-java-8.0.21.jar -o mysql-connector-java-8.0.21.jar && \
cd /boazy/data/dockerdata/flowable && ls -h /boazy/data/dockerdata/flowable/lib
```

## 授权目录

```bash
chown -R 1000:1000 /boazy/data/dockerdata/flowable
```



# 创建配置 env file

* docker --env-file 的配置文件
* 当然完全可以放在一个配置文件，而无需搞 N 个文件

## datasource.env

* 数据源配置文件

```bash
# 创建 datasource.env 文件并编辑 datasource.env 文件
mkdir -p /boazy/data/dockerdata/flowable/env && \
vi /boazy/data/dockerdata/flowable/env/datasource.env
```

> 根据 `前言` 章节信息创建并编辑 `datasource.env` 文件内容
>
> `datasource.env` 文件内容如下：

```bash
SPRING_DATASOURCE_DRIVER-CLASS-NAME=com.mysql.cj.jdbc.Driver
SPRING_DATASOURCE_URL=jdbc:mysql://192.168.9.241:3306/flowable?useUnicode=true&characterEncoding=utf8
&allowMultiQueries=true&serverTimezone=Asia/Shanghai&nullCatalogMeansCurrent=true
SPRING_DATASOURCE_USERNAME=root
SPRING_DATASOURCE_PASSWORD=mysq@xxx
```

## flowable-rest.env

```bash
# 创建 flowable-rest.env 文件并编辑 flowable-rest.env 文件
mkdir -p /boazy/data/dockerdata/flowable/env && \
vi /boazy/data/dockerdata/flowable/env/flowable-rest.env
```

> `flowable-rest.env` 文件内容如下：

```bash
FLOWABLE_REST_APP_ADMIN_USER-ID=rest-admin
FLOWABLE_REST_APP_ADMIN_PASSWORD=xXx72@xxx
FLOWABLE_REST_APP_ADMIN_FIRST-NAME=Rest
FLOWABLE_REST_APP_ADMIN_LAST-NAME=Admin
```

## flowable-idm.env

```bash
# 创建 flowable-idm.env 文件并编辑 flowable-idm.env 文件
mkdir -p /boazy/data/dockerdata/flowable/env && \
vi /boazy/data/dockerdata/flowable/env/flowable-idm.env
```

> `flowable-idm.env` 文件内容如下：

```bash
FLOWABLE_IDM_APP_ADMIN_USER-ID=admin
FLOWABLE_IDM_APP_ADMIN_PASSWORD=xXx72@admin-xxx
FLOWABLE_IDM_APP_ADMIN_FIRST-NAME=boazy
FLOWABLE_IDM_APP_ADMIN_LAST-NAME=Administrator
FLOWABLE_IDM_APP_ADMIN_EMAIL=boazy@126.com
```

## flowable-common.env

```bash
# 创建 flowable-common.env 文件并编辑 flowable-common.env 文件
mkdir -p /boazy/data/dockerdata/flowable/env && \
vi /boazy/data/dockerdata/flowable/env/flowable-common.env
```

> `flowable-common.env` 文件内容如下：

```bash
FLOWABLE_COMMON_APP_IDM-URL=http://192.168.9.241:9901/flowable-idm
FLOWABLE_COMMON_APP_IDM-ADMIN_USER=admin
FLOWABLE_COMMON_APP_IDM-ADMIN_PASSWORD=xXx72@admin
```

## flowable-admin.env

```bash
# 创建 flowable-admin 文件并编辑 flowable-admin 文件
mkdir -p /boazy/data/dockerdata/flowable/env && \
vi /boazy/data/dockerdata/flowable/env/flowable-admin.env
```

> `flowable-admin.env` 文件内容如下：

```bash
FLOWABLE_ADMIN_APP_SERVER-CONFIG_PROCESS_SERVER-ADDRESS=http://192.168.9.241
FLOWABLE_ADMIN_APP_SERVER-CONFIG_PROCESS_PORT=9900
FLOWABLE_ADMIN_APP_SERVER-CONFIG_PROCESS_CONTEXT-ROOT=flowable-rest
FLOWABLE_ADMIN_APP_SERVER-CONFIG_PROCESS_REST-ROOT=service
FLOWABLE_ADMIN_APP_SERVER-CONFIG_PROCESS_USER-NAME=rest-admin
FLOWABLE_ADMIN_APP_SERVER-CONFIG_PROCESS_password=xXx72@xxx
FLOWABLE_ADMIN_APP_SERVER-CONFIG_CMMN_SERVER-ADDRESS=http://192.168.9.241
FLOWABLE_ADMIN_APP_SERVER-CONFIG_CMMN_PORT=9900
FLOWABLE_ADMIN_APP_SERVER-CONFIG_CMMN_CONTEXT-ROOT=flowable-rest
FLOWABLE_ADMIN_APP_SERVER-CONFIG_CMMN_REST-ROOT=cmmn-api
FLOWABLE_ADMIN_APP_SERVER-CONFIG_CMMN_USER-NAME=rest-admin
FLOWABLE_ADMIN_APP_SERVER-CONFIG_CMMN_password=xXx72@xxx
FLOWABLE_ADMIN_APP_SERVER-CONFIG_DMN_SERVER-ADDRESS=http://192.168.9.241
FLOWABLE_ADMIN_APP_SERVER-CONFIG_DMN_PORT=9900
FLOWABLE_ADMIN_APP_SERVER-CONFIG_DMN_CONTEXT-ROOT=flowable-rest
FLOWABLE_ADMIN_APP_SERVER-CONFIG_DMN_REST-ROOT=dmn-api
FLOWABLE_ADMIN_APP_SERVER-CONFIG_DMN_USER-NAME=rest-admin
FLOWABLE_ADMIN_APP_SERVER-CONFIG_DMN_password=xXx72@xxx
FLOWABLE_ADMIN_APP_SERVER-CONFIG_FORM_SERVER-ADDRESS=http://192.168.9.241
FLOWABLE_ADMIN_APP_SERVER-CONFIG_FORM_PORT=9900
FLOWABLE_ADMIN_APP_SERVER-CONFIG_FORM_CONTEXT-ROOT=flowable-rest
FLOWABLE_ADMIN_APP_SERVER-CONFIG_FORM_REST-ROOT=form-api
FLOWABLE_ADMIN_APP_SERVER-CONFIG_FORM_USER-NAME=rest-admin
FLOWABLE_ADMIN_APP_SERVER-CONFIG_FORM_password=xXx72@xxx
FLOWABLE_ADMIN_APP_SERVER-CONFIG_CONTENT_SERVER-ADDRESS=http://192.168.9.241
FLOWABLE_ADMIN_APP_SERVER-CONFIG_CONTENT_PORT=9900
FLOWABLE_ADMIN_APP_SERVER-CONFIG_CONTENT_CONTEXT-ROOT=flowable-rest
FLOWABLE_ADMIN_APP_SERVER-CONFIG_CONTENT_REST-ROOT=content-api
FLOWABLE_ADMIN_APP_SERVER-CONFIG_CONTENT_USER-NAME=rest-admin
FLOWABLE_ADMIN_APP_SERVER-CONFIG_CONTENT_password=xXx72@xxx
FLOWABLE_ADMIN_APP_SERVER-CONFIG_APP_SERVER-ADDRESS=http://192.168.9.241
FLOWABLE_ADMIN_APP_SERVER-CONFIG_APP_PORT=9900
FLOWABLE_ADMIN_APP_SERVER-CONFIG_APP_CONTEXT-ROOT=flowable-rest
FLOWABLE_ADMIN_APP_SERVER-CONFIG_APP_REST-ROOT=app-api
FLOWABLE_ADMIN_APP_SERVER-CONFIG_APP_USER-NAME=rest-admin
FLOWABLE_ADMIN_APP_SERVER-CONFIG_APP_password=xXx72@xxx
```

## flowable-modeler.env

```bash
# 创建 flowable-modeler 文件并编辑 flowable-modeler 文件
mkdir -p /boazy/data/dockerdata/flowable/env && \
vi /boazy/data/dockerdata/flowable/env/flowable-modeler.env
```

> `flowable-modeler.env` 文件内容如下：

```bash
FLOWABLE_MODELER_APP_DEPLOYMENT-API-URL=192.168.9.241:9900/flowable-rest/app-api
```

## list env file

```bash
ll -h /boazy/data/dockerdata/flowable/env
```



# 安装 flowable-rest

* 运行 flowable/flowable-rest 容器

> `--restart always` 可选，正式部署要添加上此参数

```bash
docker run -d \
-p 9900:8080 \
--name=flowable-rest \
-v /boazy/data/dockerdata/flowable/lib:/javalib \
--env-file=env/datasource.env \
--env-file=env/flowable-rest.env \
--entrypoint "" \
flowable/flowable-rest:6.5.0 java -p /javalib -jar /app.war
```

> 成功安装后，数据库 flowable 库中会自动创建 78 张表，当正常启动和 78 张表创建完后再访问！

* 成功安装可访问

```bash
地址：http://192.168.9.241:9900/flowable-rest/docs
账号密码：rest-admin/xXx72@xxx
```

>查看容器运行情况：
>
>```bash
>[root@centos7-qscft flowable]# docker ps
>CONTAINER ID        IMAGE                          COMMAND                  CREATED             STATUS              PORTS                    NAMES
>596e684c62b4        flowable/flowable-rest:6.5.0   "java -p /javalib -j…"   40 minutes ago      Up 40 minutes       0.0.0.0:9900->8080/tcp   flowable-rest
>[root@centos7-qscft flowable]#
>```



# 安装 all-in-one

> Include Flowable apps
>
> * Flowable IDM
> * Flowable Admin
> * Flowable Task
> * Flowable Modeler

* 运行 flowable/all-in-one 容器

> `--restart always` 可选，正式部署要添加上此参数

```bash
docker run -d --name flowable-aio \
-p 9901:8080 \
-v /boazy/data/dockerdata/flowable/lib/mysql-connector-java-8.0.21.jar:/opt/tomcat/lib/mysql-connector-java-8.0.21.jar \
--env-file=env/datasource.env \
--env-file=env/flowable-rest.env \
--env-file=env/flowable-idm.env \
--env-file=env/flowable-common.env \
--env-file=env/flowable-admin.env \
--env-file=env/flowable-modeler.env \
flowable/all-in-one:6.5.0
```

* 成功安装后可访问

```bash
http://192.168.9.241:9901/flowable-idm
http://192.168.9.241:9901/flowable-admin
http://192.168.9.241:9901/flowable-task
http://192.168.9.241:9901/flowable-modeler
账号/密码：admin/xXx72@admin-xxx
```

>查看容器运行情况：
>
>```bash
>[root@centos7-qscft flowable]# docker ps
>CONTAINER ID        IMAGE                          COMMAND                  CREATED             STATUS              PORTS                    NAMES
>0c97ff9bd133        flowable/all-in-one:6.5.0      "/opt/tomcat/bin/cat…"   37 minutes ago      Up 37 minutes       0.0.0.0:9901->8080/tcp   flowable-aio
>596e684c62b4        flowable/flowable-rest:6.5.0   "java -p /javalib -j…"   40 minutes ago      Up 40 minutes       0.0.0.0:9900->8080/tcp   flowable-rest
>[root@centos7-qscft flowable]#
>```



# 附录-安装 Flowable UI

* 本章节读者 `可选读` ；
* 本章节可理解为，独立安装各个 Flowable UI 应用程序；
* 本章节的 `Flowable UI` 安装与前面介绍的 `all-in-one` 安装不同；
* 本章节的 `Flowable UI` 安装方式与前面介绍的 `all-in-one` 安装方式 `根据自身情况选用`；
* `all-in-one` 镜像为一个镜像中包括了 flowable-idm、flowable-admin、flowable-task 和 flowable-modeler 四个 UI 应用程序，一个 Tomcat 下部署；
* 而本章节的 `Flowable UI` 安装，是对 flowable-idm、flowable-admin、flowable-task 和 flowable-modeler 四个 UI 应用程序各自独立的镜像安装，各自拥有独立的端口。

## 安装 flowable-idm

* 运行 flowable-idm 容器

> `--restart always` 可选，正式部署要添加上此参数

```bash
docker run -d \
-p 9901:8080 \
--name=flowable-idm \
-v /boazy/data/dockerdata/flowable/lib:/javalib \
--env-file=env/datasource.env \
--env-file=env/flowable-idm.env \
--entrypoint "" \
flowable/flowable-idm:6.5.0 java -p /javalib -jar /app.war
```

* 成功安装可访问

```bash
地址：http://192.168.9.241:9901/flowable-idm
账号密码：admin/xXx72@admin-xxx
```

## 安装 flowable-admin

* 运行 flowable-admin 容器

> `--restart always` 可选，正式部署要添加上此参数

```bash
docker run -d \
-p 9902:9988 \
--name=flowable-admin \
-v /boazy/data/dockerdata/flowable/lib:/javalib \
--env-file=env/datasource.env \
--env-file=env/flowable-common.env \
--env-file=env/flowable-admin.env \
--entrypoint "" \
flowable/flowable-admin:6.5.0 java -p /javalib -jar /app.war
```

* 成功安装可访问

```bash
地址：http://192.168.9.241:9902/flowable-admin
账号密码：admin/xXx72@admin-xxx
```

## 安装 flowable-task

* 运行 flowable-task 容器

> `--restart always` 可选，正式部署要添加上此参数

```bash
docker run -d \
-p 9903:9999 \
--name=flowable-task \
-v /boazy/data/dockerdata/flowable/lib:/javalib \
--env-file=env/datasource.env \
--env-file=env/flowable-common.env \
--entrypoint "" \
flowable/flowable-task:6.5.0 java -p /javalib -jar /app.war
```

* 成功安装可访问

```bash
地址：http://192.168.9.241:9902/flowable-task
账号密码：admin/xXx72@admin-xxx
```

## 安装 flowable-modeler

* 运行 flowable-modeler 容器

> `--restart always` 可选，正式部署要添加上此参数

```bash
docker run -d \
-p 9904:8888 \
--name=flowable-modeler \
-v /boazy/data/dockerdata/flowable/lib:/javalib \
--env-file=env/datasource.env \
--env-file=env/flowable-common.env \
--env-file=env/flowable-modeler.env \
--entrypoint "" \
flowable/flowable-modeler:6.5.0 java -p /javalib -jar /app.war
```

* 成功安装可访问

```bash
地址：http://192.168.9.241:9904/flowable-modeler
账号密码：admin/xXx72@admin-xxx
```



# 附录-Spring Boot

```xml
<!-- 需要导入数据库数据表 -->
<dependency>
    <groupId>org.flowable</groupId>
    <artifactId>flowable-spring-boot-starter</artifactId>
    <version>6.5.0</version>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.21</version>
</dependency>
```

