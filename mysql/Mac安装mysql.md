Mac安装Mysql
===

本来很简单的过程，但因配置文件冗余，一直安装失败，重新卸载安装即可。过程如下。

卸载之前的Mysql
---
若之前没安装过，这步可以跳过。删除目录如下：


```
sudo rm /usr/local/mysql   
sudo rm -rf /usr/local/mysql*
sudo rm -rf /Library/StartupItems/MySQLCOM
sudo rm -rf /Library/PreferencePanes/My*
```

若`/etc`目录存在mysql配置文件，需将其重命名或删除。

安装mysql
---

官网下载mysql安装，一路继续安装完成。编辑/usr/local/mysql/support-files/mysql.server，修改`basedir`和`datadir`，如下：

```
basedir=/usr/local/mysql
datadir=/usr/local/mysql/data
```
点击系统偏好设置，点击mysql，点击`Start Mysql Server`。若启动失败，查看`/usr/local/mysql/data/mysqld.local.err`找出错误原因。

设置root用户及密码
----
进入mysql设置root用户即可。步骤如下：

```
mysql -uroot -p
Enter password:  (这里直接回车即可)
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 33
Server version: 5.6.26 MySQL Community Server (GPL)

Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> UPDATE mysql.user SET Password=Password('root') WHERE User='root';
mysql> FLUSH PRIVILEGES;
mysql> \q
```

root用户密码设置完成。