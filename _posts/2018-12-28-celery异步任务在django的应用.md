---
title: "nginx+uwsgi+django+celery"
date: 2018-12-28 12:55
author: dzt
subtitle: celery分布式任务队列在服务器上的使用
tags:
  - django
  - celery
  - nginx
  - uwsgi
---

## Celery - 分布式任务队列

Celery 是一个简单、灵活且可靠的，处理大量消息的分布式系统，并且提供维护这样一个系统的必需工具。

它是一个专注于实时处理的任务队列，同时也支持任务调度
> [源码在我的github上](https://github.com/yanshigou/MyScripts/tree/master/test_celery) ---> <https://github.com/yanshigou/MyScripts/tree/master/test_celery>




## 前言

在web应用中，用户触发一个操作，执行后台处理程序，这个程序需要执行很长时间才能返回结果。怎样才能不阻塞http请求，不让用户等待从而提高用户体验呢？这是本例需要解决的问题。

我会从服务器的配置开始



## ubuntu服务器的部署 （nginx+uwsgi+django) 

export LC_ALL=C
LC_ALL=C 是为了去除所有本地化的设置，让命令能正确执行。

1. sudo apt-get update
2. sudo apt-get install python-pip
3. pip install virtualenv  (安装后可选 virtualenvwrapper   使用workon更方便)

最基本的已部署完毕----> 可选配置好django  mysql
(   pip install django==1.11
  sudo apt-get install mysql-server
  sudo apt-get mysql-client
  sudo apt-get install libmysqlclient-dev
  sudo apt-get install python-dev
  pip install mysqlclient
)
4. mkdir www/test
5. virtualenv --no-site-packages kkwork
6. source kkwork/bin/activate  进入虚拟环境
7. pip install django==1.11
8. pip install djangorestframework
9. pip install markdown
10. pip install django-filter
11. pip install django-cors-headers
12. sudo apt-get install libmysqlclient-dev
13. pip install pymysql
14. pip install mysql-python
15. sudo apt-get install nginx
16. pip install uwsgi
17. pip install requests
18. pip install django-supervisor
    python manage.py collectstatic  配置静态文件    比如admin的css js
    使用fab pack   fab deploy  将配置好的uwsgi.ini  nginx.conf 和需要搭建的项目，打包上传和部署
19. 建立软连接 ln -s /root/www/test/test_nginx.conf   /etc/nginx/sites-enabled/  
    如果创建软连接权限不够   前面加sudo
20. sudo service nginx start





#### start_celery.sh

**写好shell命令 source 需要用bash来执行**
```sh
cd /home/ubuntu/www/cmxcelery/
source kkwork/bin/activate
celery worker -A test_celery -l info
```



#### 配置supervisor    

> 官方文档 <http://supervisord.org/configuration.html#program-x-section-settings>

\# 生成supervisor配置文件 

echo_supervisord_conf > /etc/supervisord.conf



\# 如果权限不够

echo_supervisord_conf > supervisord.conf

mv supervisord.conf  /etc/



\# 创建日志目录

mkdir /var/log/celery



\# 列出需要更改的基本配置  其他均为默认配置

\# 配置内容  ; 为注释


```ini
[unix_http_server]

# 可以修改为/var/run/supervisor.sock  在tmp下可能会清除缓存

file=/tmp/supervisor.sock ; (the path to the socket file)

chmod=0700 ; socket file mode (default 0700)



[supervisord]

logfile=/var/log/celery/celery.log ;

# 可以修改为/var/run/supervisord.pid  在tmp下可能会清除缓存

pidfile=/tmp/supervisord.pid



[supervisorctl]

# 可以修改为///var/run/supervisor.sock  需与file对应

serverurl=unix:///tmp/supervisor.sock 





# program: 后为名称

[program:celery]  

# celery命令的绝对路径

command=bash /home/ubuntu/www/cmxcelery/start_celery.sh

# 项目路径

directory=/home/ubuntu/www/cmxcelery/

# 日志文件路径

stdout_logfile=/var/log/celery/celery.log

# 自动重启

autorestart=true

# 如果设置为true,进程则会把标准错误输出到supervisord后台的标准输出文件描述符

redirect_stderr=true
```



这是我的supervisord.conf

```ini
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
file=/var/tmp/supervisor.sock   ; (the path to the socket file)
chmod=0777                 ; socket file mode (default 0700)
chown=nobody:nogroup       ; socket file uid:gid owner
#username=yinxuan              ; (default is no username (open server))
#password=yinxuan               ; (default is no password (open server))

;[inet_http_server]         ; inet (TCP) server disabled by default
;port=127.0.0.1:9001        ; (ip_address:port specifier, *:port for all iface)
;username=user              ; (default is no username (open server))
;password=123               ; (default is no password (open server))

[supervisord]
logfile=/var/log/celery/celery.log ; (main log file;default $CWD/supervisord.log)
logfile_maxbytes=50MB        ; (max main logfile bytes b4 rotation;default 50MB)
logfile_backups=10           ; (num of main logfile rotation backups;default 10)
loglevel=info                ; (log level;default info; others: debug,warn,trace)
pidfile=/var/tmp/supervisord.pid ; (supervisord pidfile;default supervisord.pid)
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
serverurl=unix:///tmp/supervisor.sock ; use a unix:// URL  for a unix socket
;serverurl=http://127.0.0.1:9001 ; use an http:// url to specify an inet socket
;username=chris              ; should be same as http_username if set
;password=123                ; should be same as http_password if set
;prompt=mysupervisor         ; cmd line prompt (default "supervisor")
;history_file=~/.sc_history  ; use readline history if available

; The below sample program section shows all possible program subsection values,
; create one or more 'real' program: sections to be able to control them under
; supervisor.

[program:celery]
command=bash /home/ubuntu/www/cmxcelery/start_celery.sh              ; the program (relative uses PATH, can take args)
;process_name=%(program_name)s ; process_name expr (default %(program_name)s)
;numprocs=1                    ; number of processes copies to start (def 1)
directory=/home/ubuntu/www/cmxcelery/                ; directory to cwd to before exec (def no cwd)
;umask=022                     ; umask for process (default None)
;priority=999                  ; the relative start priority (default 999)
;autostart=true                ; start at supervisord start (default: true)
autostart=true ;如果设置为true，当supervisord启动的时候，进程会自动重启。
;startsecs=1                   ; # of secs prog must stay up to be running (def. 1)
;startretries=3                ; max # of serial start failures when starting (default 3)
;autorestart=unexpected        ; when to restart if exited after running (def: unexpected)
autorestart=true   ; 程序崩溃时自动重启，重启次数是有限制的，默认为3次
;exitcodes=0,2                 ; 'expected' exit codes used with autorestart (default 0,2)
startsecs = 5        ; 启动 5 秒后没有异常退出，就当作已经正常启动了    
;stopsignal=QUIT               ; signal used to kill process (default TERM)
;stopwaitsecs=10               ; max num secs to wait b4 SIGKILL (default 10)
;stopasgroup=false             ; send stop signal to the UNIX process group (default false)
;killasgroup=false             ; SIGKILL the UNIX process group (def false)
;user=chrism                   ; setuid to this UNIX account to run the program
;redirect_stderr=true          ; redirect proc stderr to stdout (default false)
;stdout_logfile=/a/path        ; stdout log path, NONE for none; default AUTO
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
redirect_stderr=true        ; 错误日志重定向到标准输出
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

;[include]
;files = relative/directory/*.ini

[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface
```



#### 使用 supervisorctl 管理进程

```shell
# 停止某一个进程，program_name 为 [program:x] 里的 x：
supervisorctl stop program_name


# 启动某个进程：
supervisorctl start program_name


# 重启某个进程：
supervisorctl restart program_name


# 停止全部进程，注：start、restart、stop 都不会载入最新的配置文件：
supervisorctl stop all


# 载入最新的配置文件，停止原有进程并按新的配置启动、管理所有进程：
supervisorctl reload


# 根据最新的配置文件，启动新配置或有改动的进程，配置没有改动的进程不会受影响而重启：
supervisorctl update
```
