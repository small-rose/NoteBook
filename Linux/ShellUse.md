---
layout: default
title: Shell Linux
nav_order: 2
parent: Linux
---



# Linux - Shell-Project
{: .no_toc }

## Shell of contents for sth.
{: .no_toc .text-delta }

1. TOC
{:toc}

---

1、centos 安装 nginx

```shell script
#!/bin/sh
#author small
#qq small-rose#qq.com

#variable
TOOLS=/tmp/nginx
NGINX_VERSION=1.22.1
NGINXDIR=/usr/local/nginx-${NGINX_VERSION}

#View the current user
[ $UID -eq  0 ] ||  {
	echo "Not enough authority"
	exit 1
}

#Create a working directory
mkdir -p ${TOOLS}  && cd ${TOOLS}
if [  $? -eq 0 ] 
	then
		echo "Directory created successfully"
	else
		echo "Directory creation failed"
		exit 1
fi

#Solve dependency package
yum install gcc gcc-c++ zlib-devel pcre-devel openssl-devel -y &> /dev/null || {
echo "Yum installation error"
exit 1
}

#Download the nginx source package
wget http://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz
if [ $? -ne 0 ]
	then
		echo "Download nginx source package failed"
		exit 1
	else
		echo "Download nginx source package successfully"
fi

#Compile and install nginx
tar xf nginx-${NGINX_VERSION}.tar.gz
if [ $? -ne 0 ]
	then
		echo "Unpack the nginx source package failed"
		eixt 1
	else
		echo "Unpack the nginx source package successfully"
		cd nginx-${NGINX_VERSION} || {
			echo "Failed to enter the nginx directory"
			exit 1
		} 
fi

#Create a nginx user
useradd -r nginx
#if [ $? -ne 0 ]
#	then
#		`rpm -qa | grep nginx`  &&   yum remove nginx -y
#fi

#Compile and install nginx
printf "
parameter\n
--prefix=/usr/local/nginx-${NGINX_VERSION}\n
--user=nginx\n
--group=nginx\n
--with-pcre\n
--with-http_ssl_module\n
--with-http_stub_status_module\n
"

./configure --prefix=/usr/local/nginx-${NGINX_VERSION} \
--user=nginx \
--group=nginx \
--with-pcre \
--with-http_ssl_module \
--with-http_stub_status_module

if [ $? -ne 0 ]
	then
		echo "configure failed"
		exit 1
fi

make && make install
if [ $? -ne 0 ]
	then
		echo "make && make install failed"
		exit 1
	else
		echo "make && make install successfully"
fi

$NGINXDIR/sbin/nginx && echo "Nginx started successfully"
if [ $? -ne 0 ]
	then
		echo "Nginx started failed"
fi
```