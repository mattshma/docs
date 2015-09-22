源语法说明
===

Ubuntu
---
使用`wget -crmnp http://archive.cloudera.com/cm5/ubuntu/precise/ -R "index.html*"`下载好源后，若结构树如下：

```
ubuntu/
├── precise
│   └── amd64
│       └── cm
│           ├── dists
│           │   ├── precise-cm5.0.1
│           │   │   ├── contrib
│           │   │   │   ├── binary-amd64
│           │   │   │   │   ├── Packages
│           │   │   │   │   ├── Packages.gz
│           │   │   │   │   └── Release
│           │   │   │   └── source
│           │   │   │       ├── Release
│           │   │   │       └── Sources.gz
│           │   │   ├── InRelease
│           │   │   ├── Release
│           │   │   └── Release.gpg
│           │   └── precise-cm5.0.2
│           │       ├── contrib
│           │       │   └── binary-amd64
│           │       │       ├── Packages
│           │       │       ├── Packages.gz
│           │       │       └── Release
│           │       ├── Release
│           │       └── Release.gpg
│           └── pool
│               └── contrib
│                   ├── e
│                   │   └── enterprise
│                   │       ├── cloudera-manager-agent_5.0.2-1.cm502.p0.297~precise-cm5.0.2_amd64.deb
│                   │       ├── cloudera-manager-daemons_5.0.2-1.cm502.p0.297~precise-cm5.0.2_all.deb
│                   │       ├── cloudera-manager-server_5.0.2-1.cm502.p0.297~precise-cm5.0.2_all.deb
│                   │       ├── cloudera-manager-server-db-2_5.0.2-1.cm502.p0.297~precise-cm5.0.2_all.deb
│                   │       └── cloudera-manager-server-db_5.0.2-1.cm502.p0.297~precise-cm5.0.2_all.deb
│                   └── o
│                       ├── oracle-j2sdk1.6
│                       │   └── oracle-j2sdk1.6_1.6.0+update31_amd64.deb
│                       └── oracle-j2sdk1.7
│                           └── oracle-j2sdk1.7_1.7.0+update45-1_amd64.deb
```
在写.list文件时，语法如下：

`deb/deb-src [options] uri distribution [component1] [component2] [...]`
Archive type说明文件的归档类型，有`deb`和`deb-src`两种，`deb`说明归档文件中包含二进制包和预编译包，deb-src说明是源文件，其包括原始代码文件和Debian控制文件(.dsc)和diff.gz。

options是由一个或多个`setting=value`形式的setting构成的，多个setting之间由空格分隔，其有如下形式：

- arch=arch1,arch2,..   
 说明何种cpu架构才能下载该文件。如果未指定，则`APT:Architectures`中定义的所有cpu架构都能下载。

- arch+=arch1,arch2,.. 和 arch-=arch1,arch2,...    
 包括/不包括该种架构能下载

- trusted=yes    
 说明源是一直是经过验证的，即使Release file 没有签名。

uri是apt源的地址，而distribution是发布版本，component为发布版本中的各组件，如:

```
deb [arch=amd64] http://10.10.x.x/ubuntu/precise/amd64/cm/ precise-cm5.0.1 contrib 
```

RedHat
---

若结构如下：
```
epel/6.5/x86_64/
├── Packagesa 
├── zziplib-devel-0.13.62-1.el6.x86_64.rpm
├── zziplib-utils-0.13.62-1.el6.x86_64.rpm
└── repodata
    ├── 401dc19bda88c82c403423fb835844d64345f7e95f5b9835888189c03834cc93-filelists.xml.gz
    ├── 6bf9672d0862e8ef8b8ff05a2fd0208a922b1f5978e6589d87944c88259cb670-other.xml.gz
    ├── 77a287c136f4ff47df506229b9ba67d57273aa525f06ddf41a3fef39908d61a7-other.sqlite.bz2
    ├── 8596812757300b1d87f2682aff7d323fdeb5dd8ee28c11009e5980cb5cd4be14-primary.sqlite.bz2
    ├── dabe2ce5481d23de1f4f52bdcfee0f9af98316c9e0de2ce8123adeefa0dd08b9-primary.xml.gz
    ├── f8606d9f21d61a8bf405af7144e16f6d7cb1202becb78ba5fea7d0f1cd06a0b2-filelists.sqlite.bz2
    └── repomd.xml
```

配置仓库语法如下：
```
[repositoryid]
name=name for this repository
baseurl=url://server1/path/to/repository/
mirrorlist=url://path/to/mirrorlist/repository/
enabled=0/1
gpgcheck=0/1
gpgkey=A URL pointing to the ASCII-armoured GPG key file for the repository
```

以下是各项说明：

- repositoryid: 用于指定一个仓库
- name: 指定仓库名称，可任意
- baseurl: 本仓库的url
- mirrorlist: 指定仓库的镜像站点
- enable: 是否使用本仓库，默认为1，即启用
- gpgcheck: 用于指定是否检查软件包的GPG签名
- gpgkey: 用于指定GPG签名文件的URL

在name和baseurl中使用变量，常见变量如下：

- $releasever: 当前系统的版本号
- $basearch: 当前系统的平台架构

如下：

```
[epel]
name=epel-$releasever
baseurl=http://idc.yum.your.com/epel/$releasever/$basearch/
gpgcheck=0
``` 

其中`$releasever`为`6.5`，`$basearch`为`x86_64`即找到上述结构中的rpm包。


参考
---

- [SOURCES.LIST(5)](http://manpages.debian.org/cgi-bin/man.cgi?sektion=5&query=sources.list&apropos=0&manpath=sid&locale=en)
- [SourceList](https://wiki.debian.org/SourcesList)
