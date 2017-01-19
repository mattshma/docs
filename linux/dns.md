# dns

本文不对DNS做详细说明，只做一些笔记，具体内容可参见[鸟哥的Linux私房菜:DNS服务器](http://cn.linux.vbird.org/linux_server/0350dns.php)。

## 配置说明

### `/etc/named.conf`
如下：
```
options {
    listen-on port 53 { any; };
    directory     "/var/named";
    dump-file     "/var/named/data/cache_dump.db";
    statistics-file "/var/named/data/named_stats.txt";
    memstatistics-file "/var/named/data/named_mem_stats.txt";
    recursion yes;
    allow-query     { any; };      //指定客户端能对本DNS提出查询请求。
    allow-query-cache { any; };    
    allow-transfer { <slave_dns_ip>; };  //允许身从DNS传输数据。

    ; forward only;                // 本DNS仅做转发，此处注释。
    forwarders { <forwarder_dns_1>;<forwarder_dns_2>;<forwarder_dns_3>; };  //向上层DNS服务器转发。
    dnssec-enable yes;
    dnssec-validation yes;
    //dnssec-lookaside no;

    /* Path to ISC DLV key */
    bindkeys-file "/etc/named.iscdlv.key";

    managed-keys-directory "/var/named/dynamic";
};
```

### 正解文件记录格式

正解文件记录格式为：`<domain>	[TTL]	IN	[[RR type]	[RR data]]`。其中RR type有如下几种：

- A    
  对应主机名的IP。
- NS    
  查询管理zone的服务器主机名。
- MX   
  邮件服务器主机。
- AAAA    
  对应主机名的IPv6。
- CNAME    
  别名记录。
- PTR  
  ip对应的主机名。
- SOA     
  查询管理zone的服务器管理信息。SOA比较特殊，其会接七个参数，分别是：
  - Master DNS Server。   
  - Admin email。
  - Serial。代表这个数据库档案的新旧，序号越大则越新。以master上的序列值是否大于slave上的序列值，slave判断是否需要更新数据文件。
  - Refresh。slave多久向master请求数据更新。
  - Retry。slave重试时间。
  - Expire。若一直尝试失败，slave将不再更新，并删除自己的copy文件。此时需等待管理员处理。
  - Minimum。代表该zone文件中所有记录的TTL值，即其他DNS cache这些记录时，最长不超过这个时间。

正解文件中，记录一定要从行首开始，前面不可有空格，若有空格，则代表延续前一个domain。另外正解文件中，`@`代表该zone的意思，为简化每条记录的TTL设置，统一将TTL挪到文件第一行设置，用`$TTL`表示。一个正解文件中，至少要有`$TTL`，`SOA`和`NS`。另外，在正解文件中，还可以通过`$ORIGIN`来重新设置zone的定义。

以下是一个zone文件示例：
```
$TTL 86400
$ORIGIN yzdns.com.
@       IN      SOA     ns1.yzdns.com.  root.yzdns.com. (
                                        20160909131     ; serial
                                        2H      ; refresh
                                        15M     ; retry
                                        1W      ; expire
                                        3H )    ; minimum
        IN      NS      ns1.yzdns.com.
        IN      NS      ns2.yzdns.com.
ns1     IN      A       10.6.24.90
ns2     IN      A       10.6.28.98

bd15-001        IN      A       10.6.25.11
```

在设置完各配置文件后，可以通过`named-checkconf`检查BIND主配置文件的是否错误，`named-checkzone`检查zone配置文件是否错误。

## master/slave

如下是master的一个示例文件：
```
//反解
zone "10.in-addr.arpa" IN {
    type master;
    file "10.arpa";
    allow-update { none; };              
    notify yes;        //若主DNS更新，则立即通知从DNS
    also-notify { 10.6.28.98; };    // 指定通知哪些ip
    allow-transfer { 10.6.28.98; };      //指定从DNS ip
};

// 正解
zone "yzdns.com" IN {
    type master;
    file "yzdns.com.zone";
    allow-update { none; };
    notify yes;
    also-notify { 10.6.28.98; };
    allow-transfer { 10.6.28.98; };
};
```

slave配置文件示例：
```
zone "10.in-addr.arpa" IN {
        type slave;
        file "slaves/10.arpa";
        masters { 10.6.24.90; };           //主DNS ip
        allow-notify { 10.6.24.90; };      //允许notify的主机信息
};

zone "yzdns.com" IN {
        type slave;
        file "slaves/yzdns.com.zone";
        masters { 10.6.24.90; };           
        allow-notify { 10.6.24.90; };
};
```


## dig

反解：`dig -x <ip> @<dns_server>`。 


## 错误
slave同步master，查看slave的`/var/log/messages`，错误如下
```
Jan 19 11:38:18 bd15-127 named[7487]: transfer of 'yzdns.com/IN' from 10.6.24.90#53: connected using 10.6.28.98#55823
Jan 19 11:38:18 bd15-127 named[7487]: transfer of 'yzdns.com/IN' from 10.6.24.90#53: failed while receiving responses: REFUSED
Jan 19 11:38:18 bd15-127 named[7487]: transfer of 'yzdns.com/IN' from 10.6.24.90#53: Transfer completed: 0 messages, 0 records, 0 bytes, 0.001 secs (0 bytes/sec)
```

一般而言，出现这种问题，都是因为权限问题或配置文件错误，权限方面，确认`/etc/rndc.key`，`/etc/named*`，`/var/named/*`等为`named:named`，配置文件问题，可通过`named-checkconf`和`named-checkzone`检查。
