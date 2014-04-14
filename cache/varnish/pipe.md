Varnish Pipe
===

How-to
---

在某些情况中可能需要直接使用pipe，如请求的内容大于10M等时。以下是一个示例：

```c
sub vcl_recv {
    if (req.http.x-pipe && req.restarts > 0) {
        unset req.http.x-pipe;
        return (pipe);
    }
    //other codes..
}  

sub vcl_fetch {
    if (beresp.http.Content-Length ~ "[0-9]{8,}") {
        set req.http.x-pipe = "1";
        return (restart);
    }
    //other codes..
    return (deliver);
}

sub vcl_pipe {
    set bereq.http.connection = "close";
}
```

如果对于`no-cache`之类的请求，需要使用pipe，可以在`vcl_recv`中直接返回`pipe`
Reference
---

- [VCLExamplePipe](https://www.varnish-cache.org/trac/wiki/VCLExamplePipe)
- [using-pipe-varnish](https://www.varnish-software.com/blog/using-pipe-varnish)
- [varnish variables](https://www.varnish-cache.org/docs/3.0/reference/vcl.html#variables)
