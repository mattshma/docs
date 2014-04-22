Some Grammers
===

以下是在使用Python过程中遇到的一些问题，将之整理如下

utf-8
---
当末指定编码格式时，python源文件默认使用ascii码。所以当文中有中文时，python解释器一般会报错：

    SyntaxError: Non-ASCII character '\xe4' in file /bookstore/app/views.py on line 10, but no encoding declared; see http://www.python.org/peps/pep-0263.html for details

这里需要加上自己使用的文件编码，参照(http://legacy.python.org/dev/peps/pep-0263/)[PEP 0263 -- Defining Python Source Code Encodings]，这里有几种写法，其中最常见的是

    # -*- coding: utf-8 -*-

注意，该行必须位于源文件的第一行或者第二行。


