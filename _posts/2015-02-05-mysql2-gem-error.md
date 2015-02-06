---
layout: post
title: failed to build gem native extension 
description: 修改database.yml文件配置mysql2，运行bundle时出项的错误问题解决
category: 问题解决
tags: [ruby gem]
---
{% include JB/setup %}
#Failed to build gem native extension 
 


#写在前面：
**系统环境**：windows7 32bit, ruby 2.0.0, rails 4.0.0, mysql 6.0

#错误及原因

**错误**：直接命令安装`gem install mysql2` 或者`bundle install`安装mysql2 gem时，错误如下：
<!--break-->
![mysql2_gem_error](http://hihera.qiniudn.com/mysql2_gem_error.png)

然后查看Ruby安装路径下的错误信息文件mkmf.log：

    E:\RailsInstaller\Ruby2.0.0\lib\ruby\gems\2.0.0\gems\mysql2-0.3.17\ext\mysql2\mk.mf.log

发现有如下错误信息：

    e:/railsinstaller/devkit/mingw/bin/../lib/gcc/mingw32/4.5.2/../../../../mingw32/bin/ld.exe: cannot find -lmysqlclient

**原因**：
没有安装 `MYSQL C-connector library`，缺少mysql连接库驱动。

#解决方法：
1、下载 [MYSQL C-Connector Library](http://dev.mysql.com/downloads/connector/c/) ZIP 版本

![mysql_connector](http://hihera.qiniudn.com/mysql_connector.png)

2、解压下载的文件夹到某个路径，注意路径不能有**空格**，例如：

    C:\mysql-connector-c-6.1.5-win32

3、执行gem安装命令

    gem install mysql2 --platform=ruby -- '--with-mysql-lib="C:\mysql-connector-c-6.1.5-win32\lib" --with-mysql-include="C:\mysql-connector-c-6.1.5-win32\include" --with-mysql-dir="C:\mysql-connector-c-6.1.5-win32"'

安装完成后出现如下界面：

![mysql2_gem_success](http://hihera.qiniudn.com/mysql2_cmd_success.jpg)

4、安装完成后去ruby安装目录bin下查看是否出现`libmysql.dll`，如图:

![libmysql](http://hihera.qiniudn.com/libmysql.png)

如果`E:\RailsInstaller\Ruby2.0.0\bin`路径没有出现`libmysql.dll`文件，那么手动将 `C:\mysql-connector-c-6.1.5-win32\lib`路径下的`libmysql.dll`文件复制到上边路径即可。

#其它环境下错误解决方案：
参考：[Stack Overflow](http://stackoverflow.com/questions/3608287/error-installing-mysql2-failed-to-build-gem-native-extension)

On Ubuntu/Debian and other distributions using aptitude:

    sudo apt-get install libmysql-ruby libmysqlclient-dev


If the above command doesn't work because libmysql-ruby cannot be found, the following should be sufficient:

    sudo apt-get install libmysqlclient-dev


On Red Hat/CentOS and other distributions using yum:

    sudo yum install mysql-devel


On Mac OS X with Homebrew:

    brew install mysql








