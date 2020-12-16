---
title: "Ubuntu下用crontab 部署定时任务  定时切割django log"
date: 2020-12-16 18:18
author: dzt
subtitle: Ubuntu下用crontab 部署定时任务 
tags:
  - ubuntu
  - Django
  - crontab
---


#### crontab 
> uwsgi代理django  时间长了会让log不方便阅读 和 脚本分析  所以上网查阅了一下 log切割



crontab –e  初次使用后需要选择 3   然后进入文件编辑    

我的设置 

```
root@hecs-x-large-2-linux-20200703155013:~# crontab -l
59 23  *  *  *  sh /root/www/FirePatrolForWeChat/touchforlogrotate.sh
```

表示 每天23点59分进行操作  sh touchforlogrotate.sh



##### 常用的周期格式



周期有5个域，分别是分钟，小时，日(day of month)，月（month of year），周几（day of week）。每个域不加限制任意的话用*，整体格式为：

\* * * * * user command

每五分钟执行 */5 * * * *

每小时执行  0 * * * *

每天执行    0 0 * * *

每周执行    0 0 * * 0

每月执行    0 0 1 * *

每年执行    0 0 1 1 *

每分钟执行一次 * * * * * user command

每隔2小时执行一次**/2 ** * user command (/表示频率)

每天8:30分执行一次308 * * * user command

每小时的30和50分各执行一次  30,50 * * * * user command（,表示并列）

每个月的3号到6号的8:30执行一次 30 8 3-6 * * user command （-表示范围）

每个星期一的  和7代表周日）



service cron start   //启动服务

service cron stop   //关闭服务

service cron restart  //重启服务

service cron reload  //重新载入配置

service cron status  //查看crontab服务状态


