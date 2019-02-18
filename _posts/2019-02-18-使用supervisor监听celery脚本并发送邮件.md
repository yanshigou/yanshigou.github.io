---

title: "使用supervisor管理celery脚本并发送邮件"
date: 2019-02-18 10:10
author: dzt
subtitle:  独立的celery定时任务，不依赖于Django
tags:
  - supervisor
  - python
  - celery
---

## 前言

为了监测nginx服务器，我使用了很多个方法(之前的文章都有写到)，例如Netdata实时监测，ngxtop分析log(也可以实时监测)，但我想在nginx服务器挂掉的时候立马给我发邮件，通知我及时重新启动。

所以我写了一个脚本，定时去访问以下某个链接，看是否挂掉或者其他问题（这个方法特别的愚蠢）

但总不能一直启动脚本吧

所以，就使用了celery的定时任务来执行，这次是完全独立的，不依赖于Django了



[源码在我的github上](https://github.com/yanshigou/period) ----> <https://github.com/yanshigou/period>



## 目录结构



![](https://raw.githubusercontent.com/yanshigou/yanshigou.github.io/master/img/t/period.png)


## 介绍

这次BROKER试用的是Redis

以下是celery的设置

**celeryconf.py**

```python
# -*- coding: utf-8 -*-
from celery.schedules import crontab


BROKER_URL = "redis://localhost:6379/1"
# CELERY_RESULT_BACKEND = 'redis://localhost:6379/2'

CELERY_TIMEZONE = 'Asia/Shanghai'
CELERY_ENABLE_UTC = False

# 导入指定的任务模块
CELERY_IMPORTS = ('celery_app.tasks',)


# 下面是定时任务的设置
CELERYBEAT_SCHEDULE = {
    # 定时任务:　每1分钟，执行任务(listen_ngx_celery)
    'Listen_ngx': {
        'task': 'celery_app.tasks.listen_ngx_celery',
        'schedule': crontab(minute='*/2'),
        "args": ()
    },
}
```



**__init__.py**

```python
# -*- coding: utf-8 -*-
from celery import Celery


app = Celery("listen_ngx")
app.config_from_object('celery_app.celeryconf')  # 加载配置模块

```



任务，目的是访问链接判断其状态，如有问题则发邮件通知

**taks.py**

```python
# -*- coding: utf-8 -*-
from __future__ import absolute_import
from celery import shared_task
from datetime import datetime
from email_send import sendMail, listen_nginx


@shared_task(track_started=True)
def listen_ngx_celery():
    try:
        res = listen_nginx()
        if res.status_code != 200:
            print('not 200', datetime.now(), res)
            error_info = u'Nginx服务器似乎出了问题，\r\n状态码{0}，\r\n{1}，\r\n请立即查看。'.format(
                res.status_code, res.json
            )
            sendMail(error_info)

        else:
            print("It's ok", datetime.now())
    except Exception as e:
        print('Exception!!', datetime.now(), e)
        error_info = u'Nginx服务器似乎出了问题，\r\n{0}，\r\n请立即查看'.format(e)
        sendMail(error_info)

```



**email_send.py**

```python
# -*- coding: utf-8 -*-
__author__ = 'dzt'
__date__ = '2019/02/12 16:13'

import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
from email.header import Header
# from email import encoders
# from email.mime.base import MIMEBase
from email.utils import parseaddr, formataddr
from datetime import datetime
import requests


# 格式化邮件地址
def formatAddr(s):
    name, addr = parseaddr(s)
    return formataddr((Header(name, 'utf-8').encode(), addr))


def sendMail(body, attachment=None):
    smtp_server = 'smtp.qq.com'
    from_mail = 'yanshigou@foxmail.com'
    mail_pass = '授权码非密码'
    to_mail = ['xx@cmx-iot.com', 'xx@cmx-iot.com']  # 列表多个
    # 构造一个MIMEMultipart对象代表邮件本身
    msg = MIMEMultipart()
    # Header对中文进行转码
    msg['From'] = formatAddr('yanshigou<%s>' % from_mail).encode()
    msg['To'] = ','.join(to_mail)
    msg['Subject'] = Header('监听nginx情况{0}'.format(datetime.now()), 'utf-8').encode()
    # plain代表纯文本
    msg.attach(MIMEText(body, 'plain', 'utf-8'))
    # 二进制方式模式文件
    # with open(attachment, 'rb') as f:
    #     # MIMEBase表示附件的对象
    #     mime = MIMEBase('text', 'txt', filename=attachment)
    #     # filename是显示附件名字
    #     mime.add_header('Content-Disposition', 'attachment', filename=attachment)
    #     # 获取附件内容
    #     mime.set_payload(f.read())
    #     encoders.encode_base64(mime)
    #     # 作为附件添加到邮件
    #     msg.attach(mime)
    server = smtplib.SMTP_SSL(smtp_server, 465)
    server.set_debuglevel(1)
    server.login(from_mail, mail_pass)
    # print('kaishi')
    # print(type(msg))
    server.sendmail(from_mail, to_mail, msg.as_string())  # as_string()把MIMEText对象变成str
    server.quit()


def listen_nginx():
    url = 'http://xxx:8000/usermanager/checkLogin/?username=15922504420'
    res = requests.get(url=url)
    # print(res.status_code)
    # print(type(res.status_code))
    return res


if __name__ == "__main__":
    sendMail('附件是测试数据, 请查收！')
```



## supervisor配置

echo_supervisord_conf > /etc/supervisord.conf 生成默认配置

改动如下

**supervisord.conf**

```python
; Sample supervisor config file.
;
; For more information on the config file, please see:
; http://supervisord.org/configuration.html
;
; Notes:
;  - Shell expansion ("~" or "$HOME") is not supported.  Environment
;    variables can be expanded using this syntax: "%(ENV_HOME)s".
;  - Comments must have a leading space: "a=b ;comment" not "a=b;comment".

[unix_http_server]
file=/var/run/supervisor.sock   ; (the path to the socket file)
chmod=0700                 ; socket file mode (default 0700)
;chown=nobody:nogroup       ; socket file uid:gid owner
;username=user              ; (default is no username (open server))
;password=123               ; (default is no password (open server))

;[inet_http_server]         ; inet (TCP) server disabled by default
;port=127.0.0.1:9001        ; (ip_address:port specifier, *:port for all iface)
;username=user              ; (default is no username (open server))
;password=123               ; (default is no password (open server))

[supervisord]
logfile=/var/log/celery/supervisord.log ; (main log file;default $CWD/supervisord.log)
logfile_maxbytes=50MB        ; (max main logfile bytes b4 rotation;default 50MB)
logfile_backups=10           ; (num of main logfile rotation backups;default 10)
loglevel=info                ; (log level;default info; others: debug,warn,trace)
pidfile=/var/run/supervisord.pid ; (supervisord pidfile;default supervisord.pid)
nodaemon=false               ; (start in foreground if true;default false)
minfds=1024                  ; (min. avail startup file descriptors;default 1024)
minprocs=200                 ; (min. avail process descriptors;default 200)
;umask=022                   ; (process file creation umask;default 022)
;user=chrism                 ; (default is current user, required if root)
;identifier=supervisor       ; (supervisord identifier, default is 'supervisor')
;directory=/tmp              ; (default is not to cd during start)
;nocleanup=true              ; (don't clean up tempfiles at start;default false)
;childlogdir=/tmp            ; ('AUTO' child log dir, default $TEMP)
;environment=KEY="value"     ; (key value pairs to add to environment)
;strip_ansi=false            ; (strip ansi escape codes in logs; def. false)

; the below section must remain in the config file for RPC
; (supervisorctl/web interface) to work, additional interfaces may be
; added by defining them in separate rpcinterface: sections
[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[supervisorctl]
serverurl=unix:///var/run/supervisor.sock ; use a unix:// URL  for a unix socket
;serverurl=http://127.0.0.1:9001 ; use an http:// url to specify an inet socket
;username=chris              ; should be same as http_username if set
;password=123                ; should be same as http_password if set
;prompt=mysupervisor         ; cmd line prompt (default "supervisor")
;history_file=~/.sc_history  ; use readline history if available

; The below sample program section shows all possible program subsection values,
; create one or more 'real' program: sections to be able to control them under
; supervisor.

;[program:paoliuliang]
;command=bash /home/ubuntu/www/cmxcelery/start_celery.sh              ; the program (relative uses PATH, can take args)
;process_name=%(program_name)s ; process_name expr (default %(program_name)s)
;numprocs=1                    ; number of processes copies to start (def 1)
;directory=/home/ubuntu/www/cmxcelery/                ; directory to cwd to before exec (def no cwd)
;umask=022                     ; umask for process (default None)
;priority=999                  ; the relative start priority (default 999)
;autostart=true                ; start at supervisord start (default: true)
;startsecs=5                   ; # of secs prog must stay up to be running (def. 1)
;startretries=3                ; max # of serial start failures when starting (default 3)
;autorestart=unexpected        ; when to restart if exited after running (def: unexpected)
;autorestart=true        ; when to restart if exited after running (def: unexpected)
;exitcodes=0,2                 ; 'expected' exit codes used with autorestart (default 0,2)
;stopsignal=QUIT               ; signal used to kill process (default TERM)
;stopwaitsecs=10               ; max num secs to wait b4 SIGKILL (default 10)
;stopasgroup=false             ; send stop signal to the UNIX process group (default false)
;killasgroup=false             ; SIGKILL the UNIX process group (def false)
;user=chrism                   ; setuid to this UNIX account to run the program
;redirect_stderr=true          ; redirect proc stderr to stdout (default false)
;stdout_logfile=/var/log/celery/celery.log        ; stdout log path, NONE for none; default AUTO
;stdout_logfile_maxbytes=1MB   ; max # logfile bytes b4 rotation (default 50MB)
;stdout_logfile_backups=10     ; # of stdout logfile backups (default 10)
;stdout_capture_maxbytes=1MB   ; number of bytes in 'capturemode' (default 0)
;stdout_events_enabled=false   ; emit events on stdout writes (default false)
;stderr_logfile=/a/path        ; stderr log path, NONE for none; default AUTO
;stderr_logfile_maxbytes=1MB   ; max # logfile bytes b4 rotation (default 50MB)
;stderr_logfile_backups=10     ; # of stderr logfile backups (default 10)
;stderr_capture_maxbytes=1MB   ; number of bytes in 'capturemode' (default 0)
;stderr_events_enabled=false   ; emit events on stderr writes (default false)
;environment=A="1",B="2"       ; process environment additions (def no adds)
;serverurl=AUTO                ; override serverurl computation (childutils)

; The below sample eventlistener section shows all possible
; eventlistener subsection values, create one or more 'real'
; eventlistener: sections to be able to handle event notifications
; sent by supervisor.

;[eventlistener:theeventlistenername]
;command=/bin/eventlistener    ; the program (relative uses PATH, can take args)
;process_name=%(program_name)s ; process_name expr (default %(program_name)s)
;numprocs=1                    ; number of processes copies to start (def 1)
;events=EVENT                  ; event notif. types to subscribe to (req'd)
;buffer_size=10                ; event buffer queue size (default 10)
;directory=/tmp                ; directory to cwd to before exec (def no cwd)
;umask=022                     ; umask for process (default None)
;priority=-1                   ; the relative start priority (default -1)
;autostart=true                ; start at supervisord start (default: true)
;startsecs=1                   ; # of secs prog must stay up to be running (def. 1)
;startretries=3                ; max # of serial start failures when starting (default 3)
;autorestart=unexpected        ; autorestart if exited after running (def: unexpected)
;exitcodes=0,2                 ; 'expected' exit codes used with autorestart (default 0,2)
;stopsignal=QUIT               ; signal used to kill process (default TERM)
;stopwaitsecs=10               ; max num secs to wait b4 SIGKILL (default 10)
;stopasgroup=false             ; send stop signal to the UNIX process group (default false)
;killasgroup=false             ; SIGKILL the UNIX process group (def false)
;user=chrism                   ; setuid to this UNIX account to run the program
;redirect_stderr=false         ; redirect_stderr=true is not allowed for eventlisteners
;stdout_logfile=/a/path        ; stdout log path, NONE for none; default AUTO
;stdout_logfile_maxbytes=1MB   ; max # logfile bytes b4 rotation (default 50MB)
;stdout_logfile_backups=10     ; # of stdout logfile backups (default 10)
;stdout_events_enabled=false   ; emit events on stdout writes (default false)
;stderr_logfile=/a/path        ; stderr log path, NONE for none; default AUTO
;stderr_logfile_maxbytes=1MB   ; max # logfile bytes b4 rotation (default 50MB)
;stderr_logfile_backups=10     ; # of stderr logfile backups (default 10)
;stderr_events_enabled=false   ; emit events on stderr writes (default false)
;environment=A="1",B="2"       ; process environment additions
;serverurl=AUTO                ; override serverurl computation (childutils)

; The below sample group section shows all possible group values,
; create one or more 'real' group: sections to create "heterogeneous"
; process groups.

;[group:thegroupname]
;programs=progname1,progname2  ; each refers to 'x' in [program:x] definitions
;priority=999                  ; the relative start priority (default 999)

; The [include] section can just contain the "files" setting.  This
; setting can list multiple files (separated by whitespace or
; newlines).  It can also contain wildcards.  The filenames are
; interpreted as relative to this file.  Included files *cannot*
; include files themselves.

[include]
files = /root/www/period/period.ini

```



新建一个period.ini，写入需要管理的脚本信息

**period.ini**

```ini
[program:listen_nginx]
command=bash /root/www/period/start_period.sh              ; the program (relative uses PATH, can take args)
directory=/root/www/period/                ; directory to cwd to before exec (def no cwd)
autostart=true                ; start at supervisord start (default: true)
startsecs=5                   ; # of secs prog must stay up to be running (def. 1)
autorestart=true        ; when to restart if exited after running (def: unexpected)
redirect_stderr=true          ; redirect proc stderr to stdout (default false)
stdout_logfile=/var/log/celery/listen_access.log        ; stdout log path, NONE for none; default AUTO
stdout_logfile_backups=10     ; # of stdout logfile backups (default 10)
stderr_logfile=/var/log/celery/listen_error.log         ; stderr log path, NONE for none; default AUTO
```



shell文件

**start_period.sh**

```sh
cd /root/www/period/
source kkwork/bin/activate
celery -B -A celery_app worker -l info
```



## 部署

1. 使用xftp把所有文件上传到服务器。
2. virtualenv –no-site-packages kkwork
3. pip install supervisor
4. source kkwork/bin/activate 进入虚拟环境
5. pip install -r requirement.txt

基本环境就搭建好了，接下来就是启动

1. supervisord -c supervisord.conf
2. supervisorclt status 查看状态是否正常 RUNNING

