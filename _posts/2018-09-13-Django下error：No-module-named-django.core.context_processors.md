---
layout: post
title: Django下error：No module named 'django.core.context_processors'
description:
date: 2018-09-13 16:41:00
categories:
comments: true
tags: 
  - Django
  - Error
---

## 在Django项目中更改setting配置中 context_processors报错
```
ModuleNotFoundError: No module named 'django.core.context_processors'
```

![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/core.png)

* 因为我添加了一个

```
django.core.context_processors.media
```

![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/core2.png)

* 是因为放context_processors的路径不一样
* 只需要把路径.core改为.template就ok了
```
django.template.context_processors.media
```
![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/core3.png)

