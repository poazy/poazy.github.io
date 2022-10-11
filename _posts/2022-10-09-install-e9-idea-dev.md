---
layout: post
title:  "E9（Ecology9）本地开发环境搭建"
date:   2022-10-09 20:00:00
categories: Ecology
tags: Ecology Resin IDEA
author: poazy
---


* content
{:toc}
> 采用 IDEA + Resin 插件搭建 E9（Ecology9）本地开发环境。







------

## 搭前准备

### 备份 classbean

> 备分好 classbean 文件夹
>
> 以后开发的代码也是发布 class 到 classbean 文件夹下

![](../images/20221009-install-ecology9-idea-dev/00bak-classbean-01.png)

### 代码放到指定目录

> 开发的代码文件夹 src

![](../images/20221009-install-ecology9-idea-dev/00copy-src-01.png)





------

## IDEA 导入项目

### 打开项目

> Open Project
>
> 打开 weaver 层目录

![](../images/20221009-install-ecology9-idea-dev/01open-project-01.png)



### 设置项目

#### 设置 JDK

![](../images/20221009-install-ecology9-idea-dev/02set-sdk-01.png)

![](../images/20221009-install-ecology9-idea-dev/02set-sdk-02.png)

#### 设置 Facets

![](../images/20221009-install-ecology9-idea-dev/03set-project-facets-01.png)

![](../images/20221009-install-ecology9-idea-dev/03set-project-facets-02.png)

![](../images/20221009-install-ecology9-idea-dev/03set-project-facets-03.png)

![](../images/20221009-install-ecology9-idea-dev/03set-project-facets-04.png)

#### 设置 Libraries

![](../images/20221009-install-ecology9-idea-dev/04set-project-libraries-01.png)

![](../images/20221009-install-ecology9-idea-dev/04set-project-libraries-02.png)

![](../images/20221009-install-ecology9-idea-dev/04set-project-libraries-03.png)

#### 设置 Modules

![](../images/20221009-install-ecology9-idea-dev/05set-project-modules-01.png)

![](../images/20221009-install-ecology9-idea-dev/05set-project-modules-02.png)

![](../images/20221009-install-ecology9-idea-dev/05set-project-modules-03.png)

![](../images/20221009-install-ecology9-idea-dev/05set-project-modules-04.png)

#### 编译检查

![](../images/20221009-install-ecology9-idea-dev/06cc-01.png)

![](../images/20221009-install-ecology9-idea-dev/06cc-02.png)





------

## IDEA Resin 运行

### 安装 Resin 插件

![](../images/20221009-install-ecology9-idea-dev/07resin-plugin-install-01.png)

![](../images/20221009-install-ecology9-idea-dev/07resin-plugin-install-02.png)

### 设置 Resin

![](../images/20221009-install-ecology9-idea-dev/08resin-plugin-cfg-01.png)

![](../images/20221009-install-ecology9-idea-dev/08resin-plugin-cfg-02.png)

>VM options:  -DLC_ALL=zh_CN.gbk -DLANG=zh_CN.gbk -Djava.net.preferIPv4Stack=true

![](../images/20221009-install-ecology9-idea-dev/08resin-plugin-cfg-03.png)

### 启动 Resin

![](../images/20221009-install-ecology9-idea-dev/08resin-plugin-start-04.png)

### 访问 Resin

![](../images/20221009-install-ecology9-idea-dev/09visit-page-01.png)





------

## 附录

### 可能访问页面为空白的问题

#### 问题描述

> 是由于环境参数中的路径有包括空格字符的路径，导致 jsp 编译时就出现了异常：500 (Internal Server Error)

![](../images/20221009-install-ecology9-idea-dev/99error-01.png)

```bash
[2022.10.09 14:24:41.903]>>>>Xss(Exception):sw=com.caucho.jsp.JspParseException: javac: 无效的标记: IDEA.app/Contents/lib/idea_rt.jar:/Users/duanbo/Library/Caches/JetBrains/IntelliJIdea2022.1/captureAgent/debugger-agent.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_202.jdk/Contents/Home/jre/lib/resources.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_202.jdk/Contents/Home/jre/lib/rt.jar...

[2022.10.09 14:24:41.906]用法: javac <options> <source files>
-help 用于列出可能的选项
at com.caucho.jsp.JspCompilerInstance.compile(JspCompilerInstance.java:448)
at com.caucho.jsp.JspManager.compile(JspManager.java:286)
at com.caucho.jsp.JspManager.createPage(JspManager.java:192)
at com.caucho.jsp.JspManager.createPage(JspManager.java:173)...

[2022.10.09 14:24:41.907]	at com.caucho.env.thread2.ResinThread2.run(ResinThread2.java:118)
Caused by: com.caucho.java.JavaCompileException: javac: 无效的标识: IDEA.app/Contents/lib/idea_rt.jar:/Users/duanbo/Library/Caches/JetBrains/IntelliJIdea2022.1/captureAgent/debugger-agent.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_202.jdk/Contents/Home/jre/lib/resources.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_202.jdk/Contents/Home/jre/lib/rt.jar...
                         
[2022.10.09 14:24:41.907]用法: javac <options> <source files>
-help 用于列出可能的选项
at com.caucho.java.ExternalCompiler.compileInt(ExternalCompiler.java:263)
at com.caucho.java.AbstractJavaCompiler.run(AbstractJavaCompiler.java:112)
... 2 more
```

![](../images/20221009-install-ecology9-idea-dev/99error-02.png)

![](../images/20221009-install-ecology9-idea-dev/99error-bug-03.png)

#### 修复方案

> 将 `IntelliJ IDEA.app` 重命名为 `IntelliJIDEA.app` 去掉中间的空格，后再重新打开 IDEA 后，再启动 Resin 。

![](../images/20221009-install-ecology9-idea-dev/99error-fix-04.png)



