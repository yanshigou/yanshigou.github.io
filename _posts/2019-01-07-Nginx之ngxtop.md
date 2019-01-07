---
title: "Nginx之ngxtop的使用"
date: 2019-01-07 14:45
author: dzt
subtitle: ngxtop实时解析nginx访问日志，并且将处理结果输出到终端
tags:
  - nginx
  - study
  - python
---

## 前言

之前的nginx的日志都是我每天下载下来，再用我自己写的python脚本分析：

* 统计nginx服务器上的access.log日志，输出结果如下：

> [23/Oct/2018:06:25:01] 至 [23/Oct/2018:20:24:21]  
> 单秒最大访问量:298  
> nginx访问ip个数:9095  
> nginx一共访问量:126555  
> 统计时间: 2018-10-24 13:57:09.287000  

* 统计nginx服务器上的error.log日志，输出结果如下:

> connect() to unix:///home/ubuntu/www/cmxsite/cmxsite.sock failed (111: Connection refused) while connecting to upstream,  
> directory index of "/home/ubuntu/www/html/home/" is forbidden,  
> rewrite or internal redirection cycle while processing "/index.php",  
> connect() to unix:///home/ubuntu/www/eleccard/eleccard.sock failed (111: Connection refused) while connecting to upstream,  
>
> 06:56:21 至 20:08:17  
> nginx error出现ip个数:87   
> nginx error类型个数:4   
> nginx error次数:98  
> 统计时间: 2018-10-24 17:16:36.199000  

发现了ngxtop之后，就发现简单分log就很舒服了，但是如果需要分析error.log和详细分析access.log的话，还是需要自己写脚本



### 有了ngxtop，实时解析nginx访问日志，并且将处理结果输出到终端

> Note: `ngxtop` is primarily developed and tested with python2 but also supports python3.
>
> 主要是用python2开发和测试的，也支持python3



**使用pip进行安装**

* pip install ngxtop



### ngxtop使用详解

ngxtop 参数 print|top|avg|sum
ngxtop info 显示日志格式信息

- -l <file>或--access-log <file> 设置日志路径
- -f <format>或--log-format <format> 设置日志格式，默认格式combined，另外一种较常用格式为common
- --no-follow 处理以前的日志，实时日志不做处理
- -t <seconds> 或 --interval <seconds> 刷新频率，默认2秒
- -g <var>或 --group-by <var> 按变量分组，默认显示 request_path
- -w <var>或 --having <expr> 筛选 [default: 1]
- -o <var>或 --order-by <var> 输出的排序方式，默认: 访问数
- -n <number>或 --limit <number> 显示top多条，默认前top 10条
- -a <exp> ...或 --a <exp> ... 对输出字段做处理，可选 sum, avg, min, max
- -v或 --verbose 详细输出
- -d或 --debug debug模式，输出每行及记录
- -h或 --help 显示帮助详细
- --version 显示版本信息

高级参数

- -c <file>或 --config <file> 指定nginx配置文件，自动分析日志格式
- -i <filter-expression>或 --filter <filter-expression> 满足表达式的过滤将被处理
- -p <filter-expression>或 --pre-filter <filter-expression> in-filter expression to check in pre-parsing phase.



**另外一些变量可以在分析时用到，名字含义同日志格式里的设置：**

**remote_addr、remote_user、time_local、request、request_path、status、body_bytes_sent、http_referer、http_user_agent。**



```
(kkwork) ubuntu@VM-0-13-ubuntu:~/www/cmxcelery$ ngxtop --help
ngxtop - ad-hoc query for nginx access log.

Usage:
    ngxtop [options]
    ngxtop [options] (print|top|avg|sum) <var> ...
    ngxtop info
    ngxtop [options] query <query> ...

Options:
    -l <file>, --access-log <file>  # 需要分析的访问日志.
    -f <format>, --log-format <format> # log format 指令指定的日志格式. [默认: combined]
    --no-follow  ngxtop default behavior is to ignore current lines in log
                     and only watch for new lines as they are written to the access log.
                     Use this flag to tell ngxtop to process the current content of the access log instead.
    -t <seconds>, --interval <seconds>  report interval when running in follow mode [default: 2.0]

    -g <var>, --group-by <var>  # 根据变量分组 [默认: request_path]
    -w <var>, --having <expr>  having clause [default: 1]
    -o <var>, --order-by <var>  # 排序 [默认:: count]
    -n <number>, --limit <number>  # 显示的条数 [default: 10]
    -a <exp> ..., --a <exp> ...  add exp (must be aggregation exp: sum, avg, min, max, etc.) into output

    -v, --verbose  # 更多的输出
    -d, --debug  print every line and parsed record
    -h, --help  # 当前帮助信息.
    --version  # 输出版本信息.

    Advanced / experimental options:
    -c <file>, --config <file>  # 运行ngxtop解析nginx配置文件
    -i <filter-expression>, --filter <filter-expression>  filter in, records satisfied given expression are processed.
    -p <filter-expression>, --pre-filter <filter-expression> in-filter expression to check in pre-parsing phase.

Examples:
    All examples read nginx config file for access log location and format.
    If you want to specify the access log file and / or log format, use the -f and -a options.

    "top" like view of nginx requests
    $ ngxtop
	
    Top 10 requested path with status 404:  # 404前十的请求
    $ ngxtop top request_path --filter 'status == 404' 
	
    Top 10 requests with highest total bytes sent # 总流量前十的请求
    $ ngxtop --order-by 'avg(bytes_sent) * count'

    Top 10 remote address, e.g., who's hitting you the most # 访问量前十的ip地址
    $ ngxtop --group-by remote_addr

    Print requests with 4xx or 5xx status, together with status and http referer
    # 输出400以上状态码的请求以及请求来源
    $ ngxtop -i 'status >= 400' print request status http_referer

    Average body bytes sent of 200 responses of requested path begin with 'foo':
    $ ngxtop avg bytes_sent --filter 'status == 200 and request_path.startswith("foo")'

    Analyze apache access log from remote machine using 'common' log format
    # 使用common日志格式分析远程服务器Apache访问日志
    $ ssh remote tail -f /var/log/apache2/access.log | ngxtop -f common
```



### ngxtop使用实例

**1、分析log日志**

```
$ ngxtop -l /var/log/nginx/access.log

running for 54 seconds, 0 records processed: 0.00 req/sec

Summary:
|   count |   avg_bytes_sent |   2xx |   3xx |   4xx |   5xx |
|---------+------------------+-------+-------+-------+-------|
|    1068 |        28026.763 |  1029 |    20 |    19 |     0 |

Detailed:
| request_path           |   count |   avg_bytes_sent |   2xx |   3xx |   4xx |   5xx |
|------------------------+---------+------------------+-------+-------+-------+-------|
| /xxxxxxxxxx            |     199 |        55150.402 |   199 |     0 |     0 |     0 |
| /xxxxxxxx/xxxxx        |     167 |        47591.826 |   167 |     0 |     0 |     0 |
| /xxxxxxxxxxxxx/xxxxxx  |      25 |         7432.200 |    25 |     0 |     0 |     0 |
| /xxxx/xxxxx/x/xx/xx    |      22 |          698.727 |    22 |     0 |     0 |     0 |
| /xxxx/xxxxx/x/xxx/xxxx |      19 |         7431.632 |    19 |     0 |     0 |     0 |
| /xxxxx/xxxxx/          |      18 |         7840.889 |    18 |     0 |     0 |     0 |
| /xxxxxxxx/xxxxxxxxxx   |      15 |         7356.000 |    15 |     0 |     0 |     0 |
| /xxxxxxxxxxx/xxxxx     |      15 |         9978.800 |    15 |     0 |     0 |     0 |
| /xxxxx/                |      14 |            0.000 |     0 |    14 |     0 |     0 |
| /xxxxxxxxxx/xxxxxx/xx  |      13 |        20530.154 |    13 |     0 |     0 |     0 |
```



**2、查看访问前10的IP地址**

```
$ ngxtop top remote_addr

running for 12 seconds, 0 records processed: 0.00 req/sec

top remote_addr
| remote_addr   | count   |
| 125.26.135.219|      15 |
| 125.26.213.203|      15 |
|---------------+---------|

```



**3、查看状态码为xxx的请求 == or >=**

```
$ ngxtop -i 'status == 400' print request status http_referer

running for 8 seconds, 0 records processed: 0.00 req/sec

request, status, http_referer:
| request   | status   | http_referer   |
| -         |      400 | -              |
|-----------+----------+----------------|

```



**4、使用通用格式从远程服务器解析log日志**

可能会出现权限不足的错误

```shell
$ ssh user@remote_server tail -f /var/log/apache2/access.log | ngxtop -f common
running for 20 seconds, 1068 records processed: 53.01 req/sec

Summary:
|   count |   avg_bytes_sent |   2xx |   3xx |   4xx |   5xx |
|---------+------------------+-------+-------+-------+-------|
|    1068 |        28026.763 |  1029 |    20 |    19 |     0 |

Detailed:
| request_path           |   count |   avg_bytes_sent |   2xx |   3xx |   4xx |   5xx |
|------------------------+---------+------------------+-------+-------+-------+-------|
| /xxxxxxxxxx            |     199 |        55150.402 |   199 |     0 |     0 |     0 |
| /xxxxxxxx/xxxxx        |     167 |        47591.826 |   167 |     0 |     0 |     0 |
| /xxxxxxxxxxxxx/xxxxxx  |      25 |         7432.200 |    25 |     0 |     0 |     0 |
| /xxxx/xxxxx/x/xx/xx    |      22 |          698.727 |    22 |     0 |     0 |     0 |
| /xxxx/xxxxx/x/xxx/xxxx |      19 |         7431.632 |    19 |     0 |     0 |     0 |
| /xxxxx/xxxxx/          |      18 |         7840.889 |    18 |     0 |     0 |     0 |
| /xxxxxxxx/xxxxxxxxxx   |      15 |         7356.000 |    15 |     0 |     0 |     0 |
| /xxxxxxxxxxx/xxxxx     |      15 |         9978.800 |    15 |     0 |     0 |     0 |
| /xxxxx/                |      14 |            0.000 |     0 |    14 |     0 |     0 |
| /xxxxxxxxxx/xxxxxx/xx  |      13 |        20530.154 |    13 |     0 |     0 |     0 |
```



**5、查看当前access.log数据（前四条数据都是实时的，不统计之前的数据）**

```
$ ngxtop --no-follow
running for 0 seconds, 30 records processed: 17173.35 req/sec

Summary:
|   count |   avg_bytes_sent |   2xx |   3xx |   4xx |   5xx |
|---------+------------------+-------+-------+-------+-------|
|      30 |        35356.167 |    15 |     1 |    10 |     4 |

Detailed:
| request_path           |   count |   avg_bytes_sent |   2xx |   3xx |   4xx |   5xx |
|------------------------+---------+------------------+-------+-------+-------+-------|
| /                      |      12 |          786.667 |     8 |     0 |     1 |     3 |
| /results/              |       5 |         2471.000 |     5 |     0 |     0 |     0 |
| /favicon.ico           |       4 |          110.000 |     0 |     0 |     4 |     0 |
|                        |       3 |           60.667 |     0 |     0 |     3 |     0 |
| /cache/global/img/gs.gif|       2 |          104.000 |     0 |     0 |     1 |     1 |
| /nginx-status          |       1 |          111.000 |     0 |     0 |     1 |     0 |
| /pao/                  |       1 |          637.000 |     1 |     0 |     0 |     0 |
| /results/upload/download/paoliuliang_2019-01-05pao.xls  |       1 |            0.000 |     0 |     1 |     0 |     0 |
| /results/upload/download/paoliuliang_2019-01-05pao.xls/ |       1 |      1037312.000 |     1 |     0 |     0 |     0 |
```



**ngxtop -l /var/log/apache2/access.log --no-follow** 可以加一下参数进行详细分析，
下面几个例子

按rquest_path且是404的前10请求：

> **ngxtop -l /var/log/apache2/access.log --no-follow top request_path --filter 'status == 404'**

按总bytes sent最高的前10：

> **ngxtop -l /var/log/apache2/access.log --no-follow --order-by 'avg(bytes_sent) \* count'**

按remote address进行排序前10：

> **ngxtop -l /var/log/apache2/access.log --no-follow --group-by remote_addr**

显示400或更高返回状态码的且只显示request、status、http_referer这三列信息：

> **ngxtop -l /var/log/apache2/access.log --no-follow -i 'status >= 400' print request status http_referer**

显示bytes_sent平均值且状态码为200且request_path以vpser开始的前10：

> **ngxtop -l /var/log/apache2/access.log --no-follow avg bytes_sent --filter 'status == 200 and request_path.startswith("vpser")'**