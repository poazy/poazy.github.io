---
layout: post
title:  "Dockerfile 及打包镜像命令"
date:   2022-01-28 10:20:00
categories: Docker
tags: Dockerfile Docker
author: poazy
---


* content
{:toc}
> Spring Boot 应用和 Vue 应用的 Dockerfile 及打包镜像命令。



# Spring Boot 应用
## Dockerfile
```bash
ARG FROM_IMAGE="openjdk:8-jre-alpine"
FROM $FROM_IMAGE

RUN rm -rf /etc/localtime
RUN ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

ARG APP_NAME="default-app"
ENV APP_NAME=$APP_NAME
ENV JAVA_OPTS=""

RUN mkdir -p /boazy/app/${APP_NAME}
RUN mkdir -p /boazy/logs/${APP_NAME}

ADD target/*.jar /boazy/app/${APP_NAME}/${APP_NAME}.jar

WORKDIR /boazy/app/${APP_NAME}

VOLUME /boazy/app/${APP_NAME}
VOLUME /boazy/logs/${APP_NAME}

RUN export CURR_TIME=$(date +%Y%m%d%H%M%S)
RUN touch $APP_NAME-v$CURR_TIME
ENV APP_VERSION=$CURR_TIME

ENTRYPOINT java -Djava.security.egd=file:/dev/./urandom ${JAVA_OPTS} -jar /boazy/app/${APP_NAME}/${APP_NAME}.jar

```
## docker build
```bash
docker build -t permission-server:v0.0.1 \
--build-arg APP_NAME="permission-server" \
--build-arg FROM_IMAGE="openjdk:8-jre-alpine" \
--no-cache --platform linux/amd64 .
```

# Vue 应用
## Dockerfile
```bash
ARG FROM_IMAGE="nginx:alpine"
FROM $FROM_IMAGE

RUN rm -rf /etc/localtime
RUN ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

ARG APP_NAME="default-app"
ENV APP_NAME=$APP_NAME
ENV NGX_NGINX_DIR="/etc/nginx"
ENV NGX_HTML_DIR="/usr/share/nginx/html"
ENV NGX_CONF_DIR="/etc/nginx/conf.d"
ENV NGX_LOG_DIR="/var/log/nginx"

COPY dist/ $NGX_HTML_DIR/

RUN export CURR_TIME=$(date +%Y%m%d%H%M%S)
RUN touch $APP_NAME-v$CURR_TIME
ENV APP_VERSION=$CURR_TIME

VOLUME $NGX_HTML_DIR
VOLUME $NGX_CONF_DIR
VOLUME $NGX_LOG_DIR

ENV LISTEN_PORT="8080"
ENV CLIENT_MAX_BODY_SIZE="20m"

# 支持创建镜像实例时指定端口和请求体最大值设置（基于 nginx:alpine）
RUN sed -i '$i sed -i "s/listen       80;/listen       $LISTEN_PORT;/g" $NGX_CONF_DIR/default.conf' /docker-entrypoint.sh
RUN sed -i '$i sed -i "s/listen  \\[::\\]:80;/listen  \\[::\\]:$LISTEN_PORT;/g" $NGX_CONF_DIR/default.conf' /docker-entrypoint.sh
RUN sed -i '$i sed -i "5 i \\    client_max_body_size $CLIENT_MAX_BODY_SIZE;" $NGX_CONF_DIR/default.conf' /docker-entrypoint.sh
RUN sed -i '$i echo "LISTEN_PORT=$LISTEN_PORT"' /docker-entrypoint.sh
RUN sed -i '$i echo "CLIENT_MAX_BODY_SIZE=$CLIENT_MAX_BODY_SIZE"' /docker-entrypoint.sh
RUN sed -i '$i \ ' /docker-entrypoint.sh

```
## docker build
```bash
docker build -t permission-front:v0.0.1 \
--build-arg APP_NAME="permission-front" \
--build-arg FROM_IMAGE="nginx:alpine" \
--no-cache --platform linux/amd64 .
```
