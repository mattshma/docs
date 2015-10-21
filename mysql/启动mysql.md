
对于mysql的启动与关闭，查看[官方文档](MySQL Server and Server-Startup Programs)，这里做个笔记。

## Startup

一般而言，在Linux环境下，有如下启动方式:   

### mysqld

[mysqld](https://dev.mysql.com/doc/refman/5.6/en/mysqld.html)是mysql的守护进程，一般这种方式启动失败的话，错误信息只会在终端输出，而不会记录在错误日志文件中，所以当mysql崩溃时，将会无法查找原因，所以这种方式一般只在调试时使用。

### mysqld_safe

[mysqld_safe](https://dev.mysql.com/doc/refman/5.6/en/mysqld-safe.html)是一个脚本，其调用mysqld，并添加了一些安全特性，如在发生错误其会自动重启mysql，并将错误日志输出到错误日志中等。因此在unix系统上，推荐使用启动mysqld。

### mysql.server

[mysql.server](https://dev.mysql.com/doc/refman/5.6/en/mysql-server.html)也是一个脚本，其会调用mysqld_safe。使用mysql.server来启动mysql的命令:`shell> mysql.server start`。当运行mysql.server时，其先切换到MySQL安装目录，然后再调用mysqld_safe，所以或是通过源码将MySQL安装在非标准位置的话，还需要编辑mysql.server这个脚本，将其安装目录修改到正确位置，然后再调用mysqld_safe。

### mysqld_multi

如果一台服务器上需要运行多个mysql实例的话，使用[mysqld_multi](https://dev.mysql.com/doc/refman/5.6/en/mysqld-multi.html)方式。在my.cnf中，通过指定不同的GNR(group number)来分不同的组，如[mysqld **N** ]，若这些组有相同的配置，可以放在[mysqld]组中。启动某组实例的语法如:`shell> mysqld_multi [options] {start|stop|reload|report} [GNR[,GNR] ...]`。


   启动方式   |  读取配置组
--------------|----------------
mysqld        | [mysqld], [server], [mysqld-*major_version*]
mysqld_safe   | [mysqld], [server], [mysqld_safe]
mysql.server  | [mysqld], [server], [mysql.server]
mysqld_multi  | [mysqld], [mysqld**N**]

其中[mysqld-*major_version*]即[mysqld-5.5]或[mysqld-5.6]等等，该配置仅在对应版本中才会被读取。

为保持兼容性，`mysql.server`同时会读取[mysql_server]组，mysqld_safe同时会读取[safe_mysqld]组，但在配置文件中，应尽可能的使用[mysql.server]和[mysqld_safe]来代替。

## Shutdown

- mysql.server stop
- mysqldadmin shutdown


参考
---
- [https://dev.mysql.com/doc/refman/5.6/en/mysqld.html](https://dev.mysql.com/doc/refman/5.6/en/mysqld.html)
- [Starting and Stopping MySQL Automatically](https://dev.mysql.com/doc/refman/5.6/en/automatic-start.html)
- [option-files](https://dev.mysql.com/doc/refman/5.6/en/option-files.html)
