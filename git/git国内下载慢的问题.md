## Git 国内下载慢的问题

在下载 Hadoop 源码过程中，Git 下载速度一起几 KB/s，对于一般 Repo 而言，速度太慢了。由于自己有 Socks 代理，于是设置 Git 让其使用代理下载。



若需要拉取所有 http 协议的仓库都通过代理的话，可以设置全局代理，如下：

```
# git config --global http.proxy sock5://127.0.0.1:1080
# git config --global https.proxy sock5://127.0.0.1:1080
```



若只需要对部分网站 http 协议的仓库通过代理的话，可针对这些网站单独设置，格式为 `http.<url>.proxy`，如下是针对 Github 设置的代理：

```
# git config --global http.http://github.com.proxy sock5://127.0.0.1:1080
# git config --global http.https://github.com.proxy sock5://127.0.0.1:1080
```



注意：针对 https 协议设置的是 `http.https://github.com.proxy` 。



由于代理针对 http 协议，所以拉取 Repo 时，需要使用仓库的 http/s 地址，如 `git clone https://github.com/apache/hadoop.git ` ，而非 ssh 地址。



若需要取消代理的话，可通过 `git config --global --unset http.proxy` 的方式 取消对应的设置。



## 参考

- [git-config(1) Manual Page](https://www.kernel.org/pub/software/scm/git/docs/git-config.html#_example)