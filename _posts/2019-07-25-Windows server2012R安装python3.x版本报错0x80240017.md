---
title: "Windows server2012R安装python3.x版本报错0x80240017"
date: 2019-07-25 19:40
author: dzt
subtitle: 安装python3.x版本报错0x80240017
tags:
  - Windows
  - python
---


## 前言
今天在安装python3.6版本时候出现错误0x80240017
因为又是在内网中，无法在线更新什么的。
几经周折多数都是说win7，win10的解决方法后面去官网找找到了一个答案给大家。



## 解决方法
经本人测试需要更新3个补丁。而且耗时比较长补丁方面。
KB2919442 ，KB2919355，KB3118401 这三个补丁。
安装需按顺序进行
**其中先安装KB2919442 ，在KB2919355，最后KB3118401 **

[KB2919442链接](https://www.microsoft.com/en-us/download/details.aspx?id=42153)

[KB2919355链接](https://www.microsoft.com/en-us/download/details.aspx?id=42334)

[KB3118401链接](https://www.microsoft.com/en-us/download/details.aspx?id=50410)

**第二个需要注意的是：**
	有多个文件下载，下载**Windows8.1-KB2919355-x64.msu**这一个，第二个特别慢，我一度以为是内网造成的

**第三个是压缩包，需要注意的是：**
	下载下来解压出来也有多个文件，特别注意不要选错了，
	安装**Windows8.1-KB3118401-x64.msu**这一个
	

​	