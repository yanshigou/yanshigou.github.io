---
title: "python最火爬虫框架Scrapy入门"
date: 2019-01-16 15:59
author: dzt
subtitle:  Scrapy、MongoDB、豆瓣电影
tags:
  - scrapy
  - study
  - mongodb
  - 爬虫
---

## 前言
**Scrapy，Python开发的一个快速,高层次的web抓取框架，用于抓取web站点并从页面中提取结构化的数据。Scrapy用途广泛，可以用于数据挖掘、监测和自动化测试。任何人都可以根据需求方便的修改。**

**Scrapy是一套基于Twisted的异步处理框架，是纯python实现的爬虫框架，用户只需要定制开发几个模块就可以轻松的实现一个爬虫，用来抓取网页内容或者各种图片**



## 环境

* python3.6.5 64位
* win10 64位



## 安装

**1.cmd中进入虚拟环境**

> 这个不再过多说明
> 
> **pip install scrapy**

报出如下错误：

```
    error: Microsoft Visual C++ 14.0 is required. Get it with "Microsoft Visual C++ Build Tools": https://visualstudio.microsoft.com/downloads/

    ----------------------------------------
Command "d:\virtualenv\scrapy_py3\scripts\python3.exe -u -c "import setuptools, tokenize;__file__='C:\\Users\\shenz\\AppData\\Local\\Temp\\pip-install-bto36w94\\Twisted\\setup.py';f=getattr(tokenize, 'open', open)(__file__);code=f.read().replace('\r\n', '\n');f.close();exec(compile(code, __file__, 'exec'))" install --record C:\Users\shenz\AppData\Local\Temp\pip-record-dx1n8s3h\install-record.txt --single-version-externally-managed --compile --install-headers d:\virtualenv\scrapy_py3\include\site\python3.6\Twisted" failed with error code 1 in C:\Users\shenz\AppData\Local\Temp\pip-install-bto36w94\Twisted\
```

### 解决

1. 通过[Unofficial Windows Binaries for Python Extension Packages](http://www.lfd.uci.edu/~gohlke/pythonlibs/#twisted)下载twisted对应版本的whl文件。

> 我的为Twisted-18.4.0-cp36-cp36m-win_amd64.whl

2. 运行命令：`pip install Twisted-18.4.0-cp36-cp36m-win_amd64.whl`

>注意路径

4. 再次进行 `pip install scrapy` 即可



**2.cmd中输入scrapy**

> scrapy

```
(scrapy_py3) D:\virtualenv\scrapy_py3\Scripts>scrapy
Scrapy 1.5.1 - no active project

Usage:
  scrapy <command> [options] [args]

Available commands:
  bench         Run quick benchmark test
  fetch         Fetch a URL using the Scrapy downloader
  genspider     Generate new spider using pre-defined templates
  runspider     Run a self-contained spider (without creating a project)
  settings      Get settings values
  shell         Interactive scraping console
  startproject  Create new project
  version       Print Scrapy version
  view          Open URL in browser, as seen by Scrapy

  [ more ]      More commands available when run from project directory

Use "scrapy <command> -h" to see more info about a command
```

出现**Scrapy 1.5.1 - no active project**就表示安装成功



## 安装MongoDB

**1.在官网上下载**[https://www.mongodb.com/download-center/community](https://www.mongodb.com/download-center/community)

> 我使用的是3.6.9 windows 64-bit x64  MSI

**2.下载完成后双击安装，过程中去掉 Install MongoDB Compass**

**3.创建数据目录和日志目录**

```
mkdir mongodb
cd mongodb
mkdir log 
```



### 测试连接

**运行MongoDB服务器**

```
C:\Program Files\MongoDB\Server\3.6\bin>mongod --dbpath d:mongodb
2019-01-16T01:05:56.968-0700 I CONTROL  [initandlisten] MongoDB starting : pid=21012 port=27017 dbpath=d:mongodb 64-bit host=DESKTOP-JP2T8KQ
2019-01-16T01:05:56.968-0700 I CONTROL  [initandlisten] targetMinOS: Windows 7/Windows Server 2008 R2
2019-01-16T01:05:56.969-0700 I CONTROL  [initandlisten] db version v3.6.9
```

> 大概是这个样子 省略了很多

**再新开一个命令终端**

```
C:\Program Files\MongoDB\Server\3.6\bin>mongo.exe
MongoDB shell version v3.6.9
connecting to: mongodb://127.0.0.1:27017
xxx
> exit
bye
```

> 中间也省略了很多
>
> 出现> 表示连接成功 输入exit 退出



### 配置MongoDB的Windows服务

> 当mongod.exe被关闭时，mongo.exe 就无法连接到数据库了，因此每次想使用mongodb数据库都要开启mongod.exe程序，所以比较麻烦，此时我们可以将MongoDB安装为windows服务

**在终端输入**

> 需要使用管理员权限
>
> 配置db路径和log路径
>
> 先在log文件夹下创建一个mongdb.log

```
C:\Program Files\MongoDB\Server\3.6\bin>mongod.exe --dbpath "d:\mongodb\db" --logpath "d:\mongodb\log\mongdb.log" --install --serviceName MongoDB
```

**启动服务**

```
C:\Program Files\MongoDB\Server\3.6\bin>net start MongoDB
MongoDB 服务正在启动 ..
MongoDB 服务已经启动成功。
```

**连接MongoDB**

```
C:\Program Files\MongoDB\Server\3.6\bin>mongo
MongoDB shell version v3.6.9
connecting to: mongodb://127.0.0.1:27017
Implicit session: session { "id" : UUID("b178b4c8-cba2-4886-8e7c-b9958e839b0d") }
MongoDB server version: 3.6.9
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
> exit
bye
```

**关闭服务**

```
C:\Program Files\MongoDB\Server\3.6\bin>net stop MongoDB
MongoDB 服务正在停止.
MongoDB 服务已成功停止。
```

**删除进程**

```
C:\Program Files\MongoDB\Server\3.6\bin>mongod.exe --dbpath "d:\mongodb\db" --logpath "d:\mongodb\log\mongdb.log" --remove --serviceName MongoDB
```


### 配置环境变量

右键我的电脑 > 高级系统设置 > 环境变量 > 系统变量 > Path > 新建添加 C:\Program Files\MongoDB\Server\3.6\bin

然后新开一个终端

直接使用mongo 能进入 表明配置成功



## 使用 no sql manager for mongodb windows 可视化工具

我的版本：

> | Product							    | Version  |	Release		 |	Size	      |
> | NoSQL Manager for MongoDB Freeware | 4.10.0.4 | January 14, 2019  | 34 MByte |

使用默认设置，连接本地MongoDB

![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/mongodb.png)

连接成功

![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/mongodb1.png)





## 创建项目

**进入命令终端，虚拟环境**

```
(scrapy_py3) D:\dzt>scrapy startproject douban
New Scrapy project 'douban', using template directory 'd:\\virtualenv\\scrapy_py3\\lib\\site-packages\\scrapy\\templates\\project', created in:
    D:\dzt\douban

You can start your first spider with:
    cd douban
    scrapy genspider example example.com
```

**使用pycharm打开项目**

![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/douban.png)

**再回到终端**

> 创建爬虫文件douban_spider 和 路径movie.douban.com （豆瓣电影）

```
(scrapy_py3) D:\dzt\douban>cd douban\spiders

(scrapy_py3) D:\dzt\douban\douban\spiders>scrapy genspider douban_spider movie.douban.com
Created spider 'douban_spider' using template 'basic' in module:
  douban.spiders.douban_spider
```

![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/douban.png)

**第一步就完成了**



## 编写项目

### 修改itmes.py

```python
# -*- coding: utf-8 -*-

# Define here the models for your scraped items
#
# See documentation in:
# https://doc.scrapy.org/en/latest/topics/items.html

import scrapy


class DoubanItem(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    # 序号 电影名称 电影介绍 星级 电影评论数 电影描述
    serial_number = scrapy.Field()
    movie_name = scrapy.Field()
    introduce = scrapy.Field()
    star = scrapy.Field()
    evaluate = scrapy.Field()
    describe = scrapy.Field()

```



### 测试

douban_spider.py

```python
# -*- coding: utf-8 -*-
import scrapy


class DoubanSpiderSpider(scrapy.Spider):
    # 爬虫名
    name = 'douban_spider'
    # 域名
    allowed_domains = ['movie.douban.com']
    # 入口url, 扔到调度器里面去
    start_urls = ['https://movie.douban.com/top250']

    def parse(self, response):
        print(response.text)
```

**先只打印一下信息 测试是否可行**

**回到终端输入scrapy crawl douban_spider**

```python
(scrapy_py3) D:\dzt\douban\douban\spiders>scrapy crawl douban_spider
    
2019-01-16 18:01:08 [scrapy.utils.log] INFO: Scrapy 1.5.1 started (bot: douban)
2019-01-16 18:01:08 [scrapy.utils.log] INFO: Versions: lxml 4.3.0.0, libxml2 2.9.5, cssselect 1.0.3, parsel 1.5.1, w3lib 1.20.0, Twisted 18.9.0, Python 3.6.5 (v3.6.5:f59c0932b4, Mar 28 2018, 17:00:18) [MSC v.1900 64 bit (AMD64)], pyOpenSSL 18.0.0 (OpenSSL 1.1.0j  20 Nov 2018), cryptography 2.4.2, Platform Windows-10-10.0.17134-SP0
2019-01-16 18:01:08 [scrapy.crawler] INFO: Overridden settings: {'BOT_NAME': 'douban', 'DOWNLOAD_DELAY': 0.5, 'NEWSPIDER_MODULE': 'douban.spiders', 'SPIDER_MODULES': ['douban.spiders']}
2019-01-16 18:01:08 [scrapy.middleware] INFO: Enabled extensions:
['scrapy.extensions.corestats.CoreStats',
 'scrapy.extensions.telnet.TelnetConsole',
 'scrapy.extensions.logstats.LogStats']
2019-01-16 18:01:08 [scrapy.middleware] INFO: Enabled downloader middlewares:
['scrapy.downloadermiddlewares.httpauth.HttpAuthMiddleware',
 'scrapy.downloadermiddlewares.downloadtimeout.DownloadTimeoutMiddleware',
 'scrapy.downloadermiddlewares.defaultheaders.DefaultHeadersMiddleware',
 'scrapy.downloadermiddlewares.useragent.UserAgentMiddleware',
 'scrapy.downloadermiddlewares.retry.RetryMiddleware',
 'scrapy.downloadermiddlewares.redirect.MetaRefreshMiddleware',
 'scrapy.downloadermiddlewares.httpcompression.HttpCompressionMiddleware',
 'scrapy.downloadermiddlewares.redirect.RedirectMiddleware',
 'scrapy.downloadermiddlewares.cookies.CookiesMiddleware',
 'scrapy.downloadermiddlewares.httpproxy.HttpProxyMiddleware',
 'scrapy.downloadermiddlewares.stats.DownloaderStats']
2019-01-16 18:01:08 [scrapy.middleware] INFO: Enabled spider middlewares:
['scrapy.spidermiddlewares.httperror.HttpErrorMiddleware',
 'scrapy.spidermiddlewares.offsite.OffsiteMiddleware',
 'scrapy.spidermiddlewares.referer.RefererMiddleware',
 'scrapy.spidermiddlewares.urllength.UrlLengthMiddleware',
 'scrapy.spidermiddlewares.depth.DepthMiddleware']
2019-01-16 18:01:08 [scrapy.middleware] INFO: Enabled item pipelines:
[]
2019-01-16 18:01:08 [scrapy.core.engine] INFO: Spider opened
2019-01-16 18:01:09 [scrapy.extensions.logstats] INFO: Crawled 0 pages (at 0 pages/min), scraped 0 items (at 0 items/min)
2019-01-16 18:01:09 [scrapy.extensions.telnet] DEBUG: Telnet console listening on 127.0.0.1:6023
2019-01-16 18:01:09 [scrapy.core.engine] DEBUG: Crawled (403) <GET https://movie.douban.com/top250> (referer: None)
2019-01-16 18:01:09 [scrapy.spidermiddlewares.httperror] INFO: Ignoring response <403 https://movie.douban.com/top250>: HTTP status code is not handled or not allowed
2019-01-16 18:01:09 [scrapy.core.engine] INFO: Closing spider (finished)
2019-01-16 18:01:09 [scrapy.statscollectors] INFO: Dumping Scrapy stats:
{'downloader/request_bytes': 221,
 'downloader/request_count': 1,
 'downloader/request_method_count/GET': 1,
 'downloader/response_bytes': 248,
 'downloader/response_count': 1,
 'downloader/response_status_count/403': 1,
 'finish_reason': 'finished',
 'finish_time': datetime.datetime(2019, 1, 16, 10, 1, 9, 641424),
 'httperror/response_ignored_count': 1,
 'httperror/response_ignored_status_count/403': 1,
 'log_count/DEBUG': 2,
 'log_count/INFO': 8,
 'response_received_count': 1,
 'scheduler/dequeued': 1,
 'scheduler/dequeued/memory': 1,
 'scheduler/enqueued': 1,
 'scheduler/enqueued/memory': 1,
 'start_time': datetime.datetime(2019, 1, 16, 10, 1, 9, 117851)}
2019-01-16 18:01:09 [scrapy.core.engine] INFO: Spider closed (finished)
```

**结果返回403**

#### 解决

**修改settings**

找到USER_AGENT  修改为任意一个正确的请求头

```python
# Crawl responsibly by identifying yourself (and your website) on the user-agent
USER_AGENT = 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.77 Safari/537.36'
```

再次终端运行**scrapy crawl douban_spider**就能正确打印出整个网页的所有信息

**还可以在settings同级目录下新建一个main.py**

```python
# -*- coding: utf-8 -*-
__author__ = "dzt"
__date__ = "2019/1/17 9:42"

from scrapy import cmdline
cmdline.execute('scrapy crawl douban_spider'.split())
```

**直接run main.py 就更为方便**





### spider 编写

首先F12对网页进行分析

找到豆瓣电影下的 **\<ol class="grid_view">**

![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/ol.png)

copy它的Xpath

```python
def parse(self, response):
    movie_list = response.xpath('//*[@id="content"]/div/div[1]/ol[@class="grid_view"]/li')

    for i in movie_list:
        print(i)
        print(len(movie_list))
```

运行main.py

```
<Selector xpath='//*[@id="content"]/div/div[1]/ol[@class="grid_view"]/li' data='<li>\n            <div class="item">\n    '>
<Selector xpath='//*[@id="content"]/div/div[1]/ol[@class="grid_view"]/li' data='<li>\n            <div class="item">\n    '>
<Selector xpath='//*[@id="content"]/div/div[1]/ol[@class="grid_view"]/li' data='<li>\n            <div class="item">\n    '>
<Selector xpath='//*[@id="content"]/div/div[1]/ol[@class="grid_view"]/li' data='<li>\n            <div class="item">\n    '>
<Selector xpath='//*[@id="content"]/div/div[1]/ol[@class="grid_view"]/li' data='<li>\n            <div class="item">\n    '>
<Selector xpath='//*[@id="content"]/div/div[1]/ol[@class="grid_view"]/li' data='<li>\n            <div class="item">\n    '>
<Selector xpath='//*[@id="content"]/div/div[1]/ol[@class="grid_view"]/li' data='<li>\n            <div class="item">\n    '>
<Selector xpath='//*[@id="content"]/div/div[1]/ol[@class="grid_view"]/li' data='<li>\n            <div class="item">\n    '>
<Selector xpath='//*[@id="content"]/div/div[1]/ol[@class="grid_view"]/li' data='<li>\n            <div class="item">\n    '>
<Selector xpath='//*[@id="content"]/div/div[1]/ol[@class="grid_view"]/li' data='<li>\n            <div class="item">\n    '>
<Selector xpath='//*[@id="content"]/div/div[1]/ol[@class="grid_view"]/li' data='<li>\n            <div class="item">\n    '>
<Selector xpath='//*[@id="content"]/div/div[1]/ol[@class="grid_view"]/li' data='<li>\n            <div class="item">\n    '>
<Selector xpath='//*[@id="content"]/div/div[1]/ol[@class="grid_view"]/li' data='<li>\n            <div class="item">\n    '>
<Selector xpath='//*[@id="content"]/div/div[1]/ol[@class="grid_view"]/li' data='<li>\n            <div class="item">\n    '>
<Selector xpath='//*[@id="content"]/div/div[1]/ol[@class="grid_view"]/li' data='<li>\n            <div class="item">\n    '>
<Selector xpath='//*[@id="content"]/div/div[1]/ol[@class="grid_view"]/li' data='<li>\n            <div class="item">\n    '>
<Selector xpath='//*[@id="content"]/div/div[1]/ol[@class="grid_view"]/li' data='<li>\n            <div class="item">\n    '>
<Selector xpath='//*[@id="content"]/div/div[1]/ol[@class="grid_view"]/li' data='<li>\n            <div class="item">\n    '>
<Selector xpath='//*[@id="content"]/div/div[1]/ol[@class="grid_view"]/li' data='<li>\n            <div class="item">\n    '>
<Selector xpath='//*[@id="content"]/div/div[1]/ol[@class="grid_view"]/li' data='<li>\n            <div class="item">\n    '>
<Selector xpath='//*[@id="content"]/div/div[1]/ol[@class="grid_view"]/li' data='<li>\n            <div class="item">\n    '>
<Selector xpath='//*[@id="content"]/div/div[1]/ol[@class="grid_view"]/li' data='<li>\n            <div class="item">\n    '>
<Selector xpath='//*[@id="content"]/div/div[1]/ol[@class="grid_view"]/li' data='<li>\n            <div class="item">\n    '>
<Selector xpath='//*[@id="content"]/div/div[1]/ol[@class="grid_view"]/li' data='<li>\n            <div class="item">\n    '>
<Selector xpath='//*[@id="content"]/div/div[1]/ol[@class="grid_view"]/li' data='<li>\n            <div class="item">\n    '>
25
```

**拿到每个li之后 就可以再对li进行提取信息**



```python
    def parse(self, response):
        movie_list = response.xpath('//*[@id="content"]/div/div[1]/ol[@class="grid_view"]/li')

        for i in movie_list:
            douban_item = DoubanItem()
            douban_item['serial_number'] = i.xpath(".//div[@class='item']//em/text()").extract_first()
            douban_item['movie_name'] = i.xpath(".//div[@class='info']/div[@class='hd']/a/span[1]/text()").extract_first()
            content = i.xpath(".//div[@class='info']/div[@class='bd']/p[1]/text()").extract()
            for c in content:
                a = ''.join(c.split())
                douban_item['introduce'] = a
            douban_item['star'] = i.xpath(".//span[@class='rating_num']/text()").extract_first()
            douban_item['evaluate'] = i.xpath(".//div[@class='star']/span[4]/text()").extract_first()
            douban_item['describe'] = i.xpath(".//span[@class='inq']/text()").extract_first()
            douban_item['src'] = i.xpath(".//div[@class='info']/div[@class='hd']/a/@href").extract_first()
            print(douban_item)
```

**效果如下**

```
{'describe': '希望让人自由。',
 'evaluate': '1271973人评价',
 'introduce': '1994/美国/犯罪剧情',
 'movie_name': '肖申克的救赎',
 'serial_number': '1',
 'src': 'https://movie.douban.com/subject/1292052/',
 'star': '9.6'}
{'describe': '风华绝代。',
 'evaluate': '938353人评价',
 'introduce': '1993/中国大陆香港/剧情爱情同性',
 'movie_name': '霸王别姬',
 'serial_number': '2',
 'src': 'https://movie.douban.com/subject/1291546/',
 'star': '9.6'}
{'describe': '怪蜀黍和小萝莉不得不说的故事。',
 'evaluate': '1167588人评价',
 'introduce': '1994/法国/剧情动作犯罪',
 'movie_name': '这个杀手不太冷',
 'serial_number': '3',
 'src': 'https://movie.douban.com/subject/1295644/',
 'star': '9.4'}
{'describe': '一部美国近现代史。',
 'evaluate': '1002560人评价',
 'introduce': '1994/美国/剧情爱情',
 'movie_name': '阿甘正传',
 'serial_number': '4',
 'src': 'https://movie.douban.com/subject/1292720/',
 'star': '9.4'}
{'describe': '最美的谎言。',
 'evaluate': '586341人评价',
 'introduce': '1997/意大利/剧情喜剧爱情战争',
 'movie_name': '美丽人生',
 'serial_number': '5',
 'src': 'https://movie.douban.com/subject/1292063/',
 'star': '9.5'}
{'describe': '失去的才是永恒的。 ',
 'evaluate': '938504人评价',
 'introduce': '1997/美国/剧情爱情灾难',
 'movie_name': '泰坦尼克号',
 'serial_number': '6',
 'src': 'https://movie.douban.com/subject/1292722/',
 'star': '9.3'}
{'describe': '最好的宫崎骏，最好的久石让。 ',
 'evaluate': '931662人评价',
 'introduce': '2001/日本/剧情动画奇幻',
 'movie_name': '千与千寻',
 'serial_number': '7',
 'src': 'https://movie.douban.com/subject/1291561/',
 'star': '9.3'}
{'describe': '拯救一个人，就是拯救整个世界。',
 'evaluate': '525570人评价',
 'introduce': '1993/美国/剧情历史战争',
 'movie_name': '辛德勒的名单',
 'serial_number': '8',
 'src': 'https://movie.douban.com/subject/1295124/',
 'star': '9.5'}
{'describe': '诺兰给了我们一场无法盗取的梦。',
 'evaluate': '1015248人评价',
 'introduce': '2010/美国英国/剧情科幻悬疑冒险',
 'movie_name': '盗梦空间',
 'serial_number': '9',
 'src': 'https://movie.douban.com/subject/3541415/',
 'star': '9.3'}
{'describe': '小瓦力，大人生。',
 'evaluate': '675065人评价',
 'introduce': '2008/美国/爱情科幻动画冒险',
 'movie_name': '机器人总动员',
 'serial_number': '10',
 'src': 'https://movie.douban.com/subject/2131459/',
 'star': '9.3'}
{'describe': '永远都不能忘记你所爱的人。',
 'evaluate': '662207人评价',
 'introduce': '2009/美国英国/剧情',
 'movie_name': '忠犬八公的故事',
 'serial_number': '11',
 'src': 'https://movie.douban.com/subject/3011091/',
 'star': '9.3'}
{'describe': '英俊版憨豆，高情商版谢耳朵。',
 'evaluate': '910148人评价',
 'introduce': '2009/印度/剧情喜剧爱情歌舞',
 'movie_name': '三傻大闹宝莱坞',
 'serial_number': '12',
 'src': 'https://movie.douban.com/subject/3793023/',
 'star': '9.2'}
{'describe': '每个人都要走一条自己坚定了的路，就算是粉身碎骨。 ',
 'evaluate': '757516人评价',
 'introduce': '1998/意大利/剧情音乐',
 'movie_name': '海上钢琴师',
 'serial_number': '13',
 'src': 'https://movie.douban.com/subject/1292001/',
 'star': '9.2'}
{'describe': '天籁一般的童声，是最接近上帝的存在。 ',
 'evaluate': '627439人评价',
 'introduce': '2004/法国瑞士德国/剧情音乐',
 'movie_name': '放牛班的春天',
 'serial_number': '14',
 'src': 'https://movie.douban.com/subject/1291549/',
 'star': '9.2'}
{'describe': '一生所爱。',
 'evaluate': '697510人评价',
 'introduce': '1995/香港中国大陆/喜剧爱情奇幻冒险',
 'movie_name': '大话西游之大圣娶亲',
 'serial_number': '15',
 'src': 'https://movie.douban.com/subject/1292213/',
 'star': '9.2'}
{'describe': '如果再也不能见到你，祝你早安，午安，晚安。',
 'evaluate': '680560人评价',
 'introduce': '1998/美国/剧情科幻',
 'movie_name': '楚门的世界',
 'serial_number': '16',
 'src': 'https://movie.douban.com/subject/1292064/',
 'star': '9.2'}
{'describe': '人人心中都有个龙猫，童年就永远不会消失。',
 'evaluate': '614901人评价',
 'introduce': '1988/日本/动画奇幻冒险',
 'movie_name': '龙猫',
 'serial_number': '17',
 'src': 'https://movie.douban.com/subject/1291560/',
 'star': '9.1'}
{'describe': '爱是一种力量，让我们超越时空感知它的存在。',
 'evaluate': '691820人评价',
 'introduce': '2014/美国英国加拿大冰岛/剧情科幻冒险',
 'movie_name': '星际穿越',
 'serial_number': '18',
 'src': 'https://movie.douban.com/subject/1889243/',
 'star': '9.2'}
{'describe': '千万不要记恨你的对手，这样会让你失去理智。',
 'evaluate': '459494人评价',
 'introduce': '1972/美国/剧情犯罪',
 'movie_name': '教父',
 'serial_number': '19',
 'src': 'https://movie.douban.com/subject/1291841/',
 'star': '9.2'}
{'describe': '我们一路奋战不是为了改变世界，而是为了不让世界改变我们。',
 'evaluate': '396535人评价',
 'introduce': '2011/韩国/剧情',
 'movie_name': '熔炉',
 'serial_number': '20',
 'src': 'https://movie.douban.com/subject/5912992/',
 'star': '9.3'}
{'describe': '香港电影史上永不过时的杰作。',
 'evaluate': '573580人评价',
 'introduce': '2002/香港/剧情犯罪悬疑',
 'movie_name': '无间道',
 'serial_number': '21',
 'src': 'https://movie.douban.com/subject/1307914/',
 'star': '9.1'}
{'describe': '平民励志片。 ',
 'evaluate': '735993人评价',
 'introduce': '2006/美国/剧情传记家庭',
 'movie_name': '当幸福来敲门',
 'serial_number': '22',
 'src': 'https://movie.douban.com/subject/1849031/',
 'star': '9.0'}
{'describe': '迪士尼给我们营造的乌托邦就是这样，永远善良勇敢，永远出乎意料。',
 'evaluate': '767198人评价',
 'introduce': '2016/美国/喜剧动画冒险',
 'movie_name': '疯狂动物城',
 'serial_number': '23',
 'src': 'https://movie.douban.com/subject/25662329/',
 'star': '9.2'}
{'describe': '满满温情的高雅喜剧。',
 'evaluate': '486242人评价',
 'introduce': '2011/法国/剧情喜剧',
 'movie_name': '触不可及',
 'serial_number': '24',
 'src': 'https://movie.douban.com/subject/6786002/',
 'star': '9.2'}
{'describe': '真正的幸福是来自内心深处。',
 'evaluate': '797957人评价',
 'introduce': '2010/美国/剧情喜剧爱情',
 'movie_name': '怦然心动',
 'serial_number': '25',
 'src': 'https://movie.douban.com/subject/3319755/',
 'star': '9.0'}
```



### 解析下一页的路径

```python
next_link = response.xpath("//span[@class='next']/link/@href").extract()
# print(next_link)
if next_link:
    next_link = next_link[0]
    yield scrapy.Request("https://movie.douban.com/top250"+next_link, callback=self.parse)
```



### 完整douban_spider.py

```python
# -*- coding: utf-8 -*-
import scrapy
from douban.items import DoubanItem


class DoubanSpiderSpider(scrapy.Spider):
    # 爬虫名
    name = 'douban_spider'
    # 域名
    allowed_domains = ['movie.douban.com']
    # 入口url, 扔到调度器里面去
    start_urls = ['https://movie.douban.com/top250']

    # 默认解析方法
    def parse(self, response):
        # 解析循环条目
        movie_list = response.xpath('//*[@id="content"]/div/div[1]/ol[@class="grid_view"]/li')

        for i in movie_list:
            douban_item = DoubanItem()
            # 详细xpath 进行数据解析
            douban_item['serial_number'] = i.xpath(".//div[@class='item']//em/text()").extract_first()
            douban_item['movie_name'] = i.xpath(".//div[@class='info']/div[@class='hd']/a/span[1]/text()").extract_first()
            content = i.xpath(".//div[@class='info']/div[@class='bd']/p[1]/text()").extract()
            for c in content:
                a = ''.join(c.split())
                douban_item['introduce'] = a
            douban_item['star'] = i.xpath(".//span[@class='rating_num']/text()").extract_first()
            douban_item['evaluate'] = i.xpath(".//div[@class='star']/span[4]/text()").extract_first()
            douban_item['describe'] = i.xpath(".//span[@class='inq']/text()").extract_first()
            douban_item['src'] = i.xpath(".//div[@class='info']/div[@class='hd']/a/@href").extract_first()
            # print(douban_item)
            yield douban_item
        # 解析下一页的路径
        next_link = response.xpath("//span[@class='next']/link/@href").extract()
        # print(next_link)
        if next_link:
            next_link = next_link[0]
            yield scrapy.Request("https://movie.douban.com/top250"+next_link, callback=self.parse)
```



### json和csv文件

终端下使用

```
scrapy crawl douban_spider -o douban.csv

scrapy crawl douban_spider -o douban.json
```

**csv文件需要注意使用Notepad++修改格式为UTF-8-BOM编码**

![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/csv.png)

**json文件是Unicode存储的**

![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/json.png)