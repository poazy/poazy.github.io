---
layout: post
title:  "Docker 安装 Nacos 基于 CentOS 6"
date:   2019-12-17 00:00:00
categories: Nacos
tags: Nacos Docker Install
author: poazy
---

* content
{:toc}
> 基于 `docker 1.7.1` 版本创建并运行 `nacos/nacos-server 1.1.4` 容器。



# 安装环境

## OS 版本

* CentOS release 6.5 (Final)

## Docker 版本

* Docker version 1.7.1, build 786b29d



# nacos/nacos-server 集群安装

## 查找 nacos 镜像

```
https://hub.docker.com/r/nacos/nacos-server/tags
```

## 拉取镜像

```bash
### 选择 1.1.4 版本
### 192.168.0.79、192.168.0.82 和 192.168.0.82 三台服务器均要拉取）
docker pull nacos/nacos-server:1.1.4
```

## 安装 nacos/nacos-server

```bash
### 192.168.0.79 和 192.168.0.82 两台服务安装运行 nacos
docker run -d --restart=always --name nacos \
--net=host \
-v /boazy/data/dockerdata/nacos/logs:/home/nacos/logs \
-v /boazy/data/dockerdata/nacos/plugins/mysql:/home/nacos/plugins/mysql \
--env NACOS_SERVER_PORT=8838 --env MODE=cluster \
--env NACOS_SERVERS="192.168.0.79:8838 192.168.0.82:8838 192.168.0.95:8838" \
--env NACOS_SERVER_IP= \
--env MYSQL_DATABASE_NUM=1 \
--env MYSQL_MASTER_SERVICE_HOST=192.168.0.85 \
--env MYSQL_MASTER_SERVICE_PORT=3306 \
--env MYSQL_MASTER_SERVICE_DB_NAME=nacos_config \
--env MYSQL_MASTER_SERVICE_USER=root \
--env MYSQL_MASTER_SERVICE_PASSWORD=XXyyZZ \
nacos/nacos-server:1.1.4
### 192.168.0.95 服务器安装运行 nacos（多网卡指定 IP 地址，不然集群节点列表发现不了）
docker run -d --restart=always --name nacos \
--net=host \
-v /boazy/data/dockerdata/nacos/logs:/home/nacos/logs \
-v /boazy/data/dockerdata/nacos/plugins/mysql:/home/nacos/plugins/mysql \
--env NACOS_SERVER_PORT=8838 --env MODE=cluster \
--env NACOS_SERVERS="192.168.0.79:8838 192.168.0.82:8838 192.168.0.95:8838" \
--env NACOS_SERVER_IP=192.168.0.95 \
--env MYSQL_DATABASE_NUM=1 \
--env MYSQL_MASTER_SERVICE_HOST=192.168.0.85 \
--env MYSQL_MASTER_SERVICE_PORT=3306 \
--env MYSQL_MASTER_SERVICE_DB_NAME=nacos_config \
--env MYSQL_MASTER_SERVICE_USER=root \
--env MYSQL_MASTER_SERVICE_PASSWORD=XXyyZZ \
nacos/nacos-server:1.1.4
```

# nacos 集群代理（nginx）安装 

* 采用 nginx 作为 naocs 集群反向代理

## 查找 nginx 镜像

```
https://hub.docker.com/_/nginx?tab=tags
```

## 拉取 nginx 镜像

```bash
### CentOS 6 内核好像不支持新版本的 nginx（1.17.x 和 1.16.x 版本会报 FATAL: kernel too old）
### 192.168.0.79 服务器拉取 nginx 镜像（反向代理放在 79 服务器上）
docker pull nginx:1.15.12
```

## 创建目录创建文件

### 创建目录

```bash
### 删除目录（请注意，是删除！）
rm -rf /boazy/data/dockerdata/nacos-ngx
### 创建目录
mkdir -p /boazy/data/dockerdata/nacos-ngx/{html,conf,conf.d,log}
```

### 宿主机创建 nginx.conf 文件

* 【运行容器前可选】如果不事先创建 `nginx.conf` 文件，容器启动会报错（是由于挂载  `nginx.conf` 引起的）

```bash
Error response from daemon: Cannot start container efad0bc874256425122673d33d68e6f09e9b0c838945d6c24b65d0c7e03ff3dd: [8] System error: not a directory
```

* 【运行容器前可选】删除 `/boazy/data/dockerdata/nacos-ngx` 目录下的 `nginx.conf` 目录

```bash
### nginx.conf 这个目录是容器创建的，nginx.conf 是个文件来的
rm -rf /boazy/data/dockerdata/nacos-ngx/nginx.conf
```

* 在宿主机上创建 `nginx.conf` 文件
* 准备 `nginx.conf` 文件放到宿主机 `/boazy/data/dockerdata/nacos-ngx` 目录下，`nginx.conf` 文件内容如下（这个文件内容可以运行一个不拦截文件目录的容器，要后从这个容器中取出来）：

```bash
vi /boazy/data/dockerdata/nacos-ngx/nginx.conf
```

```nginx.conf

user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```

* 【运行容器前可选】再重新启动 `nacos-ngx` 容器就可正常运行了

### 宿主机创建 `default.conf` 文件

* 如果不事先创建 `default.conf` 文件，容器正式启动后会出现访问不到的问题
* 在宿主机上创建 `default.conf` 文件
* 准备 `default.conf` 文件放到宿主机 `/boazy/data/dockerdata/nacos-ngx/conf.d` 目录下，`default.conf ` 文件内容如下（这个文件内容可以运行一个不拦截文件目录的容器，要后从这个容器中取出来）：

```bash
vi /boazy/data/dockerdata/nacos-ngx/conf.d/default.conf
```

```
server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}

```

## 安装运行 nginx

```bash
### nacos-ngx 安装（192.168.0.79）
docker run -d --restart=always --name nacos-ngx -p 8848:80 \
-v /boazy/data/dockerdata/nacos-ngx/html:/usr/share/nginx/html \
-v /boazy/data/dockerdata/nacos-ngx/conf:/etc/nginx/conf \
-v /boazy/data/dockerdata/nacos-ngx/conf.d:/etc/nginx/conf.d \
-v /boazy/data/dockerdata/nacos-ngx/log:/var/log/nginx \
-v /boazy/data/dockerdata/nacos-ngx/nginx.conf:/etc/nginx/nginx.conf \
-e "TZ=Asia/Shanghai" \
nginx:1.15.12
```

## 配置 nacos 代理

* 添加 nacos cluster

```bash
vi /boazy/data/dockerdata/nacos-ngx/conf.d/nacos_cluster.conf
```

```
# nacos cluster 节点
upstream nacos_cluster {
    server 192.168.0.79:8838;
    server 192.168.0.82:8838;
    server 192.168.0.95:8838;
}
```

* 修改 `default.conf` ，注释原 `location /` 节点，重新配置 `location /` 节点指定 nacos cluster。调整如下：

```bash
vi /boazy/data/dockerdata/nacos-ngx/conf.d/default.conf
```

```
server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    #location / {
    #    root   /usr/share/nginx/html;
    #    index  index.html index.htm;
    #}

    location / {
        proxy_pass http://nacos_cluster;
        # 请使用 $http_host 别使用 $host
        # 否则访问 http://127.0.0.1:8848/nacos 会重定向到 http://127.0.0.1/nacos/ 打不开
        proxy_set_header Host $http_host;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}

```

* 刷新 nginx 配置，拿刚刚修改的配置失效

```bash
docker exec -it nacos-ngx nginx -s reload
```
