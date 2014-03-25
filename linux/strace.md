strace
===

strace 常用来跟踪进程执行时的系统调用和所接收到的信号。但在实际中，几乎很少人使用这个强大的工具。strace可以但远不限于使用在如下方面

- 调试程序。
- 确定花费最多时间和资源的代码片断。
- 查看正在运行程序的情况

strace 参数简述
---

- -c  
统计每个系统调用所花的时间、调用次数、错误，并以汇总的形式输出出来。

- -f  
跟踪当前进程fork出来的子进程(`-F`属于被淘汰的参数)。

- -T  
显示每个系统调用所花的时间。（所花时间显示在`<>`里）

- -t   
显示系统调用时的当前时间，精确到秒，在每条系统调用的最前面显示。`-tt`精确到微秒。

- -e expr  
具体格式如下：

        [qualifier=][!]value1[,value2]...  
    
  其中`qualifier`的值是 **trace**, **abbrev**, **verbose**, **raw**, **signal**, **read**, 或 **write** 中的一个，默认值为`trace`，如`-e open`相当于`-e trace=open`，value是一个指定的符号或者数字。 **!** 表示取反。

- -e trace=set  
跟踪指定的系统调用，如`trace=open,close,read,write`表示只跟踪这四个系统调用，默认情况下set的值为all，即`trace=all`。当指定系统调用结合`-c`参数时，可能找出程序的瓶颈。

- -e trace=file  
跟踪那些将文件作为参数的系统调用。

- -o  
将输出结果重定向。

- -p  
指定进程pid。

Usages
---

- 找出配置文件
```c
strace -e open php 2>&1 | grep php.ini
```
- strace -p pid
 有时候进程会hang住，可以通过`strace -p pid`查看该进程在哪里hang住的。当遇到问题时多使用"strace -p"能省去很多猜想的工作，有时也可以省掉重启app来使用扩展和重编译扩展的工作。

- 为什么不能连接到对方？
 当DNS挂掉，连接hang住时，可以使用tcptump来进行抓包。也可以使用strace只返回这个程序的结果。

List Of Reference
---
- [man-pages of STRACE](http://man7.org/linux/man-pages/man1/strace.1.html)
- [5 simple ways to troubleshoot using strace](http://www.hokstad.com/5-simple-ways-to-troubleshoot-using-strace)

