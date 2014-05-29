sed
===

基本语法： `sed [OPTION]... {script-only-if-no-other-script} [input-file]...`

行寻址
---
默认情况下，sed使用的命令会作用于文本数据的所有行。如果只想作用于指定行，可以指定行寻址.

行寻址有两种方式：指定行（`[address]/command`）或 指定文本（`/pattern/command`）。

在address中，可以使用数字指明，^表示第一行，$表示最后一行，使用可以使用类似这样的语法: `sed '2,$s/com1/com2/' file`。一个文本过滤的例子： `sed '/text/s/com1/com2/' file`将会匹配所有包含text的行。如果需要匹配多次，可以使用`{}`这多条命令组合起来。行寻址能应用于下面这些项中。

替换
---

替换部分的选项：`s/pattern/replacement/flags`.

其中flags有4种可用值：  
- 数字  第几处匹配的内容将被替换  
- g  所有匹配的内容都将被替换  
- p  打印修改行的内容  
- w file  将替换结果写至文件中  

`p`和`-n`配合使用，会只输出修改过的行。如果模式文本中出现特殊字符， 使用`\`转义。还有一点需要指出的是，能和被替换文本区分开的单字符（如 ;, {, *, ., etc），都可以替代`/`做字符串分隔符，这在被替换文件包含路径时会很有用。

删除
---

sed使用`d`来删除文本。如`sed '3,$d' file`来删除第三行至最后一行中的所有行。对于文本过滤运用到删除中，需要注意的地方是，两个文本模式可用来删除某个范围内的行，即第一个模式打开行删除，第二个模式关闭行删除。如果第一个模式匹配到，而第二个模式没有匹配到，将会删除从匹配行到最后行间的所有行。如

```shell
% cat test.list 
this is line 1
this is line 2
this is line 3
this is line 4
this is line 5
this is line 1
this is line 7
this is line 8
% sed '/1/, /3/d' test.list
this is line 4
this is line 5
```
插入 && 修改
---

插入与修改(c)的语法一样。其中插入可分为指定行插入(i)和指定行后插入(a)。语法是`sed '3ctext' file`，其中`3ctext`还可以写成`3c\text`或者`3c text`等。

转换
---

语法：`y/inchars/outchars`。

转换命令会进行inchars和outchars的一一映射，依次将inchars的字符一一转换为outchars中的字符 。

读写文件
---

可以使用`r file`和`w file`来读写文件。如`sed '1,2w file2' file1`将file1中的前两行写入到file2中。

其他
---

直接修改文件，需要使用`-i`这个参数。

