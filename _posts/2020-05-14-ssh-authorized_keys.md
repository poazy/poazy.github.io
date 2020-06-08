---
layout: post
title:  "通过 ssh 免密码登陆 Linux 机器"
date:   2020-05-14 23:00:00
categories: Linux SSH
tags: Linux SSH
author: poazy
---

* content
{:toc}

> 通过 ssh 免输密码登陆远程 Linux 机器
>
> 如 A 机器登陆 B 机器，那就把 A 机器的公钥信息保存到 B 机器 `~/.ssh/authorized_keys` 文件中，即可




# 生成 ssh 公私钥

> 在本机执行以下 `ssh-keygen -t rsa` 生成公私钥

```bash
# 执行 ssh-keygen -t rsa 可以不设置密码， 一路回车，就可以在 .ssh 文件夹下生成公钥和私钥对
# windows 在 C:\Users\[登陆用户]\.ssh 下，linux 在 ~/.ssh 下
ssh-keygen -t rsa
```

```bash
duxxxo@DESKTOP-OFUA77Q MINGW64 ~/.ssh
$ ll
total 6
-rw-r--r-- 1 duxxxo 197121 2602 12月 25 14:55 id_rsa
-rw-r--r-- 1 duxxxo 197121  566 12月 25 14:55 id_rsa.pub
-rw-r--r-- 1 duxxxo 197121  543  6月  8 16:13 known_hosts
```

> 执行 `cat ~/.ssh/id_rsq.pub` 查看公钥

```bash
cat ~/.ssh/id_rsa.pub
```



# 公钥放到远程机

> 将本机公钥 `id_rsa.pub` 信息放到要访问的远程机器 `~/.ssh/authorized_keys` 文件中
>
> 公钥中的内容全部粘贴添加至  authorized_keys ，保存即可
>
> 如果已存在 authorized_keys 文件信息换行追加即可

```bash
vi ~/.ssh/authorized_keys
```

```bash
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCxshUQdmhHeO0NepfZS/NOyCgfRcu8DhjFiQiC1+ZCSoxHDB0S2L283rLtslLIidSb7FZuJ5D1Qvoq5DfxPjQsSoZDQCQ6qOuYfuqbb0YIOSlaccWLIhcjn7vd6X06LLcFyIHdmmZfMUltQfqdimLv0cnpnGHYVf6Gx/FAY3Fa3wtgnWNm******************ldznShCPZThIwIz8LLbYlblbGUHJIT5q6lgWHiWDWV8gfsYB6AqL7YgvLCVbRyr2vThglz+m4AT9wAris2a0Dca1QZJoBTruz/G+zOHMS7WoKpGSoKE3j6C7DARGTqBuw7QKD9bbB4VPpn0vbEXx5Ra1P9UFaKtsdaoZCYbcyLXjQhcHws9N7UYuQWLFYxCSEiMbZKESrzC2waKKU34yEZ1FoTWE3YrPGEYsEBoWWHfh6OlTrfQFyzRcZHRHGo79Kp38XB6vjtBp8OGhMpnPV44utcIKBbVbQlpnEadUzORKwhwVHo3VihB2k+IPtYM= bxxxy@qq.com
```

```bash
[root@localhost .ssh]# pwd && ll && cat authorized_keys
/root/.ssh
总用量 12
-rw-r--r--. 1 root root  567 6月   8 16:38 authorized_keys
-rw-------. 1 root root 1679 4月  26 11:07 id_rsa
-rw-r--r--. 1 root root  401 4月  26 11:07 id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCxshUQdmhHeO0NepfZS/NOyCgfRcu8DhjFiQiC1+ZCSoxHDB0S2L283rLtslLIidSb7FZuJ5D1Qvoq5DfxPjQsSoZDQCQ6qOuYfuqbb0YIOSlaccWLIhcjn7vd6X06LLcFyIHdmmZfMUltQfqdimLv0cnpnGHYVf6Gx/FAY3Fa3wtgnWNm******************ldznShCPZThIwIz8LLbYlblbGUHJIT5q6lgWHiWDWV8gfsYB6AqL7YgvLCVbRyr2vThglz+m4AT9wAris2a0Dca1QZJoBTruz/G+zOHMS7WoKpGSoKE3j6C7DARGTqBuw7QKD9bbB4VPpn0vbEXx5Ra1P9UFaKtsdaoZCYbcyLXjQhcHws9N7UYuQWLFYxCSEiMbZKESrzC2waKKU34yEZ1FoTWE3YrPGEYsEBoWWHfh6OlTrfQFyzRcZHRHGo79Kp38XB6vjtBp8OGhMpnPV44utcIKBbVbQlpnEadUzORKwhwVHo3VihB2k+IPtYM= bxxxy@qq.com

[root@localhost .ssh]#
```



# 免密码登陆

> 设置好公钥到 `authorized_keys` 文件后就可以 ssh 密码登陆了
>
> 比如我们设置的是 root 用户的 `authorized_keys` 文件，那免密码登陆密码如下

```bash
ssh root@192.168.9.241
```

```bash
duxxxo@DESKTOP-OFUA77Q MINGW64 ~/.ssh
$ ssh root@192.168.9.241
Last login: Mon Jun  8 16:38:33 2020 from 192.168.9.251
[root@localhost ~]#
```





