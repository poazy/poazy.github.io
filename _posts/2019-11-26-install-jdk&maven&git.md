---
layout: post
title:  "CentOS 7 安装 JDK、Maven、Git"
date:   2019-11-26 00:00:00
categories: CentOS
tags: CentOS Java Maven Git Install
author: poazy
---

* content
{:toc}

# 下载文件

* JDK
  * jdk-8u202-linux-x64.tar.gz
* Maven
  * apache-maven-3.6.3-bin.tar.gz


# Java 安装

```bash
mkdir /usr/local/java
```

```bash
tar zxvf jdk-8u202-linux-x64.tar.gz -C /usr/local/java
```

```bash
vim /etc/profile
```

```bash
export JAVA_HOME=/usr/local/java/jdk1.8.0_202
export CLASSPATH=.:${JAVA_HOME}/jre/lib:${JAVA_HOME}/lib
export PATH=$PATH:${JAVA_HOME}/bin
```

```bash
source /etc/profile
```

```bash
java -version
```



# Maven 安装

## 安装

```bash
mkdir /usr/local/maven
```

```bash
tar zxvf apache-maven-3.6.3-bin.tar.gz -C /usr/local/maven
```

```bash
vim /etc/profile
```

```bash
export MAVEN_HOME=/usr/local/maven/apache-maven-3.6.3
export PATH=$PATH:$JAVA_HOME/bin:$MAVEN_HOME/bin
```

```bash
source /etc/profile
```

```bash
mvn -version
```

## 加速镜像

```bash
vi /usr/local/maven/apache-maven-3.6.3/conf/settings.xml
```

```bash
<mirror>
    <id>alimaven</id>
    <name>aliyun maven</name>
    <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
    <mirrorOf>central</mirrorOf>        
</mirror>
```



# Git 安装

## 安装

```bash
yum install -y git
```

```bash
git --version
```

## 配置

```bash
git config --global user.name "boazy"
git config --global user.email "boazy@126.com"
ssh-keygen -t rsa -C "boazy@126.com"

# 将公钥（/root/.ssh/id_rsa.pub）上传到 github
```





