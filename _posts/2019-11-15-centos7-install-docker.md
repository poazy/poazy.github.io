---
layout: post
title:  "CentOS 7 安装 Docker"
date:   2019-11-15 23:35:08
categories: JavaScript
tags: JavaScript Ajax URL HistoryApi pushState pjax
---

* content
{:toc}

[TOC]

# 安装 Docker

## 卸载 Docker

* 之前环境没有安装过 Docker 跳过

```bash
yum remove docker \
    docker-client \
    docker-client-latest \
    docker-common \
    docker-latest \
    docker-latest-logrotate \
    docker-logrotate \
    docker-engine \
    docker-ce-cli
```



## 安装 Docker

### 安装依懒

```bash
yum install -y yum-utils device-mapper-persistent-data lvm2
```

### 设置 Docker 仓库

* 选择阿里国内加速地址

```bash
# 阿里云（推介使用这个）
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

* 其他可选国内加速地址

```bash
# 
yum-config-manager \
	--add-repo https://mydream.ink/utils/container/docker-ce.repo
# 中科大
yum-config-manager \
	--add-repo https://mirrors.ustc.edu.cn/docker-ce/linux/centos/docker-ce.repo
# 官方
yum-config-manager \
    --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

### 更新 `yum` 软件源缓存

```bash
yum makecache fast
```

### 安装 Docker

```bash
yum install docker-ce docker-ce-cli containerd.io
```

### 启动 Docker

```bash
# 启动 docker 并设置 docker 开启启动
systemctl start docker && systemctl enable docker
```

### 配置镜像加速

* 这里使用阿里云镜像加速（地址从自己的账号中获取）

```bash
vi /etc/docker/daemon.json
```

```bash
{
  "registry-mirrors": [
    "https://xxx.mirror.aliyuncs.com"
  ]
}
```

```bash
# 重新加载 daemon 配置并重启 docker
systemctl daemon-reload && systemctl restart docker
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

* 选择 `1.25.0` 版本
* 安装 Docker Compose

```bash
# 拉取文件放到指定目录（下载好慢，网上还有其他的安装方式）
curl -L "https://github.com/docker/compose/releases/download/1.25.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# 给文件授权
chmod +x /usr/local/bin/docker-compose

# 查看版本
docker-compose --version
```



## 安装问题

### 问题1（设置 Docker 仓库）

* 错误代码

```bash
Could not fetch/save url https://download.docker.com/linux/centos/docker-ce.repo to file /etc/yum.repos.d/docker-ce.repo: [Errno 14] curl#60 - "Peer's Certificate has expired."
```

* 问题解决（同步时间）

```bash
# 安装 ntp （服务器可能没有安装 ntp，那执行 ntpdate 会报找不到命令）
yum install -y ntp
# 同步时间并写入硬件
ntpdate pool.ntp.org && hwclock -w
```

### 问题2（搜索镜像）

* 错误代码

```bash
Error response from daemon: Get https://index.docker.io/v1/search?q=redis&n=25: x509: certificate has expired or is not yet valid^C
```

* 问题解决（配置镜像加速）

* 问题解决（同步时间）

```bash
# 安装 ntp （服务器可能没有安装 ntp，那执行 ntpdate 会报找不到命令）
yum install -y ntp
# 同步时间并写入硬件
ntpdate pool.ntp.org && hwclock -w
```
