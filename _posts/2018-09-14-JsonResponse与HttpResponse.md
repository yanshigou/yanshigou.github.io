---
layout: post
title: Django下post请求中使用HttpResponse返回json数据的时候无法获取数据
date: 2018-09-14 23:31:00
comments: true
subtitle: 
author: "dzt"
tags: Django
---


#Django下post请求中使用HttpResponse返回json数据的时候无法获取数据

在使用HttpResponse返回Json给前端页面的时候  并不能异步获取错误提示
![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/httpresponse.png)而且我也设置了 content_type='application/json'  为json格式 也不行



但是在我换为JsonResponse的时候 就能异步返回数据
![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/JsonResponse.png)


具体原因为何暂时我也不清楚 还需要等待解答
