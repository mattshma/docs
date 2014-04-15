Twemproxy Test Report
===

以下是对twemproxy的测试报告

准备配置
---

- twemproxy: 使用两个twemproxy实例,配置情况如下：

```
listen: 0.0.0.0:22121
hash: fnv1a_64
distribution: ketama
auto_eject_hosts: true
server_retry_timeout: 30000
server_failure_limit: 3
timeout: 1000
servers:
    - 192.168.188.33:11211:1
    - 192.168.188.33:11212:1
    - 10.10.3.180:11211:1
    - 10.10.3.180:11212:1
```
另一个twemproxy监听端口为22122。启动这两个twemproxy实例和4个memcahed实例。通过twemproxy向每个memcached中写数据。

相关的php测试代码如下：

```php
<?php
 $memcache = new Memcache;
 $servers = array(
                  array('host'=>'localhost', 'port'=>"22122", 'persistent'=>true),
                  array('host'=>'localhost', 'port'=>"22121", 'persistent'=>true)
                  );
 foreach($servers as $server) {
   $memcache->addServer($server['host'], $server['port']);
 }
 
 $local1 = $memcache->get("aaaa");
 $local2 = $memcache->get("hhhh");
 $remote1 = $memcache->get("xxxx");
 $remote2 = $memcache->get("ssss");
 print "local1 : " . $local1 . "\nlocal2 : " . $local2 . "\nremote1 : " . $remote1 . "\nremote2 : " . $remote2 . "\n";
?>
```

一个twemproxy实例挂掉数据是否丢失？
---
测试使用一致性hash时，当一个twemproxy实例挂掉，是否会丢失数据。

```
maming@lucky:php$ ps axuf |grep nut
maming   18484  0.0  0.0  17936   816 ?        Sl   Apr09   0:00 src/nutcracker -d -c conf/m2.yml -s 22223
maming   17960  0.0  0.0  13624   944 pts/10   S+   09:32   0:00      \_ grep --color=auto nut
maming   17302  0.0  0.0  17936   896 ?        Sl   09:06   0:00 src/nutcracker -d -c conf/m1.yml -s 22222
maming@lucky:php$ kill 17302
maming@lucky:php$ php testnut.php 
PHP Notice:  MemcachePool::get(): Server localhost (tcp 22121, udp 0) failed with: Connection refused (111) in /home/maming/program/php/testnut.php on line 14
local1 : aaaa
local2 : hhhh
remote1 : xxxx
remote2 : ssss
...
maming@lucky:twemproxy$ src/nutcracker -d -c conf/m1.yml -s 22222
...
maming@lucky:php$ kill 18484
maming@lucky:php$ php testnut.php 
PHP Notice:  MemcachePool::get(): Server localhost (tcp 22122, udp 0) failed with: Connection refused (111) in /home/maming/program/php/testnut.php on line 12
local1 : aaaa
local2 : hhhh
remote1 : xxxx
remote2 : ssss
```
因此使用一致性hash时，当一个twemproxy挂掉后，数据不会丢失。

后端mc挂掉
---
后端挂掉一个mc，经过指定的timeout和server\_failure\_limit后，将会自动摘除该故障节点，但故障节点恢复后，还是会读取其数据，需写脚本清除数据。

实例增删改对hash的影响
---
- add instances






注意：对于之前已有数据的memcached，再通过twemproxy取key时，会有key的丢失（hash时找不到key）。因此对于已有数据的memcached使用twemproxy前，需对twemproxy预热数据。
