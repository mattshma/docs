# File操作
文件是日常工作中经常接触的，简单的读写查下官网API即可，但有时会出现一些意外的情况。这里先说下文件相关的基本操作，再说下遇到的问题及解决方案。

## open 和 file

Python自带[open](https://docs.python.org/2/library/functions.html#open)和[file](https://docs.python.org/2/library/functions.html#file)两个函数用于初始化文件对象。`file(name[, mode[, buffering]])`的参数同open一样，但`open()`函数更适合打开文件，若打开文件成功，返回[file](https://docs.python.org/2/library/functions.html#file)对象，若打开文件失败，则抛出[IOError]异常。而`file()`更适合做类型检测，如`isinstance(f, file)`。这里重点介绍下open。

open的函数原型为`open(name[, mode[, buffering]])`，各参数说明如下：

- name

将被打开的文件名。

- mode

表示该文件将被如何打开，文件有文本文件和二进制文件两种，默认为文本文件格式(可使用`'t'`指定)，若打开的是一个二进制文件，需使用`'b'`指定文件类型，因此mode有如下几种：

模式  |  说明
------|--------
r     | 以只读模式打开文件，若mode被忽略的话，默认使用文本文件的只读模式。
rb    | 以二进制格式的只读模式打开文件。
r+    | 以读写模式打开文件。'+'表示更新文件。
rb+   | 以二进制格式的读写模式打开文件。
w     | 以只写模式打开文件。如果该文件已存在则将其覆盖，否则创建新文件。
wb    | 以二进制格式的只写模式打开文件。如果该文件已存在则将其覆盖，否则创建新文件。
w+    | 以读写模式打开文件。如果该文件已存在则将其覆盖，否则创建新文件。
wb+   | 以二进制格式的读写模式打开文件。如果该文件已存在则将其覆盖，否则创建新文件。
a     | 以追加模式打开文件。如果文件已存在，文件指针将位于文件尾，否则创建新文件。
ab    | 以二进制格式的追加模式打开文件。如果文件已存在，文件指针将位于文件尾，否则创建新文件。
a+    |


- buffering

表示文件使用的buffer大小。0代表不使用buffer，1表示使用line buffer，其他正数代表buffer的大小，单位为bytes，负数代表使用系统buffer，忽略的话，也使用系统buffer。




