# Linux升级Gcc

在使用mongo时，遇到了`/usr/lib64/libstdc++.so.6: version `GLIBCXX_3.4.15' not found`的问题。查找系统所有的libstdc++.so.6，未找到GLIBCXX_3.4.15。网卡搜索rpm包，仍然一堆依赖，无奈安装高版本GCC。过程如下：

- 下载[GCC4.9.0](http://ftp.gnu.org/gnu/gcc/gcc-4.9.0/gcc-4.9.0.tar.bz2)并解压。
- 安装依赖，执行命令：`gcc-4.9.0/contrib/download_prerequisites`。
- 创建编译目录：`mkdir build_gcc_4.9.0`。
- 进入编译目录开始编译
```
# cd build_gcc_4.9.0
# ../gcc-4.9.0/configure --enable-checking=release --enable-languages=c,c++ --with-system-zlib --enable-shared --enable-threads=posix --disable-multilib
# make -j24
# make install
```

执行`gcc --version`查看版本信息。至此安装结束。

# /usr/lib64/libstdc++.so.6: version 'GLIBCXX_3.4.15' not found

对于这个问题，解决方法是找到编译目录：build_gcc_4.9.0/x86_64-unknown-linux-gnu/libstdc++-v3/src/.libs，将libstdc++.so.6.0.20拷贝到/usr/lib64目录，再将libstdc++.so.6软链到libstdc++.so.6.0.20即可。
