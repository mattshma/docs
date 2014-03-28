Some Commands...
===

以下是一些常用的命令：

find && xargs
---
当需要操作find找出来的文件时，可以用如下命令

- `find /path -type f -exec rm '{}' \;` 或 `find /path -type f -exec rm '{}' +`  
 `{}`表示find出来的文件名，`;`表示命令结束，`\`用来转义";"。

- `find /path -type f -print0 | xargs -0 rm`   
  `-print0`表示输出以null分隔（-print使用换行）；`-0`表示输入以null分隔。该命令比find的`exec`效率高。



 
