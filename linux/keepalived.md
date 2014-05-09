Keepalived
===

Install
---

解压Keepalived后，开始安装

```c
# ./configure
# make; make install
```

安装结束之后，copy文件

```c
# cp /usr/local/etc/rc.d/init.d/keepalived /etc/init.d/
# cp /usr/local/etc/sysconfig/keepalived /etc/sysconfig/
# cp /usr/local/sbin/keepalived /sbin/
# mkdir /etc/keepalived
# cp /usr/local/etc/keepalived/keepalived.conf /etc/keepalived/
```

keepalived.conf
---

Keepalived的配置可以分为三类：  

- 全局配置
- VRRP定义
- LVS定义

其中LVS只在使用keepalived来配置和管理LVS时需要使用，如果仅用keepalived做HA，LVS是不需要的。

Master的keepalived.conf如下：

```
global_defs {
   notification_email {
   	beitian@gmail.com
   }
   notification_email_from hatest.noreply1@dm.anjuke.com
   smtp_server 10.10.6.209
   smtp_connect_timeout 30
   router_id LVS_DEVEL
}

vrrp_sync_group VG1 {
	group {
		VI_1
	}
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        10.10.3.168
    }
}

virtual_server 10.10.3.168 80 {
    delay_loop 6
    lb_algo rr
    lb_kind NAT
    nat_mask 255.255.255.0
    persistence_timeout 50
    protocol TCP

    real_server 10.10.3.233 80 {
        notify_down /root/script/restart_keepalived.sh
        weight 1
        TCP_CHECK {
            connect_timeout 3
        }
    }
}
```

backup的keepalived.conf一样，除了以下几项修改外：

```
state BACKUP
priority 50
...
real_server 10.10.3.180 80 {
...
}
```

配置好之后，使用`service keepalived start`启动keepalived，通过`ifconfig`看不到VIP，可通过`ip a`看到VIP.当启动失败时，可查看`/var/log/message`中具体信息。

这里有几点注意需要：  
- 设置virtual_ipaddress之前，保证虚拟IP没有其他人使用  
- keepalived根据`priority`决定主从，而不是`state`  
- keepalived配置中所有的`{`前至少要有一个空格
- 路径均需为绝对路径（包括shell脚本里面的内容）

当权重高的机器挂掉之后，VIP漂到权重低的机器上，但当权重高的机器恢复正常后，VIP又漂回来了。为防止这种情况发生，可以在权重高的机器的配置文件中加`nopreempt`，但`nopreempt`只能在state为BACKUP的机器，所以还得将`state MASTER`修改为`state BACKUP`, 如下

```
...
vrrp_instance VI_1 {
    state BACKUP
    nopreempt
    ...  # others
}
```

正如上所说，keepalive根据`priority`决定主从。

说明一下，当`TCP CHECK`检测到real_server失败后，开始执行`notify_down`指明的脚本，此时应该让keepalived重启或关掉，让VIP漂移到BACKUP那边。以下是`restart_keepalived.sh`的内容:

```
#! /bin/bash
/etc/init.d/keepalived restart
/etc/init.d/nginx restart
```

在vrrp_instance name{}中，还可以用`notify_master`、`notify_backup`和`notify`来指定运行的脚本。

其中`notify`有三个参数，分别如下：  

- $1 = “GROUP” or “INSTANCE”
- $2 = name of group or instance
- $3 = target state of transition (“MASTER”, “BACKUP”, “FAULT”)

关于`mcast_src_ip`的一些说明：`mcast_src_ip`指定发送vrrp广播的源地址，所以这里就需要注意，如果高权重机器上指定`mcast_src_ip`为低权重机器IP，那么两个机器都将有VIP，除了低权重机器认为VIP在本机这里外，其他机器都会认为VIP为高权重机器。另外，如果设置`nopreempt`之后，若低权重机器指定`mcast_src_ip`为高权重机器，则相当于没有设置`nopreempt`，高权重机器会自动抢占成为MASTER。

vrrp\_script and track\_script
---
除了使用`notify`来指定运行脚本外，还可以使用`vrrp_script`来运行脚本。如下：

```c
vrrp_script taskname {
   script /path/to/taskshell
   interval 10  # check every 10 seconds
   weight -40   # if failed, decrease 40 of the priority
   fall   2     # require 2 failures for failures
   rise   1     # require 1 sucesses for ok
}

vrrp_instance VI_1 {
    ...
    track_script {
        taskname
    }
}
```

`track_script`是在keepalived启动之后开始执行，如需成为master时执行脚本，可使用`notify_master`。根据`vrrp_script`中的配置，可以在检测失败时减掉权重（weight -40的作用）让其低于BACKUP的权重，VIP随之漂到BACKUP那边去，当然也可检测成功时加权重让之高于MASTER。另外需注意，`taskshell`脚本使用的路径需为绝对路径，否则即使`/var/log/messages`中提示`Keepalived_vrrp[16931]: VRRP_Script(taskname) succeeded`，脚本可能也不会执行成功。

PS:这里有对权重的说明：[keepalived-vrrp-script](http://ialloc.org/2012/keealived-vrrp_script/)

关于其他具体原理及语法，此不赘述。
