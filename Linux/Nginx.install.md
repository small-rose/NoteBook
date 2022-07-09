---
layout: default
title: Nginx Linux Install
nav_order: 1
parent: Linux
---

# Nginx Linux Install
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---


## yum 方式在线安装

该种安装方式需要依赖网络。

### 前置操作

理论上是可选的。

更新系统

```bash
yum install epel-release -y
yum update
```

安装yum-utils （已安装提示）

```bash
[root@VM-0-3-centos ~]#  yum -y install yum-utils
Loaded plugins: fastestmirror, langpacks
Repository epel is listed more than once in the configuration
Loading mirror speeds from cached hostfile
Package yum-utils-1.1.31-50.el7.noarch already installed and latest version
Nothing to do
```

### 编辑yum源

```bash
[root@VM-0-3-centos ~]# vi /etc/yum.repos.d/nginx.repo
```

内容如下：

```txt
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/7/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
```

### 查看Nginx的yum源

```bash
[root@VM-0-3-centos ~]# yum list | grep nginx
Repository epel is listed more than once in the configuration
collectd-nginx.x86_64                    5.8.1-1.el7                   epel     
munin-nginx.noarch                       2.0.66-1.el7                  epel     
nginx.x86_64                             1:1.22.0-1.el7.ngx            nginx-stable
nginx-all-modules.noarch                 1:1.20.1-9.el7                epel     
nginx-debug.x86_64                       1:1.8.0-1.el7.ngx             nginx-stable
nginx-debuginfo.x86_64                   1:1.22.0-1.el7.ngx            nginx-stable
nginx-filesystem.noarch                  1:1.20.1-9.el7                epel     
nginx-mod-devel.x86_64                   1:1.20.1-9.el7                epel     
nginx-mod-http-image-filter.x86_64       1:1.20.1-9.el7                epel     
nginx-mod-http-perl.x86_64               1:1.20.1-9.el7                epel     
nginx-mod-http-xslt-filter.x86_64        1:1.20.1-9.el7                epel     
nginx-mod-mail.x86_64                    1:1.20.1-9.el7                epel     
nginx-mod-stream.x86_64                  1:1.20.1-9.el7                epel     
nginx-module-geoip.x86_64                1:1.22.0-1.el7.ngx            nginx-stable
nginx-module-geoip-debuginfo.x86_64      1:1.22.0-1.el7.ngx            nginx-stable
nginx-module-image-filter.x86_64         1:1.22.0-1.el7.ngx            nginx-stable
nginx-module-image-filter-debuginfo.x86_64
                                         1:1.22.0-1.el7.ngx            nginx-stable
nginx-module-njs.x86_64                  1:1.22.0+0.7.4-1.el7.ngx      nginx-stable
nginx-module-njs-debuginfo.x86_64        1:1.22.0+0.7.4-1.el7.ngx      nginx-stable
nginx-module-perl.x86_64                 1:1.22.0-1.el7.ngx            nginx-stable
nginx-module-perl-debuginfo.x86_64       1:1.22.0-1.el7.ngx            nginx-stable
nginx-module-xslt.x86_64                 1:1.22.0-1.el7.ngx            nginx-stable
nginx-module-xslt-debuginfo.x86_64       1:1.22.0-1.el7.ngx            nginx-stable
nginx-nr-agent.noarch                    2.0.0-12.el7.ngx              nginx-stable
owncloud-nginx.noarch                    9.1.5-1.el7                   epel     
pagure-web-nginx.noarch                  5.13.3-2.el7                  epel     
pcp-pmda-nginx.x86_64                    4.3.2-13.el7_9                updates  
python2-certbot-nginx.noarch             1.11.0-1.el7                  epel     
sympa-nginx.x86_64                       6.2.68-1.el7                  epel     
```

### 执行安装

```bash
[root@VM-0-3-centos ~]# yum -y install nginx
```

等待安装完成。

### 查看参数

查看nginx版本
```
[root@VM_159_140_centos www]# nginx -v
nginx version: nginx/1.14.2
```

查看编译参数

```bash
[root@VM-0-3-centos ~]# nginx -V
nginx version: nginx/1.22.0
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC) 
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: 
--prefix=/etc/nginx 
--sbin-path=/usr/sbin/nginx 
--modules-path=/usr/lib64/nginx/modules 
--conf-path=/etc/nginx/nginx.conf 
--error-log-path=/var/log/nginx/error.log 
--http-log-path=/var/log/nginx/access.log 
--pid-path=/var/run/nginx.pid 
--lock-path=/var/run/nginx.lock 
--http-client-body-temp-path=/var/cache/nginx/client_temp 
--http-proxy-temp-path=/var/cache/nginx/proxy_temp 
--http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp 
--http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp 
--http-scgi-temp-path=/var/cache/nginx/scgi_temp 
--user=nginx 
--group=nginx 
--with-compat 
--with-file-aio --with-threads 
--with-http_addition_module 
--with-http_auth_request_module 
--with-http_dav_module 
--with-http_flv_module 
--with-http_gunzip_module 
--with-http_gzip_static_module --with-http_mp4_module 
--with-http_random_index_module --with-http_realip_module 
--with-http_secure_link_module --with-http_slice_module 
--with-http_ssl_module --with-http_stub_status_module 
--with-http_sub_module --with-http_v2_module 
--with-mail --with-mail_ssl_module 
--with-stream --with-stream_realip_module 
--with-stream_ssl_module --with-stream_ssl_preread_module 
--with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong 
--param=ssp-buffer-size=4 -grecord-gcc-switches -m64 -mtune=generic -fPIC' 
--with-ld-opt='-Wl,-z,relro -Wl,-z,now -pie'
```

### 查看配置文件

```bash
[root@VM-0-3-centos ~]# more /etc/nginx/nginx.conf 
```

检查配置文件语法是否正确

```bash
# 也可以直接 nginx -t
[root@VM-0-3-centos ~]#  nginx -tc /etc/nginx/nginx.conf 
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

### 启停nginx服务

```bash
nginx start
nginx stop
nginx -s reload -c /etc/nginx/nginx.conf
# 或者
nginx -s reload
```

## 手工离线安装

该种安装方式机器可以没有外网，需要依赖自己下载好nginx安装包，上传到机器。


### 安装要求的环境

手工安装需要自己准备编译环境，没有的环境需安装。

1.需要安装gcc环境
```bash
[root@VM-0-3-centos ~]# yum install gcc-c++
```
2.第三方的开发包

1 PERE

PCRE(Perl Compatible Regular Expressions)是一个Perl库，包括 perl 兼容的正则表达式库。nginx的http模块使用pcre来解析正则表达式，所以需要在linux上安装pcre库。

注：pcre-devel是使用pcre开发的一个二次开发库。nginx也需要此库。

```bash
[root@VM-0-3-centos ~]# yum install -y pcre pcre-devel
```

2 zlib

zlib库提供了很多种压缩和解压缩的方式，nginx使用zlib对http包的内容进行gzip，所以需要在linux上安装zlib库。

```bash
[root@VM-0-3-centos ~]# yum install -y zlib zlib-devel
```

3 openssl

OpenSSL 是一个强大的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及SSL协议，

并提供丰富的应用程序供测试或其它目的使用。nginx不仅支持http协议，还支持https（即在ssl协议上传输http），所以需要在linux安装openssl库。

```bash
[root@VM-0-3-centos ~]# yum install -y openssl openssl-devel
```

### 上传安装包并解压

```bash
[root@VM-0-3-centos ~]# rz
[root@VM-0-3-centos ~]# tar -xvf nginx-1.22.0.tar.gz -C /u01/software/nginx-1.22.0
```

### 准备编译文件

使用cofigure命令创建一个makeFile文件.使用该命令一定要进入nginx的解压目录


```bash
cd  /u01/software/nginx-1.22.0

./configure \
--prefix=/u01/software/nginx \
--pid-path=/var/run/nginx/nginx.pid \
--lock-path=/var/lock/nginx.lock \
--error-log-path=/u01/software/nginx/logs/error.log \
--http-log-path=/u01/software/nginx/logs/access.log \
--with-http_gzip_static_module \
--http-client-body-temp-path=/var/temp/nginx/client \
--http-proxy-temp-path=/var/temp/nginx/proxy \
--http-fastcgi-temp-path=/var/temp/nginx/fastcgi \
--http-uwsgi-temp-path=/var/temp/nginx/uwsgi \
--http-scgi-temp-path=/var/temp/nginx/scgi \
--with-http_stub_status_module \
--with-http_ssl_module \
--with-file-aio \
--with-http_realip_module
```

\ 表示命令还没有输入完，换行的意思。

--prefix=/usr/local/nginx  表示软件安装到/usr/local/nginx下面。
后面执行`make install` 的时候就不用在指定安装路径。执行完成后查看目录里面已经多了一个Makefile文件

上面我们准备的目录如果没有就要创建一下，自己提前规划好目录。

```bash
mkdir /var/temp/nginx -p
```
### 编译与安装

```
# 这里是把几个命令一起执行，分开执行也是可以的
[root@VM-0-3-centos nginx-1.22.0]# cd  /u01/software/nginx-1.22.0 && make && make install
```

等待安装完成即可。

### 查看安装文件

进入安装位置/usr/local/nginx查看目录结构

```
[root@localhost nginx-1.22.0]# cd /u01/software/nginx && pwd  && ls -lhrt
/u01/software/nginx
total 16K
drwxr-xr-x. 2 root root 4.0K May  5 18:52 html
drwxr-xr-x. 2 root root 4.0K May  5 18:55 sbin
drwxr-xr-x. 2 root root 4.0K May  5 19:12 logs
drwxr-xr-x. 2 root root 4.0K May 27 17:56 conf
```
其中html是里面首页html文件。conf里面是配置文件。sbin里面只执行文件。

### 配置文件

```bash
cat /u01/software/nginx/conf/nginx.conf
```

### 启停
```bash
./u01/software/nginx/sbin/nginx start
./u01/software/nginx/sbin/nginx stop
./u01/software/nginx/sbin/nginx -t
./u01/software/nginx/sbin/nginx -s reload
```
如果不想每次执行都带完整路径，建一个软链接

```bash
ln -s /u01/software/nginx/sbin/nginx /usr/sbin/
```

以后就可以愉快的直接nginx

```bash
nginx start
nginx stop
nginx -t
nginx -s reload
```

### nginx 安装总结

在线安装

```bash
yum install gcc-c++       
yum install -y pcre pcre-devel
yum install -y zlib zlib-devel
yum install -y openssl openssl-devel

cd /usr/local
wget http://nginx.org/download/nginx-1.20.1.tar.gz
tar -zxvf  nginx-1.20.1.tar.gz
./configure && make && make install

# 这是 nginx 软连接
ln -s /usr/local/nginx/sbin/nginx  /usr/local/bin/nginx
nginx
ps -ef|grep nginx
```

