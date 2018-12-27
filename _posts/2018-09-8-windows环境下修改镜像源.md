---
layout: post
title: windows环境下永久修改pip镜像源的方法
description:
date: 2018-09-08 23:16:00
categories:
comments: true
tags: python
---

## 一、在windows环境下修改pip镜像源的方法
1. 在开始菜单栏搜索中输入 %APPDATA%
2. 会定位到一个新的目录下，在该目录下新建pip文件夹，然后到pip文件夹里面去新建个pip.ini文件(路径应该为："C:\Users\Administrator\AppData\Roaming\pip\pip.ini")
3. 在新建的pip.ini文件中输入以下内容(以豆瓣源为例)：

```
[global]
timeout = 6000
index-url = http://pypi.douban.com/simple
trusted-host = pypi.douban.com
```

pip国内镜像源。

阿里云 http://mirrors.aliyun.com/pypi/simple/
中国科技大学 https://pypi.mirrors.ustc.edu.cn/simple/
豆瓣 http://pypi.douban.com/simple
Python官方 https://pypi.python.org/simple/
v2ex http://pypi.v2ex.com/simple/
中国科学院 http://pypi.mirrors.opencas.cn/simple/
清华大学 https://pypi.tuna.tsinghua.edu.cn/simple/



2018-09-08 23:24
