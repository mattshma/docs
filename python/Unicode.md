Unicode、ASCII and UTF-8
===

Introduce
---
计算机中一切均为byte(字节)，硬盘中文件是以byte进行存储的，网络传输也是以byte为单位进行的。所以为了表示人类使用的英文字母和符号，需将这些文字符号与byte一一编码对应，这导致了字符集的产生（如UCS、Unicode、Ascii etc..)，当需要存储和传播这些字符时，需要将之编码，这导致了各种各样编码方式的产生（UTF-8, UTF-16, Ascii etc..）。

在计算机还未普及的时候，当时只有 ASCII 字符集和编码方式，但当计算机普及后，非英文国家为了显示各自的语言文字，又创造了很多字符集和编码方式，当这里不同的编码方式在网络上传播时，就产生乱码。

UCS
---
为了解决上述问题，国际标准组织 ISO 10646 定义了通用字符集（Universal Character Set，简称UCS）。UCS是其他所有字符集的超集，任何字符翻译成UCS格式，再翻译回去，不会丢失信息。ISO 10646定义了一个 31 位的字符集，UCS由一个个平面（Plane）组成，每个平面由 2<sup>16</sup> 个代码点（code point）构成。基本上大部分使用的字符，包括以前各种各样的编码格式都放在第一平面，这个平面被叫做 基本多文种平面（BMP）或 Plane 0。

UCS有三种级别的实现：

- Level 1   
  不支持组合字符和谚文字母字符

- Level 2  
  类似于级别1，但在某些文字中，允许一列固定的组合字符，因为如果没有最起码的几个组合字符，UCS就不能完整地表达这些语言

- Level 3  
  支持所有的通用字符集字符

Unicode
---
在上世纪80年代后期，除了 ISO 为了解决字符的问题，众多软件生产商也发起 Unicode 项目来解决这个问题。到90年代时，双方都意识到世界不需要两套不兼容的字符集，于是双方开始合并双方的工作成果。Unicode实现了 Level 3 的基本多平种平面，由于 Unicode方便记忆，因此使用的更加广泛。


UTF-8
---
UCS 和 Unicode 仅仅是一个指定了整数与字符间对应关系的字符集。对于一系列的bytes而言，存在若干种字符序列和它们对应的整数值序列，最明显的以 2 个byte 或 4 个byte对字符及对应的整数值进行编码，这两种编码方式分别叫 UCS-2 和 UCS-4。但是在unix世界中，使用 UCS-2 和 UCS-4 会包含像 `\0` 和 `/` 等具有特殊意思的字符。

UTF-8 是一种可变长字符编码，它可以用来表示 Unicode 标准中的任何字符，且编码的第一个字节仍与 ASCII 兼容。因此 UTF-8 逐渐成为最流行的对 Unicode 进行传播和存储的编码方式。


以下是在使用Python过程中遇到的一些问题，将之整理如下

Encode and Decode
---
Python有两种类型的字符串类型：str字符串和Unicode字符串。Python根据电脑默认的locale设置将字节流转化成字符，而大部分系统locale设置为ACSII编码。

可以使用如下方法查看一个字符串是不是Unicode字符串:

```python
if isinstance(str, unicode):
    pass;	
```

若想改变Python编码格式为其他编码格式，如UTF-8，有如下3种方法。

### sys.setdefaultencoding('utf-8')
第一种方法是改变系统编码，可以使用`sys.getdefaultencoding()`来获取系统的默认编码。通过以下方法来修改系统编码：

```
import sys
reload(sys)
sys.setdefaultencoding('utf-8')
```

**为什么要reload(sys)**
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

**更新**

这种方法不太推荐，可参考：[Why sys.setdefaultencoding() will break code](https://anonbadger.wordpress.com/2015/06/16/why-sys-setdefaultencoding-will-break-code/)和[Dangers of sys.setdefaultencoding('utf-8')](http://stackoverflow.com/questions/28657010/dangers-of-sys-setdefaultencodingutf-8)。

### #-*- encoding: utf-8 -*-
参照[PEP 0263 -- Defining Python Source Code Encodings](http://legacy.python.org/dev/peps/pep-0263/)，这里有几种写法，其中最常见的是

    # -*- coding: utf-8 -*-

注意，该行必须位于源文件的第一行或者第二行。

### Decode and Encode
以下是Decode和Encode的定义：
> encode([encoding], [errors='strict']), which returns an 8-bit string version of the Unicode string, encoded in the requested encoding.
> decode([encoding], [errors]) interprets the string using the given encoding

如encode('utf-8')用utf-8将字符串编码为unicode字符串。decode('utf-8')将unicode字符串解码为utf-8字符串。

### 实际应用

在实际应用中，通过连接数据库，来读取文件，为保证数据不乱码。有如下地方需要注意：

#### MySQL服务器配置文件配置

[配置](http://dev.mysql.com/doc/refman/5.7/en/charset-server.html)如下：

```
[mysqld]
init-connect='SET NAMES utf8'
character-set-server=utf8
collation-server=utf8_general_ci

[client]
default-character-set=utf8

[mysql]
default-character-set=utf8
```

#### MySQL的database和表

[DataBase](http://dev.mysql.com/doc/refman/5.7/en/charset-database.html)的编码设置如下：

```
CREATE DATABASE db_name CHARACTER SET utf-8
```

[Table](http://dev.mysql.com/doc/refman/5.7/en/charset-table.html)的编码设置如下：

```
CREATE TABLE tbl_name (column_list) ENGINE=InnoDB DEFAULT CHARSET=utf8
```

#### python 设置
设置python文件格式:`# -*- coding: utf-8 -*-`。

#### MySQL实例设置

初始化mysql连接时，指定utf-8:

```
import MySQLdb

conn = MySQLdb.connect(host, user, password, database, charset='utf8')
```

#### dict的设置

上述设置后，一般的问题都能解决，但对于dict和list而言，对于非ascii值仍会出现乱码。原因见[这里](https://docs.python.org/2/library/json.html#basic-usage)。

如:
```
>>> test=['a', '1', '测试']
>>> test
['a', '1', '\xe6\xb5\x8b\xe8\xaf\x95']
```
若要正确输出，需要encode为utf-8，如下：
```
# -*- coding: utf-8 -*-
import json

test1 = ['a', '1', '测试']
print json.dumps(test1, encoding="UTF-8", ensure_ascii=False)

test2 = {'key': '测试'}
print json.dumps(test2, encoding="UTF-8", ensure_ascii=False)
```
运行脚本，输出结果如下：
```
> python test.py
["a", "1", "测试"]
{"key": "测试"}
```



Referece
---
- [Pragmatic Unicode](http://nedbatchelder.com/text/unipain.html)
- [Unicode](http://www.cl.cam.ac.uk/~mgk25/unicode.html)
- [Why do we need Unicode?](http://stackoverflow.com/questions/2241348/what-is-unicode-utf-8-utf-16)
- [Unicode HOWTO](https://docs.python.org/2/howto/unicode.html)  
- [通用字符集](http://zh.wikipedia.org/wiki/Universal_Character_Set)
