# Bonding配置

之前在Ubuntu下配置Bonding，问题不大，在Centos下配置Bonding时，尝试几次后才配置成功。现记录下：

## 过程

### 操作系统是否开启/支持bonding操作
操作如下：

```
# modinfo bonding                  ; bonding模块是否已编译到内核，centos一般默认编译到内核中了。
# modprobe bonding                 ; 若已编译，则加载bonding模块到内核中
# lsmod | grep bonding             ; 查看是否加载成功
# echo "modprobe bonding" > /etc/rc.local  ; 开机加载bonding模块
```

### 修改配置文件

- 新建`/etc/sysconfig/network-scripts/ifcfg-bond0`，配置如下：
```
DEVICE=bond0
BONDING_OPTS="mode=0 miimon=500"
BOOTPROTO=none
ONBOOT=yes
IPADDR=10.6.25.11
NETMASK=255.255.255.0
GATEWAY=10.6.25.1
USERCTL=no
```

- 修改`/etc/sysconfig/network-scripts/ifcfg-eth0`，配置如下：
```
DEVICE=eth0
TYPE=Ethernet
ONBOOT=yes
BOOTPROTO=none
MASTER=bond0
SLAVE=yes
```

- 修改`/etc/sysconfig/network-scripts/ifcfg-eth1`，配置如下：
```
DEVICE=eth1
TYPE=Ethernet
ONBOOT=yes
BOOTPROTO=none
MASTER=bond0
SLAVE=yes
```

### 绑定eth0和eth1到bonding池中

执行`ifenslave bond0 eth0 eth1 > /etc/rc.local`，向bonding池中加入实际网络接口设备eth0与eth１。

### 重启网络
执行`service network restart`重启网络后，通过`ethtool bond0`可查看bond0的带宽（`Speed`项）。


## 参数说明

- BOOTPROTO   
 使用哪种协议获取ip。`none`表示不使用协议，即手动获取ip；`bootp`表示使用BOOTP协议获取ip；`dhcp`表示使用DHCP协议获取ip。

- ONBOOT   
 开机是否启动。

- PEERDNS   
 启动该设备是否修改`/etc/resolv.conf`文件。

- USERCTL   
 非root用户是否可以控制该设备。


## 参考
- [Channel Bonding Interfaces](https://www.centos.org/docs/5/html/5.1/Deployment_Guide/s2-networkscripts-interfaces-chan.html)
- [Interface Configuration Files](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/3/html/Reference_Guide/s1-networkscripts-interfaces.html)
