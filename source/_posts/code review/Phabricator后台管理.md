---
title: Phabricator后台管理
date: 2019-02-04 17:10:42
tags:
- code-review
---

# 登录后台
由于phabricator运行在docker容器中，因此需要进入到容器内部，执行：
```
docker exec -it focused_jepsen /bin/bash
```
focused_jepsen是phabricator容器的名称，可以通过docker ps命令查找到。


# 路径
/srv/phabricator/phabricator

# 管理员账号
- 账号：
- 密码：

# 访问网址
http://pha.cmrd.com/

---
需配置hosts  
- windows系统  
C:\Windows\System32\drivers\etc\HOSTS 文件
- linux系统  
/etc/hosts 文件    
增加一行ip映射  
172.21.149.13  pha.cmrd.com

# 常见问题
## 邮件发不出来，但其他功能正常
检查后台进程是否正常，可以重启试试，重启方法：到pha目录下执行  
bin/phd restart

## 无法启动
检查php、mysql、nginx是否启动正常。
- php
```
/usr/sbin/php-fpm
```
- mysql  
检查mysql容器是否运行正常：执行docker ps能找到mysql容器，并可以通过
```
mysql -uroot -P 3308 -h 172.21.149.136 -p
```
密码a123456A~,连接到mysql。
- nginx
```
/usr/sbin/nginx -s reload
```

## 未知问题
登录后，查看页面顶端左边菜单栏中的Unresolved Setup Issues

# 资料查询
- 官方文档  
https://secure.phabricator.com
- 中文站  
https://phabricator.webfuns.net