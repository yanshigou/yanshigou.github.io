---
title: "django-celery如何使用和配置"
date: 2018-12-28 13:55
author: dzt
subtitle: celery在django mysql数据库下的使用和配置
tags:
  - django
  - celery
  - mysql
---



>上一篇文章我详细的写了如何在服务器上搭建异步任务队列项目
> 这篇就文章详细的写如何创建一个django-celery项目


## 前言
首先简单介绍一下，Celery 是一个强大的分布式任务队列，它可以让任务的执行完全脱离主程序，甚至可以被分配到其他主机上运行。我们通常使用它来实现异步任务（async task）和定时任务（crontab）。





### 它的架构组成如下图： 


![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/celery-jiagoutu1.png)

![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/celery-jiagoutu2.png)



### 框架集成

* Django	django-celery
* Pyramid	pyramid_celery
* Pylons	celery-pylons
* Flask	不需要
* web2py	web2py-celery
* Tornado	tornado-celery





## 正文

> 上面简单介绍了一下，从网上获取的图片和官方的文档



可以看到，Celery 主要包含以下几个模块：

**任务模块 Task**

> 包含异步任务和定时任务。其中，异步任务通常在业务逻辑中被触发并发往任务队列，而定时任务由 Celery Beat 进程周期性地将任务发往任务队列。

**消息中间件 Broker**

> Broker，即为任务调度队列，接收任务生产者发来的消息（即任务），将任务存入队列。Celery 本身不提供队列服务，官方推荐使用 RabbitMQ 和 Redis 等。
>
> 由于本项目比较小，使用django作为Broker

**任务执行单元 Worker**

> Worker 是执行任务的处理单元，它实时监控消息队列，获取队列中调度的任务，并执行它。

**任务结果存储 Backend**

> Backend 用于存储任务的执行结果，以供查询。同消息中间件一样，存储也可使用 RabbitMQ, Redis 和 MongoDB 等。

**异步任务** 

> 使用 Celery 实现异步任务主要包含三个步骤：
>
> > 创建一个 Celery 实例 
> > 启动 Celery Worker 
> > 应用程序调用异步任务



**创建django项目和安装对应的环境我就不再说了**



### 一、修改setting配置

```python
#celery setting
import djcelery
djcelery.setup_loader() # 加载celery
BROKER_URL = 'django://ip:port'  # broker地址
BROKER_POOL_LIMIT = 0 # 连接池中可以打开最大连接数 如果设置成None或者0，连接池将会被禁用，并且每次使用连接都会重新建立连接并关闭。
CELERY_RESULT_BACKEND='djcelery.backends.database:DatabaseBackend' # 结果存储
CELERY_TIMEZONE='Asia/Shanghai' # 时区
CELERY_ENABLE_UTC = False # 关闭UTC
CELERYD_CONCURRENCY = 2      # celery worker并发数
CELERYD_MAX_TASKS_PER_CHILD = 4 # 每个worker最大执行任务数


INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',
    'djcelery', # djcelery
    'paopao', # appname
    'kombu.transport.django' # kombu

]
```

> celery任务并发只与celery配置项CELERYD_CONCURRENCY 有关，与CELERYD_MAX_TASKS_PER_CHILD没有关系，即CELERYD_CONCURRENCY=2，只能并发2个worker，此时运行较大的文件，执行两次可以看到两个task任务并行执行，而执行第三个任务时，开始排队，直到两个worker执行完毕。




> celery执行完任务不释放内存与原worker一直没有被销毁有关，因此CELERYD_MAX_TASKS_PER_CHILD可以适当配置小点，而任务并发数与CELERYD_CONCURRENCY配置项有关，每增加一个worker必然增加内存消耗，同时也影响到一个worker何时被销毁，因为celery是均匀调度任务至每个worker，因此也不宜配置过大，适当配置。



### 二、在App下创建celery文件 

```python
# -*- coding: utf-8 -*-
from __future__ import absolute_import

import os

from celery import Celery


# set the default Django settings module for the 'celery' program.
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'test_celery.settings')

from django.conf import settings  # noqa

app = Celery('paopao')

# Using a string here means the worker will not have to
# pickle the object when using Windows.
app.config_from_object('django.conf:settings')
app.autodiscover_tasks(lambda: settings.INSTALLED_APPS)


@app.task(bind=True)
def debug_task(self):
    print('Request: {0!r}'.format(self.request))



```



### 三、修改App下的init.py

```python
from __future__ import absolute_import, unicode_literals


# This will make sure the app is always imported when
# Django starts so that shared_task will use this app.
from paopao.celery import app as celery_app
```



### 四、创建task.py (tools/下)

```python
# -*- coding: utf-8 -*-
from __future__ import absolute_import
from celery import shared_task


@shared_task(track_started=True)
def test_task(time1):
    pass # 自己的逻辑
```



### 五、创建App下的Views.py

```python
# -*- coding: utf-8 -*-
from __future__ import unicode_literals
from django.shortcuts import render
from django.views.generic.base import View
from django.http import HttpResponse, HttpResponseRedirect
from tools.tasks import test_task, get_imei

class UploadView(LoginRequiredMixin, View):
    def post(self, request):
        form = ImeiInfoForm(request.POST, request.FILES)
        if form.is_valid():
            # 获取表单数据
            info = form.cleaned_data['info']
            filename = form.cleaned_data['filename']
            time1 = datetime.now()

            f = PLL()
            f.filename = filename
            f.info = info
            f.starttime = time1
            f.save()


            # 要注意 filename 为<class'django.core.files.uploadedfile.InMemoryUploadedFile'>
            # 名字不能直接等于str(filename), 不然无法保存上传文件
            result = test_task.delay(time1)  ## 这里就是把任务交给task去处理 
            p = PLL.objects.get(starttime=time1)
            p.task_id = result.id
            p.save()
            return HttpResponseRedirect('/uploaded/')
```



**在test_task.delay(time1)处，就直接把任务交给task去处理，不用阻塞，直接返回到/uploaded/**



### 六、同步数据库 

```shell
python manage.py makemigrations
python manage.py migrate
```



### 七、运行

```shell
python manage.py runserver 0.0.0.0:8000 
python manage.py celery worker -A test_celey -l info # test_celey 项目名称
```



### 八、结果图

> [源码在我的github上](https://github.com/yanshigou/MyScripts/tree/master/test_celery) ---> <https://github.com/yanshigou/MyScripts/tree/master/test_celery>

```
[2018-12-28 15:20:18,105: INFO/MainProcess] Connected to django://localhost//
[2018-12-28 15:20:18,137: WARNING/MainProcess] d:\virtualenv\py27\lib\site-packages\celery\fixups\django.py:265: UserWarning: Using settings.DEBUG leads to a memory leak, never use this setting in produc
tion environments!
  warnings.warn('Using settings.DEBUG leads to a memory leak, never '
[2018-12-28 15:20:18,137: WARNING/MainProcess] celery@DESKTOP-JP2T8KQ ready.
```

这就算celery运行成功

![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/celery-worker.png)



### 项目效果图

![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/test_celety1.png)

![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/test_celery2.png)

![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/test_celery3.png)

![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/test_celery4.png)

![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/test_celery5.png)