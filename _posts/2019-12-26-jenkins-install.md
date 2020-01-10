---
layout: post
title:  "Docker 安装 Jenkins"
date:   2019-12-26 00:00:00
categories: Jenkins
tags: Jenkins Docker
author: poazy
---

* content
{:toc}
> 基于 `docker 19.03.5` 版本创建并运行 `jenkins/jenkins:2.210` 容器。



# 安装环境

## OS 版本

* CentOS Linux release 7.5.1804 (Core)

## Docker 版本

* Docker version 19.03.5, build 633a0ea



# jenkins/jenkins 安装 

## 查找镜像版本

```
https://hub.docker.com/r/jenkins/jenkins/tags
```

* 选择 `jenkins/jenkins:2.210` 版本

## 拉取 jenkins 镜像

```bash
# 拉取镜像（可跳过）
docker pull jenkins/jenkins:2.210
```

## 创建 jenkins_home 目录

```bash
# 宿主机创建数据目录
mkdir -p /boazy/data/dockerdata/jenkins_home
# 将宿主机数据目录给容器权限（不权限会启动不了：Permission denied）
chown -R 1000:1000 /boazy/data/dockerdata/jenkins_home/
```

## 创建并运行容器

```bash
# 创建并运行镜像容器
docker run -d --restart=always --name jenkins \
    -p 8088:8080 \
    -v /boazy/data/dockerdata/jenkins_home:/var/jenkins_home \
    -t jenkins/jenkins:2.210
```

## 配置修改插件镜像地址

* 正常创建并运容器后先不要访问 jenkins，先进行`插件镜像地址`修改
* 目的加速插件下载安装，如果不修改会大量插件下载安装失败

### a) 查询加速插件镜像地址

```
http://mirrors.jenkins-ci.org/status.html
```

* 这里选择中国的地址（ [清华大学开源软件镜像站](https://mirrors.tuna.tsinghua.edu.cn/) ）

```
https://mirrors.tuna.tsinghua.edu.cn/jenkins/
```

```
https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json
```

### b) 切换到 `jenkins_home` 目录

```bash
cd /boazy/data/dockerdata/jenkins_home
```

### c) 替换插件地址

* 地址在 `/boazy/data/dockerdata/jenkins_home/hudson.model.UpdateCenter.xml` 文件

```bash
# 将 https://updates.jenkins.io/update-center.json 修改为 https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json
# 这里采用 sed 命令替换方式修改
sed -i 's/https:\/\/updates.jenkins.io\/update-center.json/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins\/updates\/update-center.json/g' hudson.model.UpdateCenter.xml
```

### d) 重启 jekinks

```bash
docker restart jenkins
```

### e) 浏览器访问 jenkins 并登陆

* 浏览器访问 jenkins
  * http://192.168.9.241:8088

* 查看初始登陆密码

```bash
cat /boazy/data/dockerdata/jenkins_home/secrets/initialAdminPassword
```

* 使用初始登陆密码登陆
* 当浏览器页面显示 “自定义Jenkins” 页面时。不要再在浏览器中操作了。这个时候 `jenkins_home` 目录下会生成 `updates` 目录 和 `updates/default.json` 文件
* 接着到 SSH 端执行命令，修改 `default.json` 文件，见下一章节

### f) 修改 default.json 文件

* 真正加速 jenkins 下载及安装插件的速度，在这里！

* 文件路径

```
/boazy/data/dockerdata/jenkins_home/updates/default.json
```

* 将 `default.json` 文件的地址 `http://updates.jenkins-ci.org/download` 改为 `https://mirrors.tuna.tsinghua.edu.cn/jenkins` ，如果不改插件下载安装还是会出错失败的

```bash
# 查看 default.json 文件中的 http://updates.jenkins-ci.org/download，替换前可以查询到相关的信息
cat /boazy/data/dockerdata/jenkins_home/updates/default.json | grep http://updates.jenkins-ci.org/download
# 
# 采用 sed 命令修改替换
sed -i 's/http:\/\/updates.jenkins-ci.org\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' updates/default.json
# 
# 查看 default.json 文件中的 http://updates.jenkins-ci.org/download
# 正常替换之后是查不到相关信息了
cat /boazy/data/dockerdata/jenkins_home/updates/default.json | grep http://updates.jenkins-ci.org/download
# 这个时候可以查询到 https://mirrors.tuna.tsinghua.edu.cn/jenkins 的信息，表示替换成功咯
cat updates/default.json | grep https://mirrors.tuna.tsinghua.edu.cn/jenkins
```

### g) 重启 jenkins

```bash
docker restart jenkins
```

### h) 浏览器访问 jenkins 并登陆

* 浏览器访问 jenkins
  * http://192.168.9.241:8088
* 查看初始登陆密码

```bash
cat /boazy/data/dockerdata/jenkins_home/secrets/initialAdminPassword
```

* 使用初始登陆密码登陆
* 当浏览器页面显示 “自定义Jenkins” 页面时，点左边选项 “安装推荐插件” 进行插件下载安装
* 哇！推荐插件一下子就下载完及安装好了，进入了  “创建第一个管理员用户”  页面

### i) 到此  “安装推荐插件” 成功完成

* 到此已成功修改插件镜像地址
* 到此已成功下载并安装好插件（速度飞快！）