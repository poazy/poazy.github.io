---
layout: post
title:  "Windows 10 安装 Node.js"
date:   2020-06-23 21:30:00
categories: Node.js
tags: Node.js npm cnpm smart-npm
author: poazy
---

* content
{:toc}
> `Windows 10` 安装 `Node.js` 及 环境配置。




# 下载 Node.js

* 下载地址：https://nodejs.org/zh-cn/

* 选择长期支持版本（LTS）下载

  > 这里下载下来的文件是：node-v12.18.1-x64.msi



# 安装 Node.js

* 安装 `node-v12.18.1-x64.msi`

  > [可选]安装时，安装目录设置为 D:\Local\nodejs

* 安装完后执行命令查看版本验证

  ```bash
  node -v
  npm -v
  ```

  > 执行情况：
  >
  > ```bash
  > Microsoft Windows [版本 10.0.18363.900]
  > (c) 2019 Microsoft Corporation。保留所有权利。
  > D:\用户名\Desktop>node -v
  > v12.18.1
  > 
  > D:\用户名\Desktop>npm -v
  > 6.14.5
  > 
  > D:\用户名\Desktop> 
  > ```



# 环境配置（可选）

* 设置全局目录和缓存目录

  > prefix 默认位置 `C:\Users\用户名\AppData\Roaming\npm`
  >
  > cache 默认位置  `C:\Users\用户名\AppData\Roaming\npmxxx`

  ```bash
  # 执行命令配置
  npm config set prefix "D:\Local\nodejs\node_global"
  npm config set cache "D:\Local\nodejs\node_cache"
  ```

  >C:\Users\用户名\\.npmrc
  >
  >prefix=D:\Local\nodejs\node_global
  >cache=D:\Local\nodejs\node_cache

* 设置环境变量 `NODE_PATH`

  > NODE_PATH=D:\Local\nodejs\node_global\node_modules

  变量名：NODE_PATH

  变量值：D:\Local\nodejs\node_global\node_modules

* 修改环境变量 `Path`

  将 `Path` 中值为 `C:\Users\用户名\AppData\Roaming\npm` 改为 `D:\Local\nodejs\node_global`



# npm 安装程序

## 安装 cnpm

> 淘宝团队做的国内镜像，因为npm的服务器位于国外可能会影响安装。

```bash
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

## 安装 smart-npm

> 它封装了 npm 和 cnpm 在里面，根据你使用的命令自动切换 npm 或者 cnpm。

```bash
npm install -g smart-npm --registry=https://registry.npm.taobao.org
```

## 安装 express

> Express 是一个简洁而灵活的 node.js Web应用框架, 提供了一系列强大特性帮助你创建各种 Web 应用，和丰富的 HTTP 工具。
>
> 使用 Express 可以快速地搭建一个完整功能的网站。

```bash
npm install -g express --registry=https://registry.npm.taobao.org
```







