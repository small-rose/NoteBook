---
layout: default
title: Docker Install
parent: Container
nav_order: 1
---

# Docker Install
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---


Docker 安装
---------------------------


## 使用 Docker 仓库进行安装

1、安装仓库

```bash
sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```

2、然后添加源：

使用官方源地址（在国外）
```bash
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

可以选择国内的一些源地址：

阿里云源

```bash
sudo yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

清华大学源

```bash
$ sudo yum-config-manager \
    --add-repo \
    https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/docker-ce.repo
```

3、安装

默认按照stable版本

```bash
sudo yum install docker-ce docker-ce-cli containerd.io -y
```
也可以指定版本安装。

a.先查看版本：

```bash
[root@VM-0-3-centos ~]# yum list docker-ce --showduplicates | sort -r
Repository epel is listed more than once in the configuration
 * remi-safe: ftp.riken.jp
 * remi-php74: ftp.riken.jp
Loading mirror speeds from cached hostfile
Loaded plugins: fastestmirror, langpacks
Installed Packages
docker-ce.x86_64            3:20.10.9-3.el7                    docker-ce-stable 
docker-ce.x86_64            3:20.10.8-3.el7                    docker-ce-stable 
docker-ce.x86_64            3:20.10.7-3.el7                    docker-ce-stable 
docker-ce.x86_64            3:20.10.6-3.el7                    docker-ce-stable 
docker-ce.x86_64            3:20.10.5-3.el7                    docker-ce-stable 
docker-ce.x86_64            3:20.10.4-3.el7                    docker-ce-stable 
docker-ce.x86_64            3:20.10.3-3.el7                    docker-ce-stable 
docker-ce.x86_64            3:20.10.2-3.el7                    docker-ce-stable 
docker-ce.x86_64            3:20.10.16-3.el7                   docker-ce-stable 
docker-ce.x86_64            3:20.10.16-3.el7                   @docker-ce-stable
```

b、指定版本。

通过其完整的软件包名称安装特定版本，该软件包名称是软件包名称（docker-ce）加上版本字符串（第二列），从第一个冒号（:）一直到第一个连字符，并用连字符（-）分隔。
例如：docker-ce-18.09.1。

```bash
sudo yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
```

4、启动 Docker

```bash
sudo systemctl start docker
```

5、验证
```
docker version
```

```bash
vi /etc/docker/daemon.json
{
 "data-root":"/var/lib/docker",
 "registry-mirrors" : [
   "https://mirror.ccs.tencentyun.com",
   "http://registry.docker-cn.com",
   "http://docker.mirrors.ustc.edu.cn",
   "http://hub-mirror.c.163.com"
 ],
 "insecure-registries" : [
   "registry.docker-cn.com",
   "docker.mirrors.ustc.edu.cn"
 ],
 "debug" : false,
 "experimental": false
}

```

6、卸载

```bash
systemctl stop docker
yum -y remove docker-ce
rm -rf /var/lib/docker
```

## docker compose 

继续安装docker compose

```bash
yum install  docker-compose-plugin
```

验证
```
[root@VM-0-3-centos ~]# docker compose version
Docker Compose version v2.5.0
```


自己打包镜像 jdk 镜像

```
docker build -t small/alpinejdk8.tar .
```


Dockerfile 文件：

```
# using alpine-glibc instead of alpine  is mainly because JDK relies on glibc
FROM alpine:3.17

LABEL maintainer="small-rose@qq.com"

ENV TZ=Asia/Shanghai

WORKDIR /app


# A streamlined jre
ADD  jdk-8u202-linux-x64.tar.gz  /usr/local/


# Alpine linux为了精简本身并没有安装太多的常用软件,apk类似于ubuntu的apt-get，
# 用来安装一些常用软V件，其语法如下：apk add bash wget curl git make vim docker
# wget是linux下的ftp/http传输工具，没安装会报错“/bin/sh: 　　wget: not found”，网上例子少安装wget
# ca-certificates证书服务，是安装glibc前置依赖
#******************更换Alpine源为mirrors.ustc.edu.cn******************
RUN ln -snf /usr/share/zoneinfo/${TZ} /etc/localtime && echo ${TZ} > /etc/timezone  \
	&& echo http://mirrors.aliyun.com/alpine/v3.17/main/ > /etc/apk/repositories && \
    echo http://mirrors.aliyun.com/alpine/v3.17/community/ >> /etc/apk/repositories \
	&& apk update && apk upgrade \
	&& apk --no-cache add libstdc++ ca-certificates bash wget \
    && wget -q -O /etc/apk/keys/sgerrand.rsa.pub https://alpine-pkgs.sgerrand.com/sgerrand.rsa.pub \
    && wget https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.35-r0/glibc-2.35-r0.apk \
    && wget https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.35-r0/glibc-bin-2.35-r0.apk \
    && wget https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.35-r0/glibc-i18n-2.35-r0.apk \
    && apk add --force-overwrite glibc-2.35-r0.apk && apk add glibc-bin-2.35-r0.apk && apk add glibc-i18n-2.35-r0.apk \
    && rm -rf /var/cache/apk/* glibc-2.35-r0.apk glibc-bin-2.35-r0.apk glibc-i18n-2.35-r0.apk \
    && rm -rf  /usr/local/jdk-8u202-linux-x64.tar.gz



# 配置环境变量
ENV JAVA_HOME /usr/local/jdk1.8.0_202
ENV JRE_HOME /usr/local/jdk1.8.0_202/jre

ENV CLASSPATH .:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

ENV PATH $JAVA_HOME/bin:$JRE_HOME/bin:$PATH

#容器启动时需要执行的命令
#CMD ["java","-version"]
```