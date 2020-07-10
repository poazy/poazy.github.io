---
layout: post
title:  "Docker 安装 MongoDB"
date:   2020-07-01 00:00:00
categories: MongoDB
tags: MongoDB
author: poazy
---

* content
{:toc}
> docker 简单安装 MongoDB





> 安装命令
```bash
docker run -d --name mongodb \
    -p 27017:27017 \
    -v /boazy/data/mongodb/data/configdb:/data/configdb \
    -v /boazy/data/mongodb/data/db:/data/db \
    mongo:4.2.8
```

> 进入 mongodb 容器

```bash
docker exec -it mongodb bash
```

> 给 admin 库设置用户名和密码
```bash
mongo
use admin
db.createUser({
 user: "root",
 pwd: "mongodb123",
 roles: [{role: "userAdminAnyDatabase", db: "admin"}]})
```