---
layout: post
title:  "E9（Ecology9）本地开发环境搭建"
date:   2022-10-09 18:00:00
categories: Ecology
tags: Ecology Ecology9 E9 IDEA Resin
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

![](../images/20221009-install-ecology9-idea-dev/99error-02.png)

![](../images/20221009-install-ecology9-idea-dev/99error-bug-03.png)

#### 修复方案

> 将 `IntelliJ IDEA.app` 重命名为 `IntelliJIDEA.app` 去掉中间的空格，后再重新打开 IDEA 后，再启动 Resin 。

![](../images/20221009-install-ecology9-idea-dev/99error-fix-04.png)
