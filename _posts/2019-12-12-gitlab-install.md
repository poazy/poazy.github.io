---
layout: post
title:  "Docker 安装 GitLab"
date:   2019-12-12 00:00:00
categories: GitLab
tags: GitLab Docker Install
author: poazy
---

* content
{:toc}
> 基于 `docker 19.03.5` 版本创建并运行 `gitlab/gitlab-ce 12.5.4` 容器。一些问题和配置的处理。



# 安装环境

## OS 版本

* CentOS Linux release 7.5.1804 (Core)

## Docker 版本

* Docker version 19.03.5, build 633a0ea



# gitlab/gitlab-ce 安装

* 官方推介配置为 2核CPU4G可用内存

## 查找镜像版本

* 通过官网URL查找所需要的版本

```
https://hub.docker.com/r/gitlab/gitlab-ce/tags
```

* 这里选择 `gitlab/gitlab-ce:12.5.4-ce.0` 版本

##  创建并运行容器

```bash
# 宿主机创建数据目录
mkdir -p /boazy/data/dockerdata/gitlab
# 拉取镜像（可跳过）
docker pull gitlab/gitlab-ce:12.5.4-ce.0

# 创建并运行镜像容器
docker run -d \
    --hostname gitlab.boazy.com \
    -P -p 443:443 -p 80:80 -p 2280:22 \
    --name gitlab \
    --restart always \
    --privileged=true \
    -v /boazy/data/dockerdata/gitlab/etc:/etc/gitlab \
    -v /boazy/data/dockerdata/gitlab/log:/var/log/gitlab \
    -v /boazy/data/dockerdata/gitlab/data:/var/opt/gitlab \
    -v /etc/localtime:/etc/localtime:ro \
    gitlab/gitlab-ce:12.5.4-ce.0
```

## 调整 `SSH` 方式 `git clone` 的端口

* 如果宿主机 `22` 端口没有被占用，则直接采用 `-p 22:22` 端口映射时可跳这一章节的设置！

  ```url
  # 22 端口的 git clone 地址
  git@gitlab.boazy.com:itp/mgds.git
  ```

* 因为 `22` 端口被宿主机占了，所以上面的端口映射采用的是 `-p 2280:22`，所以要调整 `gitlab` 中的配置为 `2280` 端口

  ```url
  # 非 22 端口的 git clone 地址
  ssh://git@gitlab.boazy.com:2280/itp/mgds.git
  ```

### 修改 Gitlab 配置文件中的 SSH 端口

```bash
# /boazy/data/dockerdata/gitlab/etc/gitlab.rb 宿主机路径等于 /etc/gitlab/gitlab.rb
# 修改 /etc/gitlab/gitlab.rb 中的：# gitlab_rails['gitlab_shell_ssh_port'] = 22
# 修改为：gitlab_rails['gitlab_shell_ssh_port'] = 2280

# 切换到配置文件目录
cd /boazy/data/dockerdata/gitlab/etc/
# 备份文件
cp gitlab.rb gitlab.rb.bak0
# 修改文件，改为：gitlab_rails['gitlab_shell_ssh_port'] = 2280
# 采用 sed 命令替换修改
sed -i "s/# gitlab_rails\['gitlab_shell_ssh_port'\] = 22/gitlab_rails\['gitlab_shell_ssh_port'\] = 2280/g" gitlab.rb
# 查看修改后的结果
cat gitlab.rb | grep 2280

# 修改后重启容器 或 刷新 gitlab 配置
docker restart gitlab
docker exec -it gitlab gitlab-ctl reconfigure
```

