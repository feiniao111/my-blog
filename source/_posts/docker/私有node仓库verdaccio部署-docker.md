---
title: 私有node仓库verdaccio部署(docker)
date: 2019-02-04 19:41:05
tags:
- docker
---

## why verdaccio?
[verdaccio](https://github.com/verdaccio/verdaccio)基于sinopia fork出来的node仓库，具备向后兼容特性，因为[sinopia](https://github.com/rlidwka/sinopia)已经不再维护,故选择verdaccio作为私有node仓库。

## why docker?
原有的方案采用[pm2](https://github.com/verdaccio/verdaccio)作为node仓库的进程管理器，它是一款优秀的node进程管理器，具有负载均衡等特点。但我们使用pm2的主要原因是为了避免仓库进程突然宕掉。从这点来看，运行docker容器就可以了，只要加上--restart=always，就能实现服务挂掉自重启。

## 镜像
采用官方镜像verdaccio/verdaccio。镜像下载命令：
```
docker pull verdaccio/verdaccio
```

## 数据持久化
在磁盘上建立目录/home/docker/appData/verdaccio，作为仓库的storage，注意由于verdaccio以非root用户运行，需要给该目录赋予写和读权限：
```
chmod -R 777 /home/docker/appData/verdaccio
```

## 运行
执行以下脚本，启动一个名叫verdaccio，宕机自重启的docker容器
```
#!/bin/bash
set -x
V_PATH=/home/docker/appData/verdaccio
docker run -it --name verdaccio -p 4873:4873 \
        -v $V_PATH/storage:/verdaccio/storage \
        --restart=always \
        verdaccio/verdaccio

```
生成容器:
```
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                                                                                NAMES
d08ca15fb244        verdaccio/verdaccio   "/usr/local/bin/dumb…"   2 hours ago         Up 2 hours          0.0.0.0:4873->4873/tcp                                                               verdaccio

```

## 使用
和sinopia完全一样

## 日志查看
集成到容器日志中，查看命令为
```
docker inspect verdaccio
```