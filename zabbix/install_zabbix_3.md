# 安装Zabbix3

## 打包
下载Zabbix3后，进行打包，SPEC文件如下：

```
%global debug_package %{nil}
%global _prefix /opt/zabbix-3.0.5
%global _exec_prefix %{_prefix}
%global _sysconfdir %{_prefix}/etc
%global _mandir %{_prefix}/man

%global init_dir /etc/rc.d/init.d
%global ln_dir /opt/zabbix
%global log_dir /var/log/zabbix
%global pid_dir /var/run/zabbix
%global user zabbix

Name:		zabbix-agent-ubd
Version:	3.0.5
Release:	1%{?dist}
Summary:	youzu bigdata zabbix agent

Group:		Applications/System
License:	GPL
URL:	        http://www.zabbix.com/
Source0:	zabbix-agent-ubd-3.0.5.tar.gz
BuildRoot:	%{_tmppath}/%{name}-%{version}-%{release}-root

Requires(pre): /usr/sbin/useradd
Requires(post): /sbin/chkconfig, /bin/ln
Requires(preun): /sbin/chkconfig
Requires(preun): /sbin/service

%description
youzu bigdata zabbix agent.

%pre
grep "^%{user}" /etc/passwd > /dev/null 2>&1
if [ $? -ne 0 ]; then
  useradd %{user}
fi

if [ ! -d %{log_dir} ]; then
  mkdir -p %{log_dir}
  chown -R %{user}:%{user} %{log_dir}
fi

if [ ! -d %{pid_dir} ]; then
  mkdir -p %{pid_dir}
  chown -R %{user}:%{user} %{pid_dir}
fi
%prep
%setup -q

%build
%configure --enable-agent
make %{?_smp_mflags}

%install
rm -rf %{buildroot}
%make_install
%{__install} -d %{buildroot}%{init_dir}
%{__install} -m 0775 misc/init.d/fedora/core/zabbix_agentd %{buildroot}%{init_dir}/%{name}
sed -i "s#BASEDIR=/usr/local#BASEDIR=/opt/zabbix#g" %{buildroot}%{init_dir}/%{name}

%post
/bin/ln -s %{_prefix} %{ln_dir}
/sbin/chkconfig --add %{name}

%clean
rm -rf %{buildroot}
rm -rf %{ln_dir}

%preun
/sbin/service %{name} stop > /dev/null 2>&1
/sbin/chkconfig --del %{name}
rm -rf /etc/init.d/%{name}

%files
%defattr(-,zabbix,zabbix,-)
%{_prefix}
%{init_dir}/%{name}

%changelog
```

说明下，此spec文件将zabbix安装路径设为/opt/zabbix-3.0.5，然后做软链/opt/zabbix指向/opt/zabbix-3.0.5这个目录，配置文件位于/opt/zabbix/etc目录中。

打包好后，放入yum源中并安装。

### 升级php
在升级php前，先安装如下一些依赖，若yum不能安装，则手动编译安装。

- jpeg   
下载[jpeg压缩包](http://www.ijg.org/files/)，解决安装，jpeg lib会安装在`/usr/local/lib`。   
- PNG    
下载[libpng](http://www.libpng.org/pub/png/libpng.html)安装。    
- FreeType     
下载[freetype压缩包](http://download.savannah.gnu.org/releases/freetype/)，解压安装。   


编译PHP：
```
./configure --prefix=/usr/local/php \
--with-mcrypt=/usr/include \
--enable-mysqlnd \
--with-mysqli \
--with-pdo-mysql \
--enable-fpm \
--with-iconv \
--with-zlib \
--enable-xml \
--enable-shmop \
--enable-sysvsem \
--enable-inline-optimization \
--enable-mbregex \
--enable-mbstring \
--enable-ftp \
--enable-gd-native-ttf \
--with-openssl \
--enable-pcntl \
--enable-sockets \
--with-xmlrpc \
--enable-zip \
--enable-soap \
--with-gettext \
--enable-session \
--with-curl \
--with-gd \
--with-jpeg-dir \
--with-freetype-dir \
--enable-opcache \
--enable-bcmath
```

注意，php编译时，zabbix依赖的参数见[Zabbix Requirements](https://www.zabbix.com/documentation/3.0/manual/installation/requirements)。

`make install`后，新建文件index.php，内容如下：
```
<?php
phpinfo();
?>
```

查看php配置文件路径，如/usr/local/php/lib，在php源文件中，拷贝文件`cp php.ini-production /usr/local/php/lib/php.ini`，并修改如下部分： 
```
max_execution_time = 300
post_max_size = 32M
date.timezone = Asia/Shanghai
max_input_time = 300
extension = "gettext.so"
```

php扩展可通过phpize来安装，在php解压包中执行`phpize`，然后configure和make 安装即可。

在`/usr/local/php/etc`中，修改 php-fpm.conf.default 为 php-fpm.conf，修改 php-fpm.d/www.conf.default 为 php-fpm.d/www.conf，并修改www.conf如下地方:

```
pm.max_children = 24
pm.start_servers = 8
pm.max_spare_servers = 16
```

在php源文件中，执行`cp sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm`，将php-fpm加入到`/etc/init.d`目录下。


## 配置nginx

新建`/etc/nginx/conf.d/zabbix.conf`如下：
```
server {
    listen      80;
    server_name zabbixbd.xxx.com;
    charset     utf-8;

    root        /var/www/zabbix;
    index       index.php index.html index.htm;

    location ~* /\.ht {
        deny  all;
    }

    location ~ \.php$ {
        fastcgi_pass        127.0.0.1:9000;
        fastcgi_index       index.php;
        include             fastcgi_params;
        fastcgi_param       SCRIPT_FILENAME  $document_root/$fastcgi_script_name;
    }

    location ~* \.(js|css|png|jpg|jpeg|gif|ico)$ {
     expires 30d;
   }
}
```

重载nginx即可。

zabbix初始用户名为admin，密码为zabbix。

## 报错

### syntax error, unexpected '[' in XXXX

启动php-fpm后，nginx后台报错如下：
```
FastCGI sent in stderr: "PHP message: PHP Parse error:  syntax error, unexpected '[' in /var/www/zabbix/index.php on line 29" while reading response header from upstream,
```

见[ZBX-10179](https://support.zabbix.com/browse/ZBX-10179)。

### PHP gd JPEG image support missing.
需要安装JPEG，并重新编译安装PHP，在configure前，先执行`make clean`。


### libc.so.6(GLIBC_2.15)(64bit) is needed by zabbix
打包完成后，yum安装报错如下：
```
libc.so.6(GLIBC_2.14)(64bit) is needed by zabbix-agent-ubd-3.0.5-1.el6.x86_64
libc.so.6(GLIBC_2.15)(64bit) is needed by zabbix-agent-ubd-3.0.5-1.el6.x86_64
```

查看libc版本：
```
# strings /lib64/libc.so.6 |grep GLIBC_
GLIBC_2.2.5
GLIBC_2.2.6
GLIBC_2.3
GLIBC_2.3.2
GLIBC_2.3.3
GLIBC_2.3.4
GLIBC_2.4
GLIBC_2.5
GLIBC_2.6
GLIBC_2.7
GLIBC_2.8
GLIBC_2.9
GLIBC_2.10
GLIBC_2.11
GLIBC_2.12
GLIBC_PRIVATE
```

Google了下，发现[Centos 6.8 zabbix 3.0 non support component](https://support.zabbix.com/browse/ZBXNEXT-3378)有如下一段话：
> Zabbix 3.0 Server is not officially supported for RHEL 6
> However we have packages http://repo.zabbix.com/zabbix/3.0/rhel/6/x86_64/deprecated/ but without any guarantee that they will work.
> CentOS 6 does not have official PHP 5.4 version. It is minimal supported version for Zabbix 3.0 web-interface.

但看[zabbix下载页](http://www.zabbix.com/download.php)提供的rpm包，包括centos6，说明应该不会存在问题的，于是下载源码包，在出现问题的机器上直接编译安装，zabbix能正常安装，这说明zabbix对该libc应该不存在要求。想到之前打包的机器升级过GCC，会不会是因为打包机器GCC版本过高导致的呢？于是在出问题的机器上打包，将rpm包放在yum repo中，其他机器重新生成cache，安装成功。


