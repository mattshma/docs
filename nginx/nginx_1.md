Nginx 学习
===

本文基本是边看官方文档边操作边记录下的形式写下来的。此时听着自己喜爱的音乐，没人打扰，窗外风景祥和。

Nginx架构
---
Nginx在启动后，会有一个master进程和多个worker进程，master进程用来管理worker进程，包括：接收来自外界的信号，向各worker进程发送信号，监控worker进程的运行状态等。一般worker进程用来处理事件，多个worker进程间是对等的。那么这些平等的worker进程是怎么处理请求的呢？首先，在master进程里面，先建立好listen的socket(listenfd)，然后fork出多个子worker进程，所有worker进程的listenfd会在新连接到来时变得可读，当一个请求过来时，所有worker进程在注册listenfd读事件抢accept_mutex，抢到互斥锁的那个进程注册listenfd读事件，在读事件里调用accept接受该连接。当这个worker进程在accept这个连接后，就开始读取请求，解析请求，处理请求，然后将结果返回给客户端，最后断开连接。

Nginx采用这种多worker方式来处理请求，一个worker对应一个请求，那何来高并发？Nginx不同于Apache（每个请求会独占一个工作绕中，并发数有多少，线程就有多少，这些线程占用大量内存，同时线程上下文的切换带来的CPU开销也很大），Nginx采用了异步非阻塞的方式来处理请求，也就是说Nginx是可以同时处理成千上万个请求的。

Start, Stop and reload
---

Nginx的`-s`参数会处理如下四种`signal`: 

- stop — fast shutdown  
- quit — graceful shutdown  
- reload — reloading the configuration file  
- reopen — reopening the log files  

以前改完配置后，一直都是relod的，对于为什么reload，只觉得是等以前老的nginx进程处理完请求后会读取新配置文件启动进程。今天看文档，有如下说明：

> Once the master process receives the signal to reload configuration, it checks the syntax validity of the new configuration file and tries to apply the configuration provided in it. If this is a success, the master process starts new worker processes and sends messages to old worker processes, requesting them to shut down. Otherwise, the master process rolls back the changes and continues to work with the old configuration. Old worker processes, receiving a command to shut down, stop accepting new connections and continue to service current requests until all such requests are serviced. After that, the old worker processes exit.

可以看出优点是有几个的。

还可以通过发送信号来控制Nginx，具体可以参考 [Controlling nginx](http://nginx.org/en/docs/control.html)。

Nginx 配置系统
---

在 Nginx 配置文件中包括若干配置项，每个配置项由配置指令和指令参数2个部分构成。当前 Nginx 支持几个指令上下文：

- main  
  nginx 在运行时与具体业务无关的一些参数，比如工作进程数等。
- http  
  与提供 http 服务相关的一些配置参数，如：是否使用keepalived，是否使用 gzip 压缩，access_log等。
- server  
  http 服务上支持的若干虚拟主机。一个虚拟主机对应一个 server 配置项， 配置项包含该虚拟主机的相关配置。
- location  
  http 服务中， 某些特定 URL 对应的一系列配置项。
- mail  
  关于 email 相关的一些配置项。

指令上下文可能出现包含的情况，比如 http上下文 一定出现在 main上下文里。一个上下文可能会多次包含另外一种类型的上下文，如 http上下文 可以包含多个 server上下文。

























