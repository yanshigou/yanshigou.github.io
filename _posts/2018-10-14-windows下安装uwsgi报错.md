---
layout: post
title: 解决windows下安装uwsgi失败
date: 2018-10-14 17:10:00
comments: true
subtitle:  Error：module 'os' has no attribute 'uname'
author: dzt
tags: 
  - python
  - Error
---



### windows下pip install uwsgi 提示 os 没有 uname()方法的解决办法

在windows下，安装uwsgi。

直接pip install uwsgi 或者 从网上下载好uwsgi后，cmd到该解压后的目录中，python setup.py install的时候，报错，说是os没有uname()这个方法。![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/uwsgiError.png)

定位到uwsgiconfig.py文件中，首先import platform后，将os.uname()都改为platform.uname()即可。

**os.uname()是不支持windows系统的。platform模块是支持任何系统**

![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/platform.png)



### 然后新的问题又出现了 没有C语音编译器 需要装一个MinGW

![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/C.png)

下载安装一个 MinGW Installation manager

![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/MinGW-Installation.png)

对着mingw32-gcc-g++ 右键mark for installation 然后主菜单栏Installation，点击Apply，然后等一下，自动安装。

![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/MinGW.png)



然后配置环境变量 在cmd中输入 gcc -v 测试是否安装成功

![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/gcc.png)





然后再重新cmd到uwsgi解压后的目录中，python setup.py install



**花了很久的时候搞后面的操作，，还是在虚拟机中使用uwsgi好**