Some Grammers
===

以下是在使用Python过程中遇到的一些问题，将之整理如下

Encode and Decode
---
Python有两种类型的字符串类型：字节字符串和Unicode字符串。Python根据电脑默认的locale设置将字节流转化成字符，而大部分系统locale设置为ACSII编码。

若想改变Python编码格式为其他编码格式，如UTF-8，有如下3种方法。

### sys.setdefaultencoding('utf-8')
第一种方法是改变系统编码，可以使用`sys.getdefaultencoding()`来获取系统的默认编码。通过以下方法来修改系统编码：

```
import sys
reload(sys)
sys.setdefaultencoding('utf-8')
```

**为什么在reload(sys)**
> 为什么要reload sys模块,先看下python的模块加载过程:
>
> ```python
> $ python -v
> # installing zipimport hook
> import zipimport # builtin
> # installed zipimport hook
> # /usr/local/lib/python2.7/site.pyc matches /usr/local/lib/python2.6/site.py
> import site # precompiled from /usr/local/lib/python2.6/site.pyc
> ....
> ```
>
>  Python运行的时候首先加载了site.py，在site.py文件里有这么一段代码:
>
> ```python
> if hasattr(sys, "setdefaultencoding"):
>      del sys.setdefaultencoding
> 在sys加载后,setdefaultencoding方法被删除了,所以我们要通过重新导入sys来设置系统编码。

### #-*- encoding: utf-8 -*-
参照[PEP 0263 -- Defining Python Source Code Encodings](http://legacy.python.org/dev/peps/pep-0263/)，这里有几种写法，其中最常见的是

    # -*- coding: utf-8 -*-

注意，该行必须位于源文件的第一行或者第二行。

### Decode and Encode
以下是Decode和Encode的定义：
> encode([encoding], [errors='strict']), which returns an 8-bit string version of the Unicode string, encoded in the requested encoding.
> decode([encoding], [errors]) interprets the string using the given encoding

如encode('utf-8')用utf-8将字符串编码为unicode字符串。decode('utf-8')将unicode字符串解码为utf-8字符串。

Referece
---
- [Unicode HOWTO](https://docs.python.org/2/howto/unicode.html)


