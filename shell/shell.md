Shell
===

## 获取脚本当前目录
命令为`dirname $0`，在脚本中经常需要先切换到该脚本的所在目录下，如下：
```
cd `dirname $0`
```

Commands that are symbols
---
内建的符号命令

 Symbol | Command
 -------|-----------------------------
 ()     | 在Subshell中执行`()`中的命令
 $()    | 命令替换(Command substitution)
 (())   | arithmetic evaluation
 $(())  | arithmetic expansion
 []     | 测试命令
 [[]]   | 条件判断


shell执行过程
---
1. 首先从命令行中找出特殊字符（元字符）

Command-line expansion
---

### 命令替换
在bash中，允许将命令的标准输出当变量值一样使用，其语法为：`$(commands)` 或 <code>``</code>，建议使用前一种方式，后一种是老样式了 。如以下例子：

    $ echo $PWD
    $ /home/lucky
    $ echo $(pwd)
    $ /home/lucky
    $ echo `pwd`
    $ /home/lucky

### 扩展
#### {}扩展  

不支持转义，`{}`内不能有空格，以`,`做为分隔，可以使用一个范围，用例如下：

    $ echo a{a,b,c}
    $ aa ab ac
    $ echo a{a..c}
    $ aa ab ac

#### ~扩展  
当进行`~`扩展时，`~`必须位于一个token的开关，即前面有个空格。

- ~   
  表示当前用户的HOME目录
- ~+  
  扩展`~+`时替换为当前工作目录
- ~-  
  扩展`~-`时替换为上一个打开的目录

#### 变量扩展  

主要是使用`$`来进行扩展的，一些操作如下：

  formal        | meaning
  --------------|---------
  ${!var}       | 间接引用，相当于C里面的指针.like:`$ a=b;b=1;echo ${!a}`，输出结果为1
  ${var:-$var2} | 如果var已被设置且非空，返回var的值，否则返回var2的值
  ${var:=$var2} | 如果var已被设置且空，刚返回var的值，否则返回var2的值
  ${var:+$var2} | 如果var已被设置且非空，则返回var2的值，否则什么也不返回
  ${var:?$var2} | 如果var已被设置且值非空，则返回var的值，否则打印var2并退出shell
  ${var:offset} | 返回var中从offset开始的字符串
  ${var:(+/-)offset:length} | 返回var中从(倒数)offset开始长为length的字符串
  ${#var}       | 返回var的字符长度
  ${var%\*pattern<sup>[1]</sup>} | 从var尾部与pattern进行最小匹配，并删除匹配到的部分
  ${var%%\*pattern} | 从var尾部与pattern进行最大匹配，并删除匹配到的部分
  ${var#pattern\*<sup>[2]</sup>}| 从var头部与pattern进行最小匹配，并删除匹配到的内容
  ${var##pattern\*} | 从var头部与pattern进行最大匹配，并删除匹配到的内容
  ${var/pattern/string} <sup>[3]</sup>|使用string替换pattern的最大匹配部分

- [1]: \*根据需要可以位于pattern之前，也可以位于pattern之后。如`echo ${0%/*}`返回当前文件的父目录路径
- [2]: \*根据需要可以位于pattern之前，也可以位于pattern之后
- [3]: 如果pattern以`/`开关则进行全局替换，否则只替换第一个匹配的位置。如果pattern以`#`开始，则起始位置必须匹配，如果以`%`开始则结尾部分必须匹配

#### 算术扩展
主要是$[]和$(())，如果需要使用更复杂的计算，可以使用bc和awk。


Others
---

- `. filename`  
  在当前shell环境里读取并执行filename中的命令，返回值是最后一条命令执行的结果。

