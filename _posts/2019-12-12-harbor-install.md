---
layout: post
title:  "Docker 安装 Harbor"
date:   2019-12-12 00:00:00
categories: Harbor
tags: Harbor Docker
author: poazy
---

* content
{:toc}
> 基于 `docker 19.03.5` 版本创建并运行 `harbor 1.10.0` 相关容器。



# 安装环境

## OS 版本

* CentOS Linux release 7.5.1804 (Core)

## Docker 版本

* Docker version 19.03.5, build 633a0ea



#   harbor 安装

* harbor 包含许多镜像，所以要采用 `Docker Compose` 方式安装
* 从官方下载

## 查找 harbor 版

```
https://github.com/goharbor/harbor/releases/
```

* 选择 `1.10.0` 版本

## 安装 harbor

### 创建数据目录

```bash
mkdir -p /boazy/data/dockerdata/harbor
```

### 切换目录

```bash
cd /boazy/data/dockerdata/harbor
```

### 下载 harbor

```bash
# 如果提示找不到 wget 命令，请通过 yum instal wget 安装
wget https://github.com/vmware/harbor/releases/download/v1.10.0/harbor-online-installer-v1.10.0.tgz
```

### 解压下载的文件

* 解压文件并将解压出来的 harbor 目录重命名为 installer

```bash
tar xvf harbor-online-installer-v1.10.0.tgz && mv harbor installer
```

* 执行结果

```bash
[root@centos7-qscft harbor]# tar xvf harbor-online-installer-v1.10.0.tgz && mv harbor installer
harbor/prepare
harbor/LICENSE
harbor/install.sh
harbor/common.sh
harbor/harbor.yml
[root@centos7-qscft harbor]# pwd
/boazy/data/dockerdata/harbor
[root@centos7-qscft harbor]#
```

* 查看目录

```bash
ll -h
```

```bash
[root@centos7-qscft harbor]# ll -h
total 12K
-rw-r--r--. 1 root root 8.3K Dec 18 14:11 harbor-online-installer-v1.10.0.tgz
drwxr-xr-x. 2 root root   89 Dec 18 14:30 installer
[root@centos7-qscft harbor]# pwd
/boazy/data/dockerdata/harbor
[root@centos7-qscft harbor]#
```

### 切换到解压的目录

```bash
cd /boazy/data/dockerdata/harbor/installer
```

* 查看目录下的文件

```bash
[root@centos7-qscft installer]# ll -h
total 32K
-rw-r--r--. 1 root root 3.4K Dec 13 13:24 common.sh
-rw-r--r--. 1 root root 5.8K Dec 13 13:24 harbor.yml
-rwxr-xr-x. 1 root root 2.3K Dec 13 13:24 install.sh
-rw-r--r--. 1 root root  12K Dec 13 13:24 LICENSE
-rwxr-xr-x. 1 root root 1.8K Dec 13 13:24 prepare
[root@centos7-qscft installer]# pwd
/boazy/data/dockerdata/harbor/installer
[root@centos7-qscft installer]#
```

### 修改 `harbor.yml` 文件的属性值

* 修改前备份文件

```bash
cp harbor.yml harbor.yml.bak191218
```

* 查看系统的 hostname

```bash
hostnamectl
```

* 修改说明

```
# 将 harbor.yml 中对应的属性值如下调整（根据自身情况设置值）
# 注释掉 https 下的 port、certificate、private_key 属性（暂不开启 https）
#https.port
#https.certificate
#https.private_key
hostname=centos7-qscft
http.port=8082
harbor_admin_password=boazy@191218.harbor
database.password=boazy@191218.harbor.db
data_volume=/boazy/data/dockerdata/harbor/data
log.local.location=/boazy/data/dockerdata/harbor/log
```

* 修改文件属性值（sed -i 替换命令）

```bash
# 修改 harbor.yml 文件，将对应的属性值调整，采用 sed -i 命令替换
# sed -i 's/Search_String/Replacement_String/g' Input_File
#
# 根据自身情况调整以下命令值后，执行以下命令修改替换
sed -i 's/hostname: reg.mydomain.com/hostname: centos7-qscft/g' harbor.yml
sed -i 's/port: 80/port: 8082/g' harbor.yml
sed -i 's/port: 443/#port: 443/g' harbor.yml
sed -i 's/certificate: /#certificate: /g' harbor.yml
sed -i 's/private_key: /#private_key: /g' harbor.yml
sed -i 's/harbor_admin_password: Harbor12345/harbor_admin_password: boazy@191218\.harbor/g' harbor.yml
sed -i 's/password: root123/password: boazy@191218\.harbor\.db/g' harbor.yml
sed -i 's/data_volume: \/data/data_volume: \/boazy\/data\/dockerdata\/harbor\/data/g' harbor.yml
sed -i 's/location: \/var\/log\/harbor/location: \/boazy\/data\/dockerdata\/harbor\/log/g' harbor.yml
```

* 检查修改替换是否正确

```bash
# 查看检查
cat harbor.yml
```

### 先执行 `./prepare` 

```bash
./prepare
```

```bash
[root@centos7-qscft installer]# ./prepare
prepare base dir is set to /dsmm/data/dockerdata/harbor/installer
Unable to find image 'goharbor/prepare:v1.10.0' locally
v1.10.0: Pulling from goharbor/prepare
b950b5dd94ab: Downloading [====================>                              ]  6.028MB/14.68MB
0acfdc1f2dfc: Downloading [==========>                                        ]  9.801MB/49MB
fd741c339942: Downloading [=========>                                         ]  4.468MB/23.39MB
dc5f326adae3: Waiting
6c159c23a3a1: Waiting
d08ec21dd899: Waiting
245c8635bc17: Waiting
v1.10.0: Pulling from goharbor/prepare
b950b5dd94ab: Pull complete
0acfdc1f2dfc: Pull complete
fd741c339942: Pull complete
dc5f326adae3: Pull complete
6c159c23a3a1: Pull complete
d08ec21dd899: Pull complete
245c8635bc17: Pull complete
Digest: sha256:261640c817df8b8a9daa7f88da93bf2b045defdba26ad08720085ea334c62a4e
Status: Downloaded newer image for goharbor/prepare:v1.10.0
WARNING:root:WARNING: HTTP protocol is insecure. Harbor will deprecate http protocol in the future. Please make sure to upgrade to https
Generated configuration file: /config/log/logrotate.conf
Generated configuration file: /config/log/rsyslog_docker.conf
Generated configuration file: /config/nginx/nginx.conf
Generated configuration file: /config/core/env
Generated configuration file: /config/core/app.conf
Generated configuration file: /config/registry/config.yml
Generated configuration file: /config/registryctl/env
Generated configuration file: /config/db/env
Generated configuration file: /config/jobservice/env
Generated configuration file: /config/jobservice/config.yml
Generated and saved secret to file: /secret/keys/secretkey
Generated certificate, key file: /secret/core/private_key.pem, cert file: /secret/registry/root.crt
Generated configuration file: /compose_location/docker-compose.yml
Clean up the input dir
[root@centos7-qscft installer]#
```

* 这个命令会生成 `docker-compose.yml` 文件和 `common` 目录

```bash
[root@centos7-qscft installer]# ll -h
total 48K
drwxr-xr-x. 3 root root   20 Dec 18 14:38 common
-rw-r--r--. 1 root root 3.4K Dec 13 13:24 common.sh
-rw-r--r--. 1 root root 5.5K Dec 18 14:38 docker-compose.yml
-rw-r--r--. 1 root root 5.9K Dec 18 14:37 harbor.yml
-rw-r--r--. 1 root root 5.8K Dec 18 14:34 harbor.yml.bak191218
-rwxr-xr-x. 1 root root 2.3K Dec 13 13:24 install.sh
-rw-r--r--. 1 root root  12K Dec 13 13:24 LICENSE
-rwxr-xr-x. 1 root root 1.8K Dec 13 13:24 prepare
[root@centos7-qscft installer]#
```

### 【可选】修改 `docker-compose.yml` 文件
* 可选，可选则修改，也可选则不修改

```bash
# 备份文件
cp docker-compose.yml docker-compose.yml.bak191218
# 修改文件，将相对路径 ./common/ 修改为 /boazy/data/dockerdata/harbor/installer/common/
sed -i 's/\.\/common\//\/boazy\/data\/dockerdata\/harbor\/installer\/common\//g' docker-compose.yml
```

* 修改前

```
[root@centos7-qscft installer]# cat docker-compose.yml |grep /common/
      - ./common/config/log/logrotate.conf:/etc/logrotate.d/logrotate.conf:z
      - ./common/config/log/rsyslog_docker.conf:/etc/rsyslog.d/rsyslog_docker.conf:z
      - ./common/config/registry/:/etc/registry/:z
      - ./common/config/registryctl/env
      - ./common/config/registry/:/etc/registry/:z
        source: ./common/config/registryctl/config.yml
      - ./common/config/db/env
      - ./common/config/core/env
      - ./common/config/core/certificates/:/etc/core/certificates/:z
        source: ./common/config/core/app.conf
      - ./common/config/jobservice/env
        source: ./common/config/jobservice/config.yml
      - ./common/config/nginx:/etc/nginx:z
[root@centos7-qscft installer]#
```

* 修改后

```
[root@centos7-qscft installer]# cat docker-compose.yml |grep /common/
      - /boazy/data/dockerdata/harbor/installer/common/config/log/logrotate.conf:/etc/logrotate.d/logrotate.conf:z
      - /boazy/data/dockerdata/harbor/installer/common/config/log/rsyslog_docker.conf:/etc/rsyslog.d/rsyslog_docker.conf:z
      - /boazy/data/dockerdata/harbor/installer/common/config/registry/:/etc/registry/:z
      - /boazy/data/dockerdata/harbor/installer/common/config/registryctl/env
      - /boazy/data/dockerdata/harbor/installer/common/config/registry/:/etc/registry/:z
        source: /boazy/data/dockerdata/harbor/installer/common/config/registryctl/config.yml
      - /boazy/data/dockerdata/harbor/installer/common/config/db/env
      - /boazy/data/dockerdata/harbor/installer/common/config/core/env
      - /boazy/data/dockerdata/harbor/installer/common/config/core/certificates/:/etc/core/certificates/:z
        source: /boazy/data/dockerdata/harbor/installer/common/config/core/app.conf
      - /boazy/data/dockerdata/harbor/installer/common/config/jobservice/env
        source: /boazy/data/dockerdata/harbor/installer/common/config/jobservice/config.yml
      - /boazy/data/dockerdata/harbor/installer/common/config/nginx:/etc/nginx:z
[root@centos7-qscft installer]# 
```

### 再执行 `./install.sh` 安装 harbor

*  `./install.sh`  会根据 docker-compose.yml 文件内容进行安装（拉取相关镜像及运行相关镜像）

```bash
./install.sh
```

* 结果成功（第一次会 pull 相关镜像的）

```bash
[root@centos7-qscft installer]# ./install.sh

[Step 0]: checking if docker is installed ...

Note: docker version: 19.03.5

[Step 1]: checking docker-compose is installed ...

Note: docker-compose version: 1.25.0


[Step 2]: preparing environment ...

[Step 3]: preparing harbor configs ...
prepare base dir is set to /boazy/data/dockerdata/harbor/installer
WARNING:root:WARNING: HTTP protocol is insecure. Harbor will deprecate http protocol in the future. Please make sure to upgrade to https
Clearing the configuration file: /config/log/logrotate.conf
Clearing the configuration file: /config/log/rsyslog_docker.conf
Clearing the configuration file: /config/nginx/nginx.conf
Clearing the configuration file: /config/core/env
Clearing the configuration file: /config/core/app.conf
Clearing the configuration file: /config/registry/config.yml
Clearing the configuration file: /config/registryctl/env
Clearing the configuration file: /config/registryctl/config.yml
Clearing the configuration file: /config/db/env
Clearing the configuration file: /config/jobservice/env
Clearing the configuration file: /config/jobservice/config.yml
Generated configuration file: /config/log/logrotate.conf
Generated configuration file: /config/log/rsyslog_docker.conf
Generated configuration file: /config/nginx/nginx.conf
Generated configuration file: /config/core/env
Generated configuration file: /config/core/app.conf
Generated configuration file: /config/registry/config.yml
Generated configuration file: /config/registryctl/env
Generated configuration file: /config/db/env
Generated configuration file: /config/jobservice/env
Generated configuration file: /config/jobservice/config.yml
loaded secret from file: /secret/keys/secretkey
Generated configuration file: /compose_location/docker-compose.yml
Clean up the input dir



[Step 4]: starting Harbor ...
Pulling log (goharbor/harbor-log:v1.10.0)...
v1.10.0: Pulling from goharbor/harbor-log
b950b5dd94ab: Already exists
b950b5dd94ab: Already exists
e7faa42698e5: Pull complete
c62ba13baa6a: Pull complete
5b66605d276a: Pull complete
f028d640f037: Pull complete
72cc133fdd38: Pull complete
262d8bda66e4: Pull complete
d5791461e6cc: Pull complete
Digest: sha256:9e5a4a81751cb8b8f3ced4f6dfc0f2e888b2f1319bb94992318a620f0a9058ec
Status: Downloaded newer image for goharbor/harbor-log:v1.10.0
Pulling registry (goharbor/registry-photon:v2.7.1-patch-2819-2553-v1.10.0)...
v2.7.1-patch-2819-2553-v1.10.0: Pulling from goharbor/registry-photon
b950b5dd94ab: Already exists
b950b5dd94ab: Already exists
4315a9eaa1df: Pull complete
fad1e50e6f96: Pull complete
cc524298aa8f: Pull complete
fcb55cf49807: Pull complete
cff39f92fcc8: Pull complete
Digest: sha256:873c73954aaae94267cecdeee83676c04a03dbb3230473ba478ec94fa6299549
Status: Downloaded newer image for goharbor/registry-photon:v2.7.1-patch-2819-2553-v1.10.0
Pulling registryctl (goharbor/harbor-registryctl:v1.10.0)...
v1.10.0: Pulling from goharbor/harbor-registryctl
b950b5dd94ab: Already exists
5478e1e20a37: Pull complete
a9bbd19c4f15: Pull complete
0f7f23d1525a: Pull complete
00aee17e8d59: Pull complete
f365720d48f2: Pull complete
9847a9856d7f: Pull complete
Digest: sha256:15ad1012f31a6ba37fbeb66f66fa5ba7fd93081bb1968ed630a349dc4792d04a
Status: Downloaded newer image for goharbor/harbor-registryctl:v1.10.0
Pulling postgresql (goharbor/harbor-db:v1.10.0)...
v1.10.0: Pulling from goharbor/harbor-db
b950b5dd94ab: Already exists
b950b5dd94ab: Already exists
a66af5ebde94: Pull complete
05611b2cb74d: Pull complete
c2ec687f7795: Pull complete
a516bbf5835b: Pull complete
eac82587810b: Pull complete
5cdc9f9fbaab: Pull complete
b97542249e3d: Pull complete
cb1d1ff41157: Pull complete
Digest: sha256:6dbbd03abbd8d18f2692ba8f89fdeb55cf18d41029aefcce47b522cf3ea967c2
Status: Downloaded newer image for goharbor/harbor-db:v1.10.0
Pulling portal (goharbor/harbor-portal:v1.10.0)...
v1.10.0: Pulling from goharbor/harbor-portal
b950b5dd94ab: Already exists
eb4d63ffe3c3: Pull complete
b7eb3d9757c6: Pull complete
8a9ae53938d2: Pull complete
6b361634d09a: Pull complete
49d6df61d284: Pull complete
366c13a6be9c: Pull complete
Digest: sha256:a143f4b76fbf8def8ca4cdd37be84bc3aaa0c018a4b8b272a25f0e7b78059553
Status: Downloaded newer image for goharbor/harbor-portal:v1.10.0
Pulling redis (goharbor/redis-photon:v1.10.0)...
v1.10.0: Pulling from goharbor/redis-photon
b950b5dd94ab: Already exists
b950b5dd94ab: Already exists
530a1659421b: Pull complete
c9adf29f84d5: Pull complete
ac30c4f4350e: Pull complete
e2a312c02f45: Pull complete
Digest: sha256:2d01e205a2bcab2fe41270669053103dac95cb950fe1bc08d2e92ee7f047a11f
Status: Downloaded newer image for goharbor/redis-photon:v1.10.0
Pulling core (goharbor/harbor-core:v1.10.0)...
v1.10.0: Pulling from goharbor/harbor-core
b950b5dd94ab: Already exists
3a55daf6690a: Pull complete
1d94c9f673e2: Pull complete
fb4a8e96737f: Pull complete
6327d82ea67b: Pull complete
74385b23d541: Pull complete
Digest: sha256:a97d36e3b26b908dd6212500ed27f6d710042466d46e3585581b56f83029d885
Status: Downloaded newer image for goharbor/harbor-core:v1.10.0
Pulling jobservice (goharbor/harbor-jobservice:v1.10.0)...
v1.10.0: Pulling from goharbor/harbor-jobservice
b950b5dd94ab: Already exists
b950b5dd94ab: Already exists
4e5e30f0e23e: Already exists
14b8a2732a83: Already exists
Digest: sha256:7a4678b7e34dd18c530891bee63342fa8ffe0db6fb7fd8c580e093cbcd5e5f8a
Status: Downloaded newer image for goharbor/harbor-jobservice:v1.10.0
Pulling proxy (goharbor/nginx-photon:v1.10.0)...
v1.10.0: Pulling from goharbor/nginx-photon
b950b5dd94ab: Already exists
b950b5dd94ab: Already exists
08229261eeba: Already exists
Digest: sha256:af95713bd4e1f5732b96d35ccb01ec97ee56c8a99da584cf34009287d03897d0
Status: Downloaded newer image for goharbor/nginx-photon:v1.10.0
Creating harbor-log ... done
Creating harbor-portal ... done
Creating redis         ... done
Creating harbor-db     ... done
Creating registry      ... done
Creating registryctl   ... done
Creating harbor-core   ... done
Creating nginx             ... done
Creating harbor-jobservice ... done
✔ ----Harbor has been installed and started successfully.----
[root@centos7-qscft installer]#
```

* 查看运行状态

```bash
[root@centos7-qscft installer]# docker ps -a
CONTAINER ID        IMAGE                                                     COMMAND                  CREATED             STATUS                       PORTS                                                            NAMES
a88bce4651a9        goharbor/harbor-jobservice:v1.10.0                        "/harbor/harbor_jobs…"   4 minutes ago       Up 4 minutes (healthy)                                                                        harbor-jobservice
a32bd323f86a        goharbor/nginx-photon:v1.10.0                             "nginx -g 'daemon of…"   4 minutes ago       Up 4 minutes (healthy)       0.0.0.0:8082->8080/tcp                                           nginx
a187efb3f4d7        goharbor/harbor-core:v1.10.0                              "/harbor/harbor_core"    4 minutes ago       Up 4 minutes (healthy)                                                                        harbor-core
605c637a707a        goharbor/registry-photon:v2.7.1-patch-2819-2553-v1.10.0   "/home/harbor/entryp…"   4 minutes ago       Up 4 minutes (healthy)       5000/tcp                                                         registry
6d79c65ba5bd        goharbor/harbor-registryctl:v1.10.0                       "/home/harbor/start.…"   4 minutes ago       Up 4 minutes (healthy)                                                                        registryctl
6fdd2cb70d3d        goharbor/redis-photon:v1.10.0                             "redis-server /etc/r…"   4 minutes ago       Up 4 minutes (healthy)       6379/tcp                                                         redis
6a73fbe425b7        goharbor/harbor-db:v1.10.0                                "/docker-entrypoint.…"   4 minutes ago       Up 4 minutes (healthy)       5432/tcp                                                         harbor-db
ff7a44a924ca        goharbor/harbor-portal:v1.10.0                            "nginx -g 'daemon of…"   4 minutes ago       Up 4 minutes (healthy)       8080/tcp                                                         harbor-portal
877e28133251        goharbor/harbor-log:v1.10.0                               "/bin/sh -c /usr/loc…"   4 minutes ago       Up 4 minutes (healthy)       127.0.0.1:1514->10514/tcp                                        harbor-log
[root@centos7-qscft installer]#
```

* 访问一下地址（成功访问）

```bash
[root@centos7-qscft installer]# curl http://centos7-qscft:8082
<!doctype html>
<html>

<head>
    <meta charset="utf-8">
    <title>Harbor</title>
    <base href="/">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="icon" type="image/x-icon" href="favicon.ico?v=2">
<link rel="stylesheet" href="styles.701dc5ee3007bd83bfa4.css"></head>

<body>
    <harbor-app>
        <div class="spinner spinner-lg app-loading">
            Loading...
        </div>
    </harbor-app>
<script src="runtime.9ad22a88fcc70a015907.js" defer></script><script src="polyfills-es5.d01e8ad6bc0c07b49ab6.js" nomodule defer></script><script src="scripts.7fa3fa51e1a86dfba2c8.js" defer></script><script src="main.b0eb2486094b1093b347.js" defer></script></body>

</html>[root@centos7-qscft installer]#
```



## harbor 安装所拉取的镜像

```bash
[root@centos7-qscft installer]# docker images
REPOSITORY                    TAG                              IMAGE ID            CREATED             SIZE
goharbor/redis-photon         v1.10.0                          6df66e5c1ca7        12 days ago         111MB
goharbor/harbor-registryctl   v1.10.0                          c550280445e6        12 days ago         104MB
goharbor/registry-photon      v2.7.1-patch-2819-2553-v1.10.0   2115e08fa399        12 days ago         86.5MB
goharbor/nginx-photon         v1.10.0                          f7ed614c3abc        12 days ago         44MB
goharbor/harbor-log           v1.10.0                          fb15f6772e9a        12 days ago         82.3MB
goharbor/harbor-jobservice    v1.10.0                          d6d4f2b125f6        12 days ago         142MB
goharbor/harbor-core          v1.10.0                          f3a3065b3af2        12 days ago         128MB
goharbor/harbor-portal        v1.10.0                          fbaeb1fdacad        12 days ago         52.1MB
goharbor/harbor-db            v1.10.0                          634404a417cf        12 days ago         148MB
goharbor/prepare              v1.10.0                          927062458494        12 days ago         149MB
[root@centos7-qscft installer]#
```



## 其他命令

```bash
# 重启 harbor 容器
docker restart harbor-jobservice nginx harbor-core registryctl registry redis harbor-db harbor-portal harbor-log
# 删除 harbor 容器
docker rm -f harbor-jobservice nginx harbor-core registry registryctl redis harbor-db harbor-portal harbor-log
```

