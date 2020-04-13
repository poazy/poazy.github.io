---
layout: post
title:  "Github 进行 fork 后与原仓库同步"
date:   2020-04-13 20:20:00
categories: Github
tags: Github fork
author: poazy
---

* content
{:toc}

> Github 进行 fork 后与原仓库同步，fork 没更改的情况下。




# Github 进行 fork 后与原仓库同步

```bash
# 克隆上游仓库
# git clone <upstream_repo_url>
git clone https://github.com/xxx/yyy-plugin.git
# 切换到克隆项目本地目录
cd yyy-plugin
# 添加远程地址
# git remote add fork <fork_repo_url>
git remote add fork https://github.com/aaa/yyy-plugin.git
# fetch 上游仓库
git fetch origin

# origin/master -> fork/master 同步
# 重置本地 master 分支为上游仓库的版本
# git reset --hard origin/<branch>
git reset --hard origin/master
# 将本地 master 推送到 fork 项目 master 上 （origin/master -> fork/master）
# git push -u fork <branch>
git push -u fork master

# origin/dev -> fork/dev 同步
# 本地切换到 dev 分支
# git checkout -b <branch>
git checkout -b dev
# 重置本地 dev 分支为上上游仓库的版本
# git reset --hard origin/<branch>
git reset --hard origin/dev
# 将本地 dev 推送到 fork 项目 dev 上 （origin/dev -> fork/dev）
# git push -u fork <branch>
git push -u fork dev

# 切换回 master 分支
# git checkout master
# 查看分支
# git branch -a
```


http://www.qtcn.org/bbs/simple/?t53628.html



