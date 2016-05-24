Centos6.6安装mysql5.6
===

在centos6.5中安装的好好的，结果在centos6.6中，出了些问题，这里说下。

安装依赖
---

如下依赖需一一安装：

- gcc
- gcc-c++
- cmake
- ncurses-devel

若在A机器上出现找不到头文件，而B机器上已安装成功的话，可以按如下步骤来安装依赖：

B机器上执行如下命令: 

```
> updatedb
> locate cxxabi.h
/usr/include/c++/4.4.4/cxxabi.h
> rpm -qf /usr/include/c++/4.4.4/cxxabi.h
libstdc++-devel-4.4.7-4.el6.x86_64
```
在A机器上执行`rpm -qa |grep libstdc++-devel`，若没有则安装之。


