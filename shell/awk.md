AWK
===
记录些关于awk的容易忘记的地方。

### awk引用shell变量

在shell脚本，通常会遇到awk引用shell变量的情况，通常使用`"'$var'"`（双引号内加单引号）的形式来引用，如下：
```
port=3306
/bin/netstat -ltnp | awk '$4~"'$port'"' |wc -l
```


Reference
---

- [awk cheat sheet](http://www.catonmat.net/download/awk.cheat.sheet.pdf)
