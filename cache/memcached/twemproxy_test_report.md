Twemproxy Test Report
===

以下是对twemproxy的测试报告

php使用的hash和twemproxy使用的hash是否相同？
---
出现miss.故之后的测试twemproxy都没有使用hash了。

一个twemproxy实例挂掉数据是否丢失？
---
当一个twemproxy实例挂掉，是否会丢失数据。

```
$ php testnut.php 
PHP Notice:  MemcachePool::get(): Server localhost (tcp 22122, udp 0) failed with: Connection refused (111) in /home/maming/program/php/testnut.php on line 26
local1 : 70979ha1-0
local2 : 13OU7AZJP709
remote1 : aaaaaa
remote2 : hhhh
```

因此使用一致性hash时，当一个twemproxy挂掉后，数据不会丢失。

后端mc挂掉
---
后端挂掉一个mc，经过指定的timeout和server\_failure\_limit后，将会自动摘除该故障节点，但故障节点恢复后，还是会读取其数据，需写脚本清除数据。

摘除后其他数据也会无法读取？？？

实例增删改对hash的影响
---
- 期望为1/n

性能测试
---

一个memcached和一个twemproxy对比
```
$ src/mcperf --num-calls=10 --num-conns=20000 --sizes=u1,16 
[Fri Apr 18 18:26:02 2014] mcp_conn_generator.c:64 created 0 20000 of 20000 connections
[Fri Apr 18 18:26:02 2014] mcp_conn_generator.c:94 destroyed 20000 of 20000 of 20000 connections

Total: connections 20000 requests 200000 responses 200000 test-duration 3.532 s

Connection rate: 5662.1 conn/s (0.2 ms/conn <= 1 concurrent connections)
Connection time [ms]: avg 0.2 min 0.2 max 5.8 stddev 0.08
Connect time [ms]: avg 0.0 min 0.0 max 0.3 stddev 0.00

Request rate: 56621.0 req/s (0.0 ms/req)
Request size [B]: avg 35.9 min 28.0 max 44.0 stddev 4.78

Response rate: 56621.0 rsp/s (0.0 ms/rsp)
Response size [B]: avg 8.0 min 8.0 max 8.0 stddev 0.00
Response time [ms]: avg 0.0 min 0.0 max 5.5 stddev 0.00
Response time [ms]: p25 1.0 p50 1.0 p75 1.0
Response time [ms]: p95 1.0 p99 1.0 p999 1.0
Response type: stored 200000 not_stored 0 exists 0 not_found 0
Response type: num 0 deleted 0 end 0 value 0
Response type: error 0 client_error 0 server_error 0

Errors: total 0 client-timo 0 socket-timo 0 connrefused 0 connreset 0
Errors: fd-unavail 0 ftab-full 0 addrunavail 0 other 0

CPU time [s]: user 1.04 system 2.49 (user 29.4% system 70.4% total 99.9%)
Net I/O: bytes 8.4 MB rate 2429.6 KB/s (19.9*10^6 bps)


$ src/mcperf --num-calls=10 --num-conns=20000 --sizes=u1,16 -p 22121
[Fri Apr 18 18:27:17 2014] mcp_conn_generator.c:64 created 0 20000 of 20000 connections
[Fri Apr 18 18:27:17 2014] mcp_conn_generator.c:94 destroyed 20000 of 20000 of 20000 connections

Total: connections 20000 requests 200000 responses 200000 test-duration 8.799 s

Connection rate: 2273.0 conn/s (0.4 ms/conn <= 1 concurrent connections)
Connection time [ms]: avg 0.4 min 0.4 max 19.2 stddev 0.27
Connect time [ms]: avg 0.0 min 0.0 max 0.2 stddev 0.00

Request rate: 22729.5 req/s (0.0 ms/req)
Request size [B]: avg 35.9 min 28.0 max 44.0 stddev 4.78

Response rate: 22729.5 rsp/s (0.0 ms/rsp)
Response size [B]: avg 8.0 min 8.0 max 8.0 stddev 0.00
Response time [ms]: avg 0.0 min 0.0 max 8.3 stddev 0.00
Response time [ms]: p25 1.0 p50 1.0 p75 1.0
Response time [ms]: p95 1.0 p99 1.0 p999 1.0
Response type: stored 200000 not_stored 0 exists 0 not_found 0
Response type: num 0 deleted 0 end 0 value 0
Response type: error 0 client_error 0 server_error 0

Errors: total 0 client-timo 0 socket-timo 0 connrefused 0 connreset 0
Errors: fd-unavail 0 ftab-full 0 addrunavail 0 other 0

CPU time [s]: user 2.52 system 6.26 (user 28.7% system 71.1% total 99.8%)
Net I/O: bytes 8.4 MB rate 975.3 KB/s (8.0*10^6 bps)


$ src/mcperf --num-calls=10 --num-conns=20000 --sizes=u1,16 -p 22121 -p 22122 -p 22123 -p 22124 -p 22125 -p 22126
[Fri Apr 18 18:29:34 2014] mcp_conn_generator.c:64 created 0 20000 of 20000 connections
[Fri Apr 18 18:29:34 2014] mcp_conn_generator.c:94 destroyed 20000 of 20000 of 20000 connections

Total: connections 20000 requests 200000 responses 200000 test-duration 8.262 s

Connection rate: 2420.9 conn/s (0.4 ms/conn <= 1 concurrent connections)
Connection time [ms]: avg 0.4 min 0.4 max 6.5 stddev 0.14
Connect time [ms]: avg 0.0 min 0.0 max 0.2 stddev 0.00

Request rate: 24208.6 req/s (0.0 ms/req)
Request size [B]: avg 35.9 min 28.0 max 44.0 stddev 4.78

Response rate: 24208.6 rsp/s (0.0 ms/rsp)
Response size [B]: avg 8.0 min 8.0 max 8.0 stddev 0.00
Response time [ms]: avg 0.0 min 0.0 max 5.8 stddev 0.00
Response time [ms]: p25 1.0 p50 1.0 p75 1.0
Response time [ms]: p95 1.0 p99 1.0 p999 1.0
Response type: stored 200000 not_stored 0 exists 0 not_found 0
Response type: num 0 deleted 0 end 0 value 0
Response type: error 0 client_error 0 server_error 0

Errors: total 0 client-timo 0 socket-timo 0 connrefused 0 connreset 0
Errors: fd-unavail 0 ftab-full 0 addrunavail 0 other 0

CPU time [s]: user 2.49 system 5.76 (user 30.2% system 69.7% total 99.9%)
Net I/O: bytes 8.4 MB rate 1038.8 KB/s (8.5*10^6 bps)
```



注意：对于之前已有数据的memcached，再通过twemproxy取key时，会有key的丢失（hash时找不到key）。因此对于已有数据的memcached使用twemproxy前，需对twemproxy预热数据。

