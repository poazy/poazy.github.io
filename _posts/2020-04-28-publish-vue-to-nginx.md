---
layout: post
title:  "Vue 前端部署到 Nginx（docker 版本）"
date:   2020-04-28 22:30:00
categories: Vue
tags: Nginx Docker Vue
author: poazy
---

* content
{:toc}

> 将 Vue 前端应用部署到 Docker Nginx 容器中。
> 
> 这没有采用打包成 Docker 镜像，也没有脚本自动创建运行 Docker 容器。




# 创建 nginx 容器

## 创建容器挂载目录文件

```bash
mkdir -p /boazy/data/dockerdata/front-nginx/{html,conf,conf.d,log}
```

> 创建 nginx.conf 文件

```bash
# https://github.com/nginx/nginx/blob/branches/stable-1.18/conf/nginx.conf 不包含 server 修改
vi /boazy/data/dockerdata/front-nginx/nginx.conf
```

> nginx.config 文件内容如下：

```bash

user  nginx;
worker_processes  2;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```

## 运行 nginx 容器

> 创建并运行 front-nginx 的 nginx 容器

```bash
docker run -d --restart=always --name front-nginx \
--net=host \
-v /boazy/data/dockerdata/front-nginx/html:/usr/share/nginx/html \
-v /boazy/data/dockerdata/front-nginx/conf:/etc/nginx/conf \
-v /boazy/data/dockerdata/front-nginx/conf.d:/etc/nginx/conf.d \
-v /boazy/data/dockerdata/front-nginx/log:/var/log/nginx \
-v /boazy/data/dockerdata/front-nginx/nginx.conf:/etc/nginx/nginx.conf \
-e "TZ=Asia/Shanghai" \
nginx:1.18.0
```



# 发布 Vue 前端应用

## 发布 config.d 配置

> 创建 sso.conf 文件（自己要发布的应用配置文件）

```bash
# https://github.com/nginx/nginx/blob/branches/stable-1.18/conf/nginx.conf 只取 server 修改
vi /boazy/data/dockerdata/front-nginx/conf.d/front-sso.conf
```

> sso.confg 文件内容如下：

```bash
server {
    listen       8010;
    server_name  localhost;
    location / {
        root   /usr/share/nginx/html/front-sso;
        index  index.html index.htm;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

## 发布 Vue 前端代码

> 在 html 目录下登陆 front-sso 目录，用于发布前端应用代码

```bash
mkdir -p /boazy/data/dockerdata/front-nginx/html/front-sso
```

> 将 Vue 前端代码上传到 front-sso 目录下即可

```bash
# /boazy/data/dockerdata/front-nginx/html/front-sso 为前端应用主目录
```

## 重新加载 nginx 

> 重新加载 nginx 使新修改的配置生成，就可以访问了

```bash
# 命令中 front-nginx 为容器名
docker exec front-nginx nginx -s reload
```



# 附录：jenkins 脚本

```groovy
def timeVersion() {
	// 20191210175842
    return new Date().format('yyyyMMddHHmmss')
}

def getRemoteServer(args) {
    def remote = [:]
    remote.name = 'SERVER_' + args.host
    remote.host = args.host
    remote.port = args.port
    remote.allowAnyHosts = true
    withCredentials([usernamePassword(credentialsId: args.credentialsId, passwordVariable: 'password', usernameVariable: 'username')]) {
        remote.user = "${username}"
        remote.password = "${password}"
    }
    remote.gateway = args.gateway

    return remote
}

def deployFront(remoteServer) {
    sh """
        cd dist
        tar czvf ${_timeVersion}_${appName}.tar.gz *
        pwd && ls -l
    """
    sshCommand remote: remoteServer, command: "mkdir -p /boazy/data/dockerdata/${deployNginxName}/html/${appName}"
    sshPut remote: remoteServer, from: "dist/${_timeVersion}_${appName}.tar.gz", into: "/boazy/data/dockerdata/${deployNginxName}/html/${appName}"
    sshCommand remote: remoteServer, command: """
        cd /boazy/data/dockerdata/${deployNginxName}/html/${appName}
        tar -zxvf ${_timeVersion}_${appName}.tar.gz -C ./
    """
    sshCommand remote: remoteServer, command: """
        cd /boazy/data/dockerdata/${deployNginxName}/html/${appName}
        mkdir -p bak && mv ${_timeVersion}_${appName}.tar.gz bak/${_timeVersion}_${appName}.tar.gz
        pwd && ls -l
        ls -l bak
    """
    sh """
        cd dist
        rm -rf *_${appName}.tar.gz
        pwd && ls -l
    """
    sshCommand remote: remoteServer, command: """
        cd /boazy/data/dockerdata/${deployNginxName}/conf.d
        mkdir -p bak && mv ${appName}.conf bak/${_timeVersion}_${appName}.conf
        pwd && ls -l
        ls -l bak
    """, failOnError: false
    script {
        def configFileContent = """server {
    listen       ${deployPort};
    server_name  localhost;
    location / {
        root   /usr/share/nginx/html/${appName};
        index  index.html index.htm;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}"""
        writeFile file: "${appName}.conf", text: configFileContent, encoding: "UTF-8"
    }
    sshPut remote: remoteServer, from: "${appName}.conf", into: "/boazy/data/dockerdata/${deployNginxName}/conf.d"
    sh "rm -rf ${appName}.conf"
    sshCommand remote: remoteServer, command: "docker exec ${deployNginxName} nginx -s reload"
}

pipeline {
    agent any

    environment {
        _timeVersion = timeVersion()
    }

    stages {
        stage('Checkout') {
            steps{
                echo 'Checkout code'
                checkout([$class: 'GitSCM', branches: [[name: "${gitBranch}"]]
                        , doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: []
                        , userRemoteConfigs: [[credentialsId: '0a6953af58dc', url: "${gitUrl}"]]
                ])
            }
        }
        stage('Config env') {
            steps {
                echo "Config ${active} env"
                sh """
                    cat config/${active}.env.js
                    sed -i 's/http:\\/\\/IP:PORT/http:\\/\\/${kmgatewayHost}/g' config/${active}.env.js
                    sed -i 's/http:\\/\\/IP:PORT/http:\\/\\/${ssoHost}/g' config/${active}.env.js
                    cat config/${active}.env.js
                """
            }
        }
        stage('Build') {
            steps {
                echo "Build Code"
                script {
                    def placeHost = getRemoteServer([host: "192.168.9.95", credentialsId: "de8837a563f8"])
                    def workspace = pwd()
                    workspace = workspace.substring(5)
                    sshCommand remote: placeHost, command: "cd /boazy/data/dockerdata/${workspace}"
                    sshCommand remote: placeHost, command: """
                        cd /boazy/data/dockerdata/${workspace}
                        npm update -g cnpm --registry=https://registry.npm.taobao.org
                        cnpm install
                        npm run ${active}
                    """
                }
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploy Front'
                script {
                    def gatewayHost = getRemoteServer([host: "192.168.9.251", port: 2299, credentialsId: "51cd0da7ac19da"])
                    def remoteServer = getRemoteServer([host: "192.168.9.241", credentialsId: "41cd0da7ac19da", gateway: gatewayHost])
                    deployFront(remoteServer)
                }
            }
        }
    }
}
```





