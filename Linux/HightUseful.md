---
layout: default
title: Linux Command
parent: Linux
nav_order: 11
---

# Linux Command
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Centos-7.repo

```bash

mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup && \
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo && \
yum clean all  && yum makecache && yum update 
```

## docker

```bash
yum install -y yum-utils   device-mapper-persistent-data  lvm2 

yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

yum install docker-ce docker-ce-cli containerd.io -y

vi  /etc/docker/daemon.json

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
EOF

systemctl enable --now  docker
```

## tar

```bash

# 解压
tar -xvf abc.tar
tar -zxvf abc.tar.gz

# 压缩
tar cvf  abc.tar file1 fil2 
tar zcvf  abc.tar,gz  file1 file2
```

## firewall

```bash
firewall-cmd --permanent --add-port=80/tcp  && firewall-cmd --reload
firewall-cmd --permanent --add-port=8080/tcp  && firewall-cmd --reload
firewall-cmd --permanent --add-port=8081/tcp  && firewall-cmd --reload
firewall-cmd --permanent --add-port=8096/tcp  && firewall-cmd --reload
firewall-cmd --permanent --add-port=8848/tcp  && firewall-cmd --reload

firewall-cmd --permanent --add-port=9000/tcp  && firewall-cmd --reload


firewall-cmd --permanent --add-port=3306/tcp  && firewall-cmd --reload
firewall-cmd --permanent --add-port=1521/tcp  && firewall-cmd --reload
firewall-cmd --permanent --add-port=6379/tcp  && firewall-cmd --reload
firewall-cmd --permanent --add-port=27017/tcp  && firewall-cmd --reload

```

## nginx

```bash

mkdir -p /opt/docker-vdata/nginx/{conf/conf.d,html,log}
docker run -d --name nginx-test nginx:1.22.1

docker cp nginx-test:/etc/nginx/nginx.conf  /opt/docker-vdata/nginx/conf/nginx.conf
docker cp nginx-test:/etc/nginx/conf.d/default.conf  /opt/docker-vdata/nginx/conf/conf.d/default.conf
docker cp nginx-test:/usr/share/nginx/html/index.html /opt/docker-vdata/nginx/html/index.html
docker cp nginx-test:/usr/share/nginx/html/index.html /opt/docker-vdata/nginx/html/index.html


docker run -p 80:80 --name nginx \
    -v /opt/docker-vdata/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
    -v /opt/docker-vdata/nginx/conf/conf.d:/etc/nginx/conf.d \
    -v /opt/docker-vdata/nginx/log:/var/log/nginx \
    -v /opt/docker-vdata/nginx/html:/usr/share/nginx/html  \
    --restart always  -d nginx:1.22.1

docker run -p 80:80 --name nginx-php \
    -v /opt/docker-vdata/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
    -v /opt/docker-vdata/nginx/conf/conf.d:/etc/nginx/conf.d \
    -v /opt/docker-vdata/nginx/log:/var/log/nginx \
    -v /opt/docker-vdata/nginx/html:/usr/share/nginx/html  \
    --privileged=true --link php-prod:php  --restart always  \
    -d nginx:1.22.1

```

## php 7.4

```bash
docker pull mailcow/phpfpm:1.69

docker run --name php-test -d mailcow/phpfpm:1.69


docker run --name php-prod -v /opt/docker-vdata/nginx/html:/www -p 9001:9000 -d mailcow/phpfpm:1.69

docker run --name php-prod --ip 172.17.0.4 -v /opt/docker-vdata/nginx/html:/var/www/html -p 9002:9000 -d mailcow/phpfpm:1.69

docker exec -it php-prod /bin/bash

cd /usr/local/bin/
./docker-php-ext-install pdo_mysql
```


## redis

```bash
docker run -d --name redis-7 

mkdir -p /opt/docker-vdata/redis/data 

docker run -d --name redis-test  redis 
docker cp redis-test:/etc/redis/redis.conf  /opt/docker-vdata/redis/redis.conf
docker stop redis-test  && docker rm redis-test && ls /opt/docker-vdata/redis

docker run -d --restart=always -p 6379:6379 --name redis-7 -v /opt/docker-vdata/redis/data:/data -v /opt/docker-vdata/redis/redis.conf:/etc/redis/redis.conf redis 
```

## mongoDB

```bash
mkdir -p /u01/data/mongodb_data/{data,conf,log}

docker run -d --name mongoDB --privileged=true --restart always -p 27017:27017 -v /u01/data/mongodb_data/data:/data/db -v /u01/data/mongodb_data/conf/:/data/conf -v /u01/data/mongodb_data/log:/data/log -e MONGO_INITDB_ROOT_USERNAME=admin  -e MONGO_INITDB_ROOT_PASSWORD=Fnd123456  mongo:4.0.5
```




## jellyfin

```bash

mkdir -p /opt/docker-vdata/jellyfin/{conf,data}

# 启动jellyfin容器
docker run -d --restart=always \
--name=jellyfin \
-p 8096:8096 -p 7359:7359/udp -p 1900:1900/udp \
-v /opt/docker-vdata/jellyfin/conf:/config -v /opt/docker-vdata/jellyfin/data:/media \
--add-host api.themoviedb.org:52.84.125.121 \
linuxserver/jellyfin:latest
```
http://192.168.147.100:8096