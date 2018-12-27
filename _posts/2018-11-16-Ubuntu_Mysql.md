---
title: "最近使用ubuntu服务器遇到的事儿"
date: 2018-11-16 11:06
author: dzt
subtitle: 服务器部署，mysql配置，......
tags:
  - ubuntu
  - mysql
  - study
---



# ubuntu vim

ubuntu默认没有安装vim，出现：

root@apple20181115:~ vim vim /etc/mysql/my.cnf

The program 'vim' can be found in the following packages:

* vim
* vim-gnome
* vim-tiny
* vim-athena
* vim-gtk
* vim-nox

    Try: sudo apt-get install <selected package>

解决：


命令行输入：sudo apt-get install vim　　然后安装，出现确认的时候，输入y确认就行



配置：如果想更强大，请移步<http://www.cnblogs.com/ma6174/archive/2011/12/10/2283393.html>



ubuntu12.04中使用的vim的版本不支持像语法高亮和文件类型检测等配置



#sudo apt-get install vim



vim默认的配置使用起来还不能让人满意，还需要自己配置

默认配置文件是：/etc/vim/vimrc

我们可以在家目录下建立自己的配置文件

切换到家目录 #cd ~

touch一个名为.vimrc的文件（以.开头的为隐藏文件）

#vi .vimrc

输入以下配置：

set nocompatible       不使用vi默认键盘布局   

set number               显示行号

set autoindent          自动对齐

set smartindent         智能对齐

set showmatch          括号匹配模式

set ruler                　显示状态行

set incsearch            查询时非常方便，如要查找book单词，当输入到/b时，会自动找到   第一个b开头的单词，当输入到/bo时，会自动找到第一个bo开头的单词，依次类推，进行查找时，使用此设置会快速找到答案，当你找要匹配的单词时，别忘记回车.

set tabstop=4           tab键为4个空格

set shiftwidth=4    　换行时行间交错使用4个空格

set softtabstop=4　 设置（软）制表符宽度为4

set cindent              C语言格式对齐

set nobackup            不要备份文件

set clipboard+=unnamed   与windows共享剪贴板









# ubuntu 服务器部署



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



使用fab pack   fab deploy  将配置好的uwsgi.ini  nginx.conf 和需要搭建的项目，打包上传和部署



18. 建立软连接 ln -s /root/www/test/test_nginx.conf   /etc/nginx/sites-enabled/

19. sudo service nginx start







\# mysql



\### 修改密码



1、安全模式登入MySQL



\```

$ sudo /etc/init.d/mysql stop

\-------------------------------------

[sudo] wl 的密码：

[ ok ] Stopping mysql (via systemctl): mysql.service.

$ sudo /usr/bin/mysqld_safe --skip-grant-tables --skip-networking &

\```



输入第一行终止MySQL运行，成功，会提示下面两行；输入第四行，成功，没有任何报错则可以另外打开一个终端窗口进行下一步操作；但是一般会报错，比如提示`mysqld_safe Directory ‘/var/run/mysqld’ for UNIX socket file don’t exists`



因此我们尝试输入以下代码



\```

$ sudo mkdir -p /var/run/mysqld

$ sudo chown mysql:mysql /var/run/mysqld

\```



最后再次输入：



\```

sudo /usr/bin/mysqld_safe ``--skip-grant-tables --skip-networking &

\```



到了这里不在提示错误，可以打开另一个终端端口了，尝试无密码登入MySQL。



\```

mysql -u root

\```



到这里应该可以进入MySQL了，继续操作



\```

\> use mysql;

\> update user set authentication_string=PASSWORD("这里输入你要改的密码") where User = 'root'; #更改密码

\> update user set plugin= "mysql_native_password"; #如果没这一行可能也会报一个错误，因此需要运行这一行

\> flush  privileges ; #更新所有操作权限

\> quit;

\```



2、使用修改的密码登入MySQL



经过上面一系列的操作，应该可以正常使用你更改的密码登入了。



\```

\> sudo /etc/init.d/mysql stop

\>

\> sudo /etc/init.d/mysql start # reset mysql

\> mysql -u root -p

\```



第一行先终止数据库运行，第二行重启数据库服务，第三行root用户登入。







\# 远程权限



\### 开放3306端口



首先确认3306端口是否对外开放，mysql默认状态下是不开放对外访问功能的。查看方法如下：



\```

\# netstat -an | grep 3306

tcp    0   0 127.0.0.1:3306      0.0.0.0:*         LISTEN

\```



从上面可以看出，mysql的3306端口只是监听本地连接`127.0.0.1`。我们做下修改，使其对外其他地址开放。

打开`/etc/mysql/my.cnf`文件



\```

\# vim /etc/mysql/my.cnf

\```



找到`bind-address = 127.0.0.1`这一行，大概在47行，我们将它注释掉。









![img](https:////upload-images.jianshu.io/upload_images/31920-1427a7891cea11f7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/591/format/webp)







\###  授权用户远程访问



为了让访问mysql的客户端的用户有访问权限，我们可以通过如下方式为用户进行授权：

首先进入mysql



\```

\# mysql -uroot -pyour_password

\```



授权：



\```

mysql> grant all on *.* to user_name@'%' identified by 'user_password';

\```



\### 重启mysql服务，使配置生效



\```

\# /etc/init.d/mysql restart

\```



\# 远程db没有mysql库，添加root密码



1. 关闭mysql



   `service mysqld stop`



2. 屏蔽权限



\```

\# mysqld_safe --skip-grant-table

or

\# /usr/bin/mysqld_safe --skip-grant-table

\```



3. 新开终端



   \```

   mysql -u root mysql

   UPDATE user SET Password=PASSWORD('newpassword') where USER='root';//重置root密码

    

   delete from user where USER='';//删除空用户

    

   FLUSH PRIVILEGES;//记得要这句话，否则如果关闭先前的终端，又会出现原来的错误

   exit

   

   \```



4. `service mysqld restart`