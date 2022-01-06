---
layout: post
title:  "macOS NFS Server 安装试用"
date:   2021-01-05 23:00:00
categories: NFS Server
tags: NFS macOS
author: poazy
---

* content
{:toc}
> HugeGraph 安装及试用（hugegraph、hugegraph-studio、hugegraph-hubble、hugegraph-tools、hugegraph-loader）




# NFS Server 配置

## 检查 nfsd 的状况

```bash
xxx@xxxdeMacBook-Pro ~ % sudo nfsd status
Password:
nfsd service is enabled
nfsd is not running
xxx@xxxdeMacBook-Pro ~ % 
```

## 设置共享目录

```bash
xxx@xxxdeMacBook-Pro ~ % sudo vi /etc/exports
Password:
xxx@dxxxdeMacBook-Pro ~ % cat /etc/exports 
/Users/xxx/Volumes/nfs-share -alldirs -maproot=root:wheel -network=192.168.0.0 -mask=255.255.0.0
xxx@xxxdeMacBook-Pro ~ % 
```
> /Users/xxx/Volumes/nfs-share 指定共享目录
> 
> -alldirs 共享目录下的所有目录
> -maproot 把 client 端的 root 用户映射为 macOS 上的 root，client 端的 root 组映射为 macOS 上的 wheel (gid=0) 组
> 
> -network -mask 指定本工作网段
> 

## 检查配置状态

```bash
xxx@xxxdeMacBook-Pro ~ % nfsd checkexports
xxx@xxxdeMacBook-Pro ~ % 
```

## 重启服务

```bash
xxx@xxxdeMacBook-Pro ~ % sudo nfsd restart
xxx@xxxdeMacBook-Pro ~ % 
```

## 查看挂载状态

```bash
xxx@xxxdeMacBook-Pro ~ % showmount -e     
Exports list on localhost:
/Users/xxx/Volumes/nfs-share     192.168.0.0
xxx@xxxdeMacBook-Pro ~ %
```

## k8s 挂载设置
> 在 Docker 容器中挂载上述 NFS 文件时一直提示 Permission Denied。经过大量查找资料，发现其实只需添加一项配置到/etc/nfs.conf 中

```bash
xxx@xxxdeMacBook-Pro ~ % cat /etc/nfs.conf 
#
# nfs.conf: the NFS configuration file
#
xxx@xxxdeMacBook-Pro ~ % sudo vi /etc/nfs.conf 
Password:
xxx@xxxdeMacBook-Pro ~ % cat /etc/nfs.conf    
#
# nfs.conf: the NFS configuration file
#
nfs.server.mount.require_resv_port = 0
xxx@xxxdeMacBook-Pro ~ % sudo nfsd update
xxx@xxxdeMacBook-Pro ~ % 
```

# 客户端使用

## 挂载（远程）共享目录
> 这里找一台远程机器（CentOS 7）作为客户端，挂载共享目录

```bash
[root@localhost ~]# showmount -e 192.168.9.13
Export list for 192.168.9.13:
/Users/xxx/Volumes/nfs-share 192.168.0.0
[root@localhost ~]# mount -t nfs 192.168.9.13:/Users/xxx/Volumes/nfs-share /xxx/data/nfs-data
mount.nfs: mount point /xxx/data/nfs-data does not exist
[root@localhost ~]# mkdir -p /xxx/data/nfs-data
[root@localhost ~]# mount -t nfs 192.168.9.13:/Users/xxx/Volumes/nfs-share /xxx/data/nfs-data
[root@localhost ~]# cd /xxx/data/nfs-data
[root@localhost nfs-data]# ls
[root@localhost nfs-data]# vi xxx.txt
[root@localhost nfs-data]# ls -lh
总用量 4.0K
-rw-r--r--. 1 root games 7 1月   5 14:09 xxx.txt
[root@localhost nfs-data]#
```

## 验证客户端创建的文件
> 回到 macOS NFS Server 上查看共享目录有没有客户端创建的文件 xxx.txt 

```bash
xxx@xxxdeMacBook-Pro ~ % cd /Users/xxx/Volumes/nfs-share
xxx@xxxdeMacBook-Pro nfs-share % ls -lh
total 8
-rw-r--r--  1 root  staff     7B  1  5 14:09 xxx.txt
xxx@xxxdeMacBook-Pro nfs-share % 
```

## 客户端卸载 umount

```bash
[root@localhost ~]# umount /xxx/data/nfs-data
[root@localhost ~]# ls -h /xxx/data/nfs-data
[root@localhost ~]#
```