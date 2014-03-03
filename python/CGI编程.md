CGI(Common Gateway Interface)
===

什么是CGI？
---
在Web开发过程中，如果需要客户端和服务端进行数据交换，如查询信息或者向服务器传递参数，单纯的HTML是无法做到这点的，这时候就需要使用CGI了。CGI是一段运行在服务器端的程序，它提供了服务器端同客户端HTML交互的接口，该程序块是实时执行的，因此能够动态输出信息。

HTTP协议简谈
---
当访问一个网址时，即客户端向服务器发送一个 **请求**，当服务器接收到请求后开始处理，处理完后返回 **响应**。请求与响应都是由“首部”（headers）与“实体”（entity）组成。首部是0组或者多组的键值对，每组HTTP首部结尾都需要使用 CRLF（Linux/Unix为`\n`，Mac中为`\r`，Windows中为`\r\n`）来代表该组结束。当首部部分结束的时候，还需要一个 CRLF 来表示整个首部结束，即使首部部分为空。在首部的后面可能会跟上实体，实体是实际上报文要传输的东西，比如说我们向服务器传输的文件，服务器向我们返回的页面，都是在实体之中的。如下首部：
```html
           Accept   */*\r\n
  Accept-Encoding   gzip, deflate\r\n
    Cache-Control	max-age=0\r\n
       User-Agent	Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:25.0) Gecko/20100101 Firefox/25.0\r\n
       \r\n
```

服务器配置
---
如果使用Apache或者Nginx做为服务器，还需要配置服务器。其中Nginx配置CGI的过程，可以参考[https://nginx.localdomain.pl/wiki/FcgiWrap](Simple CGI support for Nginx (fcgiwrap))。此处使用Python写CGI程序，其自带CGIHTTPServer，不需要配置。默认情况下CGI程序放在`mydirectory/cgi-bin`中，新建`cgi-bin`目录，权限为775。

CGI测试
---
假设有一个index.hmtl文件，如下：

```HTML
<form action="/cgi-bin/mycgi.py" method="get"> 
Input something: <input type="text" name="mycgi">
<input type="submit" value="Submit" />
</form>
```

在cgi-bin目录下，新建文件mycgi.py，权限为775。

    #!/usr/bin/python

    import cgi

    form = cgi.FieldStorage()  # 实例化 FieldStorage 
    mycgi = form.getvalue('mycgi')
    print "\r"
    print "<html><head><meta charset='utf-8'/></head><body><h2>happy to see %s </h2></body></html>" % mycgi

在`mydirectory`目录下，运行`python -m CGIHTTPServer`,在浏览器中输入`localhost:8000`，即可使用。

CGI的不足
---
当客户端发送给服务器一个动态请求的时候，服务器端会folk出一个新进程来运行外部的CGI代码（如上的python脚本）。这些代码会把处理完的数据返回给服务器，服务器再将这些数据响应给客户端，并退出刚才的folk进程。每次动态请求过来，都会执行上述过程，当请求过多的时候，会造成CGI处理效率低下。现在改进的方法是使用FastCGI。

------
List of Reference:
- [python cgi programming](http://www.tutorialspoint.com/python/python_cgi_programming.htm)
- [cgi — Common Gateway Interface support](http://docs.python.org/2/library/cgi.html)
