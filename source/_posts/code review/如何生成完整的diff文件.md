---
title: 如何生成完整的diff文件
date: 2019-02-04 14:38:08
tags:
- code-review
---

我们知道，一般执行`svn diff`命令，只会列出发生文件变更的行及前后几行，不会完整的显示出所有的代码。
```
$ svn diff README.md
Index: README.md
===================================================================
--- README.md   (revision 2608)
+++ README.md   (working copy)
@@ -1,6 +1,7 @@
 # frontend

 > A Vue.js project
+测试diff

 ## Build Setup
```
因此通过`arc diff`命令，上传到phabricator网页的代码，也是非完整的。这在审查代码时有时（例如仅一行代码发生变更）无法获取上下文语境。  
为了查看完整的diff文件，可以采用Gnu diff工具。

## 安装Gnu diff
下载网页：http://gnuwin32.sourceforge.net/packages/diffutils.htm，
点击下载Complete package，然后安装。
```
Description		Download		Size		Last change		Md5sum
• Complete package, except sources	 	Setup	 	2145352	 	24 May 2004	 	d39323cc46969ccbbab26321e238953e
```
安装完成后，在磁盘上创建一个批处理文件——`mydiff.bat`，输入以下内容：
```
@echo off
F:\GnuWin32\bin\diff.exe  -U9999999 -L %3 -L %5 %6 %7
```
这里`F:\GnuWin32\bin\diff.exe`是该文件的路径，替换成你自己的。

## 修改svn配置
这里以Tortotise SVN客户端为例，右键->settings->General, 点击Edit按钮，打开Subversion configuration file, 找到
```
# diff-cmd = diff_program (diff, gdiff, etc.)
```
这行，后面添加
```
diff-cmd = F:\GnuWin32\mydiff.bat
```
保存文件。
现在执行`svn diff`就可以得到获取完整的文件了，
```
$ svn diff README.md
Index: README.md
===================================================================
--- README.md   (revision 2608)
+++ README.md   (working copy)
@@ -1,21 +1,22 @@
 # frontend

 > A Vue.js project
+测试diff

 ## Build Setup

 ``` bash
 # install dependencies
 npm install

 # serve with hot reload at localhost:8080
 npm run dev

 # build for production with minification
 npm run build

 # build for production and view the bundle analyzer report
 npm run build --report

 For a detailed explanation on how things work, check out the [guide](http://vuejs-templates.github.io/webpack/) and [docs for vue-loader](http://vuejs.github.io/vue-loader).
```
