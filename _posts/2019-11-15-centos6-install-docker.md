---
layout: post
title:  "CentOS 6 安装 Docker"
date:   2019-11-15 00:00:00
categories: Docker
tags: CentOS Docker
author: poazy
---

* content
{:toc}
> CentOS 6 安装 `Docker` 和 `Docker Compose` 。



# 安装 Docker

* 相关参考文章

https://www.cnblogs.com/maodot/p/7654918.html 

https://blog.51cto.com/fatty/1766055 

https://www.cnblogs.com/YatHo/p/8275559.html 

https://www.cnblogs.com/sdadx/p/10016427.html 

## 安装依懒（可选）

```bash
yum install -y yum-utils device-mapper-persistent-data lvm2
```

## 更新 `yum` 软件源缓存（可选）

```bash
yum makecache fast
```

## 安装 Docker

```bash
# 安装 docker 1.7.1
yum install https://get.docker.com/rpm/1.7.1/centos-6/RPMS/x86_64/docker-engine-1.7.1-1.el6.x86_64.rpm
```

## 启动 Docker

```bash
# 启动 docker 并设置 docker 开启启动
service docker start && chkconfig docker on
```

## 配置镜像加速

* 这里使用阿里云镜像加速（地址从自己的账号中获取）

```bash
vi /etc/sysconfig/docker
```

```bash
# 使用阿里云帐号的镜像加速地址（创建阿里云到 容器镜像服务->镜像中心->镜像加速器 中查看）
other_args="--registry-mirror=https://xxx.mirror.aliyuncs.com"
```

```bash
# 重启 docker
service docker restart
# 并查看进程，发现已经改掉
ps -ef|grep docker
```

* 还可以选择国内其他镜像加速地址

```bash
https://dockerhub.azk8s.cn
https://reg-mirror.qiniu.com
```

## 安装 Docker  Compose

* 查看版本

```html
https://docs.docker.com/compose/compose-file/compose-versioning/

https://github.com/docker/compose/releases
```

* 选择 `1.5.2` 版本，1.5.2 才兼容 Docker 1.7.1
* 安装 Docker Compose

```bash
# 拉取文件放到指定目录（下载好慢，网上还有其他的安装方式）
curl -L https://github.com/docker/compose/releases/download/1.5.2/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

# 给文件授权
chmod +x /usr/local/bin/docker-compose

# 查看版本
docker-compose --version
```
