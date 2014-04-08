Curl
===

以下是工作中可能会用到的curl相关知识

指定Host
---

在服务器上做完相关操作后，需要测试该服务器是否运行正常。此时可以通过绑定host来进行本地测试，更好的方法是使用curl的`-H`参数来指定host，如下：

    curl -H "Host: 10.10.3.100" http://someurl

通过观察该服务器上日志，判断是否有异常。

指定代理
---

在nginx做负载均衡的系统中，当修改完nginx的配置，需要测试修改的文件是否正确，可以使用curl的`-x`参数来测试下，如：

    curl -x nginxIP someurl

指定一个url，观察nginx上日志是否正常。


Reference
---

- [manpage](http://curl.haxx.se/docs/manpage.html)
