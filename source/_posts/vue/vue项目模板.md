---
title: vue项目模板
date: 2019-02-04 17:29:26
tags:
- vue
---
背景
目前vue-cli官方提供的webpack模板，和实际组内项目有所区别，每次新建一个项目，都需要对用官方模板创建的项目做一些细节调整，具体包括:
  1.  输出目录改为webapp(默认为dist);
  2.  隐藏生产环境源码（默认可见);
  3.  安装项目依赖（jquery、node-sass等）
等等，为了加快项目开发效率，有必要对现有项目进行梳理，提取出一套适用于CM web项目的webpack模板。

定制模板
vue-cli提供了定制模板的方法，允许开发团队定制自己的个性模板。这里大致介绍一下我们的定制模板的功能以及目录结构。
功能
  ● 国际化
  ● 服务器代理
  ● 提供mock环境
  ● 预置jquery, node-sass, vue-router, vuex, mockjs等常用插件
  ● 生产环境代码隐藏
  ● 编译产出目录为webapp

<a href="http://note.youdao.com/noteshare?id=366f5d0692c89c7a0d5f44683719bded&sub=F04160B34B874315AB9435EA15459460" target="_blank">项目模板</a>