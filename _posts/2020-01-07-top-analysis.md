---
layout: post
title:  "top 命令信息解释"
date:   2020-01-07 00:00:00
categories: Linux
tags: Linux
author: poazy
---

* content
{:toc}



# top 命令
```bash
top
```
```
top - 14:33:19 up 4 days,  5:03,  1 user,  load average: 0.01, 0.10, 0.17
Tasks: 146 total,   1 running, 144 sleeping,   0 stopped,   1 zombie
%Cpu(s):  7.9 us, 11.0 sy,  0.0 ni, 81.0 id,  0.0 wa,  0.0 hi,  0.2 si,  0.0 st
KiB Mem :  5944864 total,  2422572 free,  2445336 used,  1076956 buff/cache
KiB Swap:  2097148 total,  1386784 free,   710364 used.  3135480 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 2724 root      20   0 1221272  24980   2752 S   1.3  0.4  14:43.68 containerd
 2728 root      20   0  743372  56048   5548 S   1.3  0.9  21:16.52 dockerd
  654 root      20   0  109092   6616   2408 S   0.7  0.1   4:35.33 containerd-shim
 3055 polkitd   20   0 1885108 363416   5880 S   0.7  6.1   9:33.41 mysqld
 4044 root      20   0  109092   9432   2448 S   0.7  0.2   4:34.29 containerd-shim
 4116 root      20   0  109092   9692   2344 S   0.7  0.2   4:41.06 containerd-shim
32678 root      20   0  109092   9004   2356 S   0.7  0.2   4:45.97 containerd-shim
   14 root      20   0       0      0      0 S   0.3  0.0   0:52.17 ksoftirqd/1
 2442 root      20   0  109092   6596   2076 S   0.3  0.1   4:43.15 containerd-shim
 3980 200       20   0 4342488 334796   5904 S   0.3  5.6  30:51.99 java
 4086 vagrant   20   0 4096480 720024   6392 S   0.3 12.1   6:44.82 java
28508 root      20   0   13124   1692   1328 S   0.3  0.0   0:01.46 bash
    1 root      20   0  128280   5264   3176 S   0.0  0.1   2:42.23 systemd
    2 root      20   0       0      0      0 S   0.0  0.0   0:00.03 kthreadd
    3 root      20   0       0      0      0 S   0.0  0.0   0:55.85 ksoftirqd/0
```

# top 信息解释
## 第一行信息

![](/images/20200107-top/top-1.png)

## 第二行信息

![](/images/20200107-top/top-2.png)

## 第三行信息

![](/images/20200107-top/top-3.png)

## 第四行信息

![](/images/20200107-top/top-4.png)

## 每五行信息

![](/images/20200107-top/top-5.png)

## 进程信息

![](/images/20200107-top/top-6.png)

