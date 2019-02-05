---
title: Phabricator部署(docker)
date: 2019-02-04 17:01:18
tags:
- docker
- code-review
---
## 特点
Phabricator的部署改用了docker容器的方式，相比传统安装方式，具有以下特点：
- 能够更快速的安装，并省去配置环节。传统方式需要安装nginx、php-fpm、mysql、python，并对它们进行配置，比较繁琐。采用docker部署后，只需要从docker-hub上下载两个镜像（mysql，phabricator），启用容器，即完成了部署，无需额外配置。
- 轻松移植到其他服务器。由于镜像都是开源的，并托管到docker-hub上，因此可以方便地在不同服务器上移植。

## 准备
### docker
推荐采用新的稳定版本，本文采用的是17.12.1-ce。
```
[root@localhost ~]# docker version
Client:
 Version:	17.12.1-ce
 API version:	1.35
 Go version:	go1.9.4
 Git commit:	7390fc6
 Built:	Tue Feb 27 22:15:20 2018
 OS/Arch:	linux/amd64

Server:
 Engine:
  Version:	17.12.1-ce
  API version:	1.35 (minimum version 1.12)
  Go version:	go1.9.4
  Git commit:	7390fc6
  Built:	Tue Feb 27 22:17:54 2018
  OS/Arch:	linux/amd64
  Experimental:	false

```
### [mysql镜像](https://hub.docker.com/_/mysql/)
这里采用的是经典的5.6.35版本，下载命令：
```
docker pull mysql:5.6.35
```
注意这个mysql还需要对配置文件进行调整，才能完美适配phabricator，具体见下文安装小节。

### [phabricator镜像](https://hub.docker.com/r/hachque/phabricator/)
已经有大牛将phabricator镜像上传到了docker-hub，直接使用就好，该镜像集成了所需环境（pha源码，nginx，php-fpm，python），虽然有1.21GB，不过对我们来说是完全可接受的。
```
docker pull hachque/phabricator
```
下载完以上两个镜像，执行`docker images`，可以看到
```
REPOSITORY                 TAG                 IMAGE ID            CREATED             SIZE
hachque/phabricator        latest              29e3817de0d2        5 weeks ago         1.21GB
mysql                      5.6.35              a0f5d7301767        12 months ago       329MB

```

### 数据持久化
由于容器无法持久化数据，删除容器就会丢失数据，因此需要在宿主机上准备目录用于存储mysql和phabricator的数据。
- mysql  
1. 建立/home/docker/appData/pha_mysql_data/目录，用来存放mysql的数据库文件；
2. 建立/home/docker/appData/pha_mysql_conf.d/目录，用来存放mysql的pha配置文件。将mysql的配置文件拷贝到该目录下，这个配置文件是根据pha官网配置的，具体见附录。

- phabricator  
建立/home/docker/appData/pha_repos目录，用来作为pha的repo；

## 安装
执行脚本
```
#!/bin/bash
set -x
MYSQL_DATA_DIR=/home/docker/appData/pha_mysql_data/
MYSQL_PORT=3308
MYSQL_CONF_DIR=/home/docker/appData/pha_mysql_conf.d/
MYSQL_PASSWORD=a123456A~
MYSQL_HOST=172.21.149.13
PHA_HOST=pha.cmrd.com
PHA_REPOS=/home/docker/appData/pha_repos

#启动mysql容器
docker run -d --name pha_mysql \
        -p ${MYSQL_PORT}:3306 \
        -v ${MYSQL_DATA_DIR}:/var/lib/mysql \
        -v ${MYSQL_CONF_DIR}:/etc/mysql/conf.d  \
        -e MYSQL_ROOT_PASSWORD=${MYSQL_PASSWORD} \
        --restart=always \
        mysql:5.6.35

#启动docker容器
docker run \
    -p 80:80 -p 443:443 -p 23:22 \
    --env PHABRICATOR_HOST=${PHA_HOST} \
    --env MYSQL_HOST=${MYSQL_HOST} \
    --env MYSQL_PORT=${MYSQL_PORT} \
    --env MYSQL_USER=root \
    --env MYSQL_PASS=${MYSQL_PASSWORD} \
    --env PHABRICATOR_REPOSITORY_PATH=/repos \
    -v ${PHA_REPOS}:/repos \
    --restart=always \
    hachque/phabricator
```

执行完该脚本，会产生两个容器，皆处于运行状态
```
[root@localhost pha_mysql_data]# docker ps
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                                                                  NAMES
4f7366e65be0        hachque/phabricator   "/bin/bash /app/init…"   About an hour ago   Up About an hour    0.0.0.0:80->80/tcp, 24/tcp, 0.0.0.0:443->443/tcp, 0.0.0.0:23->22/tcp   focused_jepsen
a84101824751        mysql:5.6.35          "docker-entrypoint.s…"   About an hour ago   Up About an hour    0.0.0.0:3308->3306/tcp                                                 pha_mysql

```
- pha_mysql是mysql容器，提供mysql功能，对宿主机曝露端口为3308，用户名root，密码a123456A~；
- focused_jepsen是phabricator容器，使用了pha_mysql容器来作为数据库，使用了宿主机的3个端口：80（http），443（https），23（ssh）。首页为
```
http://pha.cmrd.com
```
==注意==游览器所在的电脑要配置host，
```
172.21.149.13  pha.cmrd.com
```
访问[http://pha.cmrd.com]()或者[http://pha.cmrd.com:80]()，正常会跳至管理员账号设置页，说明phabricator已经部署完成。

## 附录
### mysql配置文件
mysql的配置文件pha_mysql.cnf, 存放在宿主机的/home/docker/appData/pha_mysql_conf.d/：
```
#针对phabricator的mysql配置
[mysqld]
character_set_server=utf8
init_connect='SET NAMES utf8'
max_allowed_packet=33554432
sql_mode=STRICT_ALL_TABLES
innodb_buffer_pool_size=1600M
```
注意pha_mysql.cnf的权限必须是644，否则mysql可能拒绝采用。
