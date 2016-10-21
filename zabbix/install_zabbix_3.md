# 安装Zabbix3

## 打包
下载Zabbix3后，进行打包，SPEC文件如下：

```
%global debug_package %{nil}
%global _prefix /opt/zabbix
%global _exec_prefix /opt/zabbix
%global _sysconfdir /opt/zabbix/etc
%global _mandir /opt/zabbix/man

Name:		zabbix-server-ubd
Version:	3.0.5
Release:	1%{?dist}
Summary:	youzu bigdata zabbix server

Group:		Applications/System
License:	GPL
URL:	        http://www.zabbix.com/
Source0:	zabbix-server-ubd-3.0.5.tar.gz
BuildRoot:      %{_tmppath}/%{name}-%{version}-%{release}-root

BuildRequires: net-snmp-devel, libxml2-devel, libcurl-devel, mysql-devel
Requires(pre): /usr/sbin/useradd
Requires(post): /sbin/chkconfig
Requires(preun): /sbin/chkconfig
Requires(preun): /sbin/service

%description
youzu bigdata zabbix server.

%pre
%define log_dir /var/log/zabbix
%define user zabbix

grep "^%{user}" /etc/passwd
if [ $? -ne 0 ]; then
  useradd %{user}
fi

if [ ! -d %{log_dir} ]; then
  mkdir -p %{log_dir}
  chown -R %{user}:%{user} %{log_dir}
fi

%prep
%setup -q

%build
%configure --enable-server --enable-agent --with-mysql --enable-ipv6 --with-net-snmp --with-libcurl --with-libxml2
make %{?_smp_mflags}

%install
rm -rf %{buildroot}
make install
install -m 0775 misc/init.d/fedora/core/zabbix_agentd /etc/init.d/zabbix-agent-ubd
install -m 0775 misc/init.d/fedora/core/zabbix_server /etc/init.d/%{name}
sed -i "s#BASEDIR=/usr/local#BASEDIR=/opt/zabbix#g" /etc/init.d/zabbix-agent-ubd
sed -i "s#BASEDIR=/usr/local#BASEDIR=/opt/zabbix#g" /etc/init.d/%{name}

%post
/sbin/chkconfig --add %{name}
/sbin/chkconfig --add zabbix-agent-ubd

%clean
rm -rf %{buildroot}

%postun
rm -rf /etc/init.d/zabbix-*-ubd
/sbin/service %{name} stop > /dev/null 2>&1
/sbin/chkconfig --del %{name}
/sbin/chkconfig --del zabbix-agent-ubd
rm -rf /etc/init.d/%{name}
rm -rf /etc/init.d/zabbix-agent-ubd

%files
%defattr(-,%{user},%{user},-)
%doc

%changelog
```

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

在`/usr/local/php/etc`中，修改php-fpm.conf.default为php-fpm.conf，修改php-fpm.d/www.conf.default为php-fpm.d/www.conf，并修改www.conf如下地方
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

