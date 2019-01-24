---
title: "Ubuntu中安装Netdata系统监测工具"
date: 2019-01-24 14:35
author: dzt
subtitle:  Netdata系统实时监测工具
tags:
  - ubuntu
  - Netdata
  - study
---



[**项目Github地址**](https://github.com/ktsaou/netdata)



## 前言

**netdata**是**分布式、实时、性能和健康监控系统和应用程序**。它是一个高度优化的监视代理，可以安装在所有系统和容器上。

**NetData**是**免费的开源软件**，目前完美运行在**Linux**、**FreeBSD**和**MacOS**上。



Netdata是一个高度优化的Linux守护进程，它为Linux系统，应用程序，SNMP服务等提供实时的性能监测。

它用可视化的手段，将被监测者最细微的细节，展现了出来。这样，你便可以清晰地了解你的系统和应用程序此时的状况。





## 监测内容

> 功能真的是特别特别多
> 我主要是用来监测服务器状态和分析Nginx服务器的日志
> 默认配置，完全不需要自己配置
> **除非你需要更多功能或者取消部分功能**

1. CPU的使用率,中断，软中断和频率(总量和每个单核)
2. RAM，互换和内核内存的使用率（包括KSM和内核内存deduper）
3. 硬盘输入/输出(每个硬盘的带宽，操作，整理，利用等)
4. IPv4网络（数据包，错误，分片）：TCP：连接，数据包，错误，握手；UDP:  数据包，错误；广播：带宽，数据包；组播：带宽，数据包
5. Netfilter/iptables Linux防火墙(连接，连接跟踪事件，错误等)
6. 进程(运行，受阻，分叉，活动等)
7. NFS 文件服务器, v2, v3, v4 (I/O, 高速缓存, 预读, RPC 调用)
8. 网络服务质量（唯一一个可实时可视化网络状况的工具）
9. 应用程序，通过对进程树进行分组（CPU,内存，硬盘读取，硬盘写入，交换，线程，管道，套接字等）
10. Apache web 服务器 mod 状态 (v2.2, v2.4)
11. Nginx web 服务器状态
12. Mysql数据库（多台服务器，单个显示：带宽，查询/s, 处理者，锁，问题，临时操作，连接，二进制日志，线程，innodb引擎等）
13. ISC Bind域名服务器（多个服务器，单个显示：客户，请求，查询，更新，失败等）
14. Postfix邮件服务器的消息队列（条目，大小）
15. Squid代理服务器（客户带宽和请求，服务带宽和请求）
16. 硬件传感器（温度，电压，风扇，电源，湿度等）
17. NUT UPSes（负载，充电，电池电压，温度，使用指标，输出指标）





## 效果

![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/Netdata.gif)

![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/Netdata1.png)

![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/Netdata2.png)

## 安装

```
sudo apt-get install zlib1g-dev gcc make git autoconf autogen automake pkg-config
sudo git clone https://github.com/netdata/netdata.git --depth=1
sudo apt-get install uuid-dev
sudo cd netdata
sudo ./netdata-installer.sh
```

```python

  ^
  |.-.   .-.   .-.   .-.   .  netdata                                        
  |   '-'   '-'   '-'   '-'   real-time performance monitoring, done right!  
  +----+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+--->


  You are about to build and install netdata to your system.

  It will be installed at these locations:

   - the daemon     at /usr/sbin/netdata
   - config files   in /etc/netdata
   - web files      in /usr/share/netdata
   - plugins        in /usr/libexec/netdata
   - cache files    in /var/cache/netdata
   - db files       in /var/lib/netdata
   - log files      in /var/log/netdata
   - pid file       at /var/run/netdata.pid
   - logrotate file at /etc/logrotate.d/netdata

  This installer allows you to change the installation path.
  Press Control-C and run the same command with --help for help.

Press ENTER to build and install netdata to your system > 

```

![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/netdata4.png)



**回车进行编译安装**

静静等待安装完成

```python
 --- We are done! --- 

  ^
  |.-.   .-.   .-.   .-.   .-.   .  netdata                          .-.   .-
  |   '-'   '-'   '-'   '-'   '-'   is installed and running now!  -'   '-'  
  +----+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+--->

  enjoy real-time performance and health monitoring...
```

![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/netdata3.png)



**正常情况下安装成功，直接访问ip:19999就能使用了，如果不行手动开启**

```
sudo /usr/sbin/netdata
```



[**项目Github地址**](https://github.com/ktsaou/netdata)