---
title: "Windows server内网下部署Django项目"
date: 2019-07-06 17:20
author: dzt
subtitle: 在Windows Server内网下解决mysql、python环境、IIS相关问题 
tags:
  - mysql
  - Django
  - Windows
  - IIS
---



## 前言

工作原因需要在公安内网中搭建一个后台管理系统，我是用Django+html实现的，本来是用的mysql作为数据库，但在实际操作过程中，碰到了很多mysql的问题，比如缺少dll文件，程序0xc00007b等错误，后来改为了sqlserver数据库，但也碰到了更多问题，因为不太熟悉微软的sqlserver数据库，又尝试着手解决mysql相关的问题，查阅过很多资料，尝试了10多种方法，成功离线解决了mysql相关的问题



## 离线搭建python环境

> 这一步其实很简单，记录一下

```python
# 导出依赖信息
pip freeze > requirements.txt
# 安装pip2pi
pip install pip2pi
# 创建依赖的文件夹
mkdir dependences
# 下载依赖包到文件夹
pip2pi dependences -r requirements.txt

```



**将python安装包、virtualenv.whl、dependences、requirements.txt拷贝至服务器**

> 先安装好python，搭建好虚拟环境

```python
pip install --no-index --find-links=dependences -r requirements.txt
```





## Windows上IIS部署Django、Windows server上IIS部署django

> 网上资料很多、贴上地址

**Windows-server：**

[django中文网-Windows server iis部署Django详细操作](https://www.django.cn/article/show-21.html)



**Windows：**

[Django Windows+IIS+wfastcgi 环境下部署](https://www.cnblogs.com/wcwnina/p/10960242.html)

![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/IIS0.png)

](https://www.cnblogs.com/wcwnina/p/10960242.html)

![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/IIS1.png)

](https://www.cnblogs.com/wcwnina/p/10960242.html)

![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/IIS2.png)

](https://www.cnblogs.com/wcwnina/p/10960242.html)

![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/IIS3.png)

```
安装IIS
勾选FTP Web管理工具 万维网服务
wfastcgi-enable 获取python路径和wfastcgi路径
IIS添加网站
物理路径为文件夹路径
配置web.config
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <system.webServer>
        <handlers>
            <add name="Python FastCGI" 
                 path="*" 
                 verb="*" 
                 modules="FastCgiModule" 
                 scriptProcessor="python路径和wfastcgi路径" 
                 resourceType="Unspecified" 
                 requireAccess="Script"/>
        </handlers>
    </system.webServer>
    <appSettings>
        <add key="WSGI_HANDLER" value="django.core.wsgi.get_wsgi_application()" />
        <add key="PYTHONPATH" value="Django目录" />
        <add key="DJANGO_SETTINGS_MODULE" value="traffic_mgmt.settings" />
    </appSettings>
</configuration>
添加虚拟目录static
同样配置一个web.config
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <system.webServer>
        <!-- this configuration overrides the FastCGI handler to let IIS serve the static files -->
        <handlers>
            <clear/>
            <add name="StaticFile" path="*" verb="*" modules="StaticFileModule" resourceType="File" requireAccess="Read" />
        </handlers>
    </system.webServer>
</configuration>
media同理
```

