---
title: vue项目模板demo部署(docker)
date: 2019-02-04 17:19:49
tags:
- docker
---
## 优点
将使用模板生成的项目打包成docker镜像，使其可以快速的安装和运行。另外在生成docker容器时添加
```
--restart=always
```
选项，即使服务挂掉也会自动重启，省去了运维工作(#\^.^#)

## Dockerfile
基于[node官方镜像](https://hub.docker.com/_/node/)制作。

- Dockerfile
```
FROM node:8
WORKDIR /app
#安装模板脚本
COPY install.sh /app
# 配置镜像源
RUN mv /etc/apt/sources.list /etc/apt/sources.list.bak && \
    echo "deb http://mirrors.163.com/debian/ jessie main non-free contrib" >/etc/apt/sources.list && \
    echo "deb http://mirrors.163.com/debian/ jessie-proposed-updates main non-free contrib" >>/etc/apt/sources.list && \
    echo "deb-src http://mirrors.163.com/debian/ jessie main non-free contrib" >>/etc/apt/sources.list && \
    echo "deb-src http://mirrors.163.com/debian/ jessie-proposed-updates main non-free contrib" >>/etc/apt/sources.list
# 安装vue-cli
RUN apt-get update && apt-get -y install expect && npm install -g vue-cli && apt-get -y install net-tools
# 安装模板
RUN chmod +x install.sh && ./install.sh

WORKDIR /app/frontend
#启动模板脚本
COPY inrun.sh /app/frontend

EXPOSE 8080 8081
#CMD [ "npm", "run", "dev" ]
CMD [ "sh", "inrun.sh" ]
```

- install.sh  
下载项目模板并生成项目。Dockerfile使用。
```
#!/usr/bin/expect
#等待从github上下载模板源码
set timeout 200
spawn vue init feiniao111/vueWebTemplate frontend
expect (frontend)
send "\r"
set timeout 10
expect (A*Vue.js*project)
send "\r"
expect fe)
send "\r"
expect >)
send "\r"
expect elsewhere\r
send "\r"
#等待安装node依赖
set timeout 5000
expect myself
send "\r"

expect https://vuejs-templates.github.io/webpack
send \003

```
- inrun.sh  
运行dev环境和mock环境。启动容器时执行。
```
#!/bin/bash
#更改主机ip
ip=`ifconfig -a|grep inet|grep -v 127.0.0.1|grep -v inet6|awk '{print $2}'|tr -d "addr:"`
sed  -i "s/host: 'localhost'/host: '$ip'/" config/index.js
npm run dev&
npm run mock
```

## 制作镜像
在Dockerfile所在目录，执行
```
docker build -t cm_vue_template:1.0 .
```
制作成功后，执行`docker images`查看该镜像
```
REPOSITORY                 TAG                 IMAGE ID            CREATED             SIZE
cm_vue_template            1.0                 0cae166e736c        About an hour ago   977MB

node                       8                   41a1f5b81103        10 days ago         673MB
```
可以看到，cm_vue_template:1.0有977MB，这是因为它的基础镜像node:8就有673MB（内置了svn、git、node等），并且cm_vue_template还安装好了运行vue项目所需的node依赖。注意：因为要从github上下载模板，由于网络的原因可能会制作失败，多试几次就好。

## 运行容器
镜像生成后，执行
```
docker run -d --name template -p 8080-8081:8080-8081 --restart=always cm_vue_template:1.0
```
注意这里暴露的8080是dev环境的端口，8081是mock环境，`--restart-always`可以在容器停止后重新启动。  
容器运行成功后，执行`docker ps`,可以看到正在运行的template容器
```
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                                                                  NAMES
018a83d10925        cm_vue_template:1.0   "sh inrun.sh"            2 hours ago         Up 2 hours          0.0.0.0:8080-8081->8080-8081/tcp                                       template

```

## 访问
容器运行成功后,访问所在机器的ip+端口号，如：  
http://172.21.149.13:8080  
http://172.21.149.13:8081