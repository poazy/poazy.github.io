---
layout: post
title:  "重置 Jenkins 构建历史"
date:   2020-05-02 23:30:00
categories: Jenkins
tags: Jenkins
author: poazy
---

* content
{:toc}

> 重置 Jenkins 构建历史及序号



# 重置构建历史

> 脚本命令行重置构建历史

```groovy
Jenkins -> 系统管理 -> 工具和动作 -> 脚本命令行
```

> 在脚本命令行输入框输入如下命令运行：

```groovy
// 执行前修改 xxx/yyy 为你实际的 jenkins name
item = Jenkins.instance.getItemByFullName("xxx/yyy")
// THIS WILL REMOVE ALL BUILD HISTORY
item.builds.each() { 
	build -> build.delete()
}
item.updateNextBuildNumber(1)
```





