Location解释
===

这里说下nginx对location的处理。nginx wiki中对此顺序的解释如下：
> The order in which location directives are checked is as follows:
>
> 1. Directives with the "=" prefix that match the query exactly (literal string). If found, searching stops.
> 2. All remaining directives with conventional strings. If this match used the "^~" prefix, searching stops.
> 3. Regular expressions, in the order they are defined in the configuration file.
> 4. If #3 yielded a match, that result is used. Otherwise, the match from #2 is used.

nginx首先检查`literal strings`（普通匹配），如果`literal strings`是以`=`或者`^~`进行匹配的，则不会再进行`regular expressions`(正则匹配)，否则继续进行正则匹配。如果正则匹配成功，则返回该匹配，否则返回`literal strings`的最大匹配结果。

这里有两点需要说明下：

1. `^~`仍满足最大匹配。  
2. `literal strings`与配置文件的顺序无关（与最大匹配有关），`regular expressions`与配置文件的顺序有关。

Reference
---
- [nginx wiki](http://wiki.nginx.org/NginxHttpCoreModule#location)
- [Nginx 关于 location 的匹配规则详解](http://eyesmore.iteye.com/blog/1141660)
