---
layout: post
title: 如何解决windows下mysql-python 安装报错的情况
description:
date: 2018-09-05 19:13:00
categories:
comments: true
tags: 
  - python
  - Error
---



###  cmd下 pip install mysql-python 报错

> 今天在windows使用pycharm连接数据库时失败，需要安装mysql-python
>
> 但在安装mysql-python时报错：

error: Microsoft Visual C++ 14.0 is required. Get it with “Microsoft Visual C++ Build Tools”: <http://landinghub.visualstudio.com/visual-cpp-build-tools>

![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/mysql-python.png)



在https://www.lfd.uci.edu/~gohlke/pythonlibs/#wordcloud页面中找到mysql-python

![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/whl.png)

下载到本地进行pip install  xxx

但是，接下来又出现了：

![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/error2.png)



试了网上几种解决方法，终于搞懂为什么了:

> 这是Python2.7版本的，需要手动把whl文件名改为cp36(因为我使用的为Python3.6)

接下来就安装成功了

![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/ok.png)



2018-09-05 19:10 解决完问题

2018-09-05 19:56 解决好markdown格式、图片显示等问题；


**！！！py3.6使用mysqlclient就完美解决**
**mysqlclient与mysql-python接口是一样的**
**https://www.lfd.uci.edu/~gohlke/pythonlibs/#wordcloud  网址下搜索mysqlclient 找到cp36 下载 安装 OK！**


