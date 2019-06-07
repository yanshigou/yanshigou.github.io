---

title: "解决Mac下安装mysqlclient报错"
date: 2019-06-07 13:43
author: dzt
subtitle: 
tags:
  - Mac
  - mysql
  - Error
---


## OSError: mysql_config not found

查找mysql_config文件夹位置，我的在`/usr/local/opt/mysql@5.7/bin`这个路径下，解决方法：
 将mysql_config链接到`/usr/local/bin`目录下
 `ln -s /usr/local/opt/mysql@5.7/bin/mysql_config /usr/local/bin/mysql_config`
 再次执行 `pip3 install mysqlclient`，但是我又报出了另一个错误，如下



## error: command 'clang' failed with exit status 1

> 找了很多种解决方法，最后这种办法成功解决
>
> https://codelover.link/2019/05/09/mac-not-found-stdio/

```shell
cd /Library/Developer/CommandLineTools/Packages
open macOS_SDK_headers_for_macOS_xx.pgk
# macOS_SDK_headers_for_macOS_xx.pgk 对应自己的系统版本
```

跳出安装界面，安装完成！

再次执行 `pip3 install mysqlclient`

**Successfully installed mysqlclient-1.4.2**