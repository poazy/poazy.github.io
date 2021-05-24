---
layout: post
title:  "CentOS 7 开机启动执行脚本"
date:   2021-05-24 23:50:00
categories: CentOS
tags: CentOS CentOS7 SH
author: poazy
---

* content
{:toc}
> CentOS 7 设置开机启动时执行指定的 sh 脚本文件。






## 修改 `rc.local` 方式
### 对 sh 脚本文件授权
```bash
chmod +x /poazy/middleware/hugegraph/hugegraph-0.11.2/bin/start-hugegraph.sh
chmod +x /poazy/middleware/hugegraph/hugegraph-studio-0.11.0/bin/hugegraph-studio.sh
chmod +x /poazy/middleware/hugegraph/hugegraph-hubble-1.5.0/bin/start-hubble.sh
```
### 对 `rc.local` 授权
```bash
# 这一步必不可少，否则开机启动不成功
chmod +x /etc/rc.d/rc.local
```
### 在 `rc.local` 中添加启动
```bash
vi /etc/rc.d/rc.local
```
> 添加内容如下：
```bash
su - root -c 'systemctl restart network'
su - root -c 'sh /poazy/middleware/hugegraph/hugegraph-0.11.2/bin/start-hugegraph.sh'
su - root -c 'nohup /poazy/middleware/hugegraph/hugegraph-studio-0.11.0/bin/hugegraph-studio.sh &'
su - root -c 'sh /poazy/middleware/hugegraph/hugegraph-hubble-1.5.0/bin/start-hubble.sh'
```

