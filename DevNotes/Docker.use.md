---
layout: default
title: Docker Use
parent: DevNotes
nav_order: 5
---

# Docker Use
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}


## 固定IP配置

```bash
vi /etc/sysconfig/network-scripts/ifcfg-ens33 
```

```text
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=none
DEFROUTE=yes
IPV5_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=9cf4f495-18f1-4c56-80b5-d433c5273d37
DEVICE=ens33
ONBOOT=yes
IPADDR=192.168.80.81
GATEWAY=192.168.80.1
IPV6_PRIVACY=no
PREFIX=24
IPV4_FAILURE_FATAL=no

REERDNS=no
DNS1=192.168.80.1
DNS2=8.8.8.8
DNS3=223.6.6.6
DNS4=114.114.114.114
```


docker UI
-----------------------

## Portainer.io

**搜索**

```bash
docker search portainer
```

**下载**

```bash
docker pull portainer/portainer　
```
　　 
**启动**

```bash
docker run -d -p 9000:9000 --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data --name prtainer-zzy portainer/portainer
```

参数说明：

```bash
docker run -d 　　　　　　　　# 后台运行容器
　　-p 9000:9000 　　　　　　 # 默认9000端口，映射到宿主机，通过本地地址访问
　　--name prtainer-test 　　　# 指定容器名
　　--restart=always 　　　　  # 设置自动启动
    -v /opt/portainer:/data     # 保存portainer数据到宿主机
    -v /var/run/docker.sock:/var/run/docker.sock 　　# 单机方式必须指定
　　portainer/portainer　
```

**访问**

```
http://192.168.80.81:9000/#!/home
admin
1qaz2wsx
```
## Redis 


**下载**

```bash
docker pull redis
```

测试启动，无密码

```bash
docker run -itd --name redis-test -p 6379:6379 redis
```

正式启动，持久化容器卷
 (1) 持久化配置文件
 (2) 持久化数据目录
 (3) 密码等其他参数
 
```bash
docker run --restart=always --log-opt max-size=100m --log-opt max-file=2 -p 6379:6379 --name myredis -v /home/redis/myredis/myredis.conf:/etc/redis/redis.conf -v /home/redis/myredis/data:/data -d redis redis-server /etc/redis/redis.conf  --appendonly yes  --requirepass 123456
```
 - –restart=always 总是开机启动
 - –log是日志方面的
 - -p 6379:6379 将6379端口挂载出去
 - –name 给这个容器取一个名字
 - -v 数据卷挂载

    /opt/redis/myredis/myredis.conf:/etc/redis/redis.conf 这里是将 liunx 路径下的myredis.conf 和redis下的redis.conf 挂载在一起。
    /opt/redis/myredis/data:/data 这个同上

  - -d redis 表示后台启动redis。redis-server /etc/redis/redis.conf 以配置文件启动redis，加载容器内的conf文件，最终找到的是挂载的目录 /etc/redis/redis.conf 也就是liunx下的/home/redis/myredis/myredis.conf
  - –appendonly yes 开启redis 持久化
  - –requirepass 000415 设置密码 

如果忘记密码了，可以进入容器查看

```bash
docker exec -it myredis redis-cli
# 密码登录，如果忘记密码，可以跳过
auth 密码
# 查看是否设置密码
config get requirepass
```

## Mysql 安装

**下载**

```bash
docker pull mysql:5.7.39
```
　　
**启动**

```bash
docker run --name mysql01 -d -p 3310:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 mysql:5.7.39
```

解析：

```text
--name mysql-sr                                #  对容器的命名
-d                                             # 后台运行
-p 3310:3306                                   #对外暴露端口号3310
-v /home/mysql/conf:/etc/mysql/conf.d     #配置文件挂载到当前宿主机的/home/mysql/conf
-v /home/mysql/data:/var/lib/mysql            #数据挂载到当前宿主机的 /home/mysql/data
-e MYSQL_ROOT_PASSWORD=123456    #设置mysql的root用户的密码是：·123456
```

也可以界面部署

ENV:

```text
MYSQL_ROOT_PASSWORD   12345678
```

## oracle_11g

在 dockerHub 上可以搜 `oracle` 或 `oracle_11g`

使用阿里镜像

```bash
docker pull  docker pull registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g
```
使用 `docker images` 查看镜像


启动容器

默认启动容器的方式
```bash
docker run -d -it -p 1521:1521 --name oracle11g --restart=always registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g
```

持久化启动的方式

```bash
docker run -d -it -p 1521:1521 --name oracle11g --restart=always --mount source=oracle_vol,target=/home/oracle/app/oracle/oradata registry.cn-hangzhou.aliyuncs.com/helowin/oracle_11g
```

查看启动的线程 

```bash
netstat -tulnp
```
可以看到1521端口的映射。

容器内环境配置

进入容器 
```bash
docker exec -it oracle11g bash
```

切换到 root 用户 su root，密码为 helowin
```bash
su root
```

docker容器配置环境变量不是在 /etc/profile 中，容器启动不会走这个文件。

可以将环境变量的配置设置在 /home/oracle/.bashrc 文件下，这样可以省略掉软连接的创建 `ln -s $ORACLE_HOME/bin/sqlplus /usr/bin`

编辑环境变量 
```bash
vi /home/oracle/.bashrc
```

在文件最后加入以下命令

```text
export ORACLE_HOME=/home/oracle/app/oracle/product/11.2.0/dbhome_2

export ORACLE_SID=helowin

export PATH=$ORACLE_HOME/bin:$PATH
```


wq 保存并退出。然后使用 

```bash
source /home/oracle/.bashrc 
```
刷新环境变量，并使之生效

进入 oracle 命令行

使用 

```bash
sqlplus /nolog 
```
进入oracle命令行

登录oracle

```bash
conn / as sysdba
```
 
如果直接使用默认的 root 用户登录，会报登录失败。这里必须使用 su - oracle 命令，将当前用户切换到 oracle，然后在执行登录命令

ORA-12514, TNS:listener does not currently know of service requested in connect descriptor

这个错误是由于数据库名用错了

```bash
su - oracle

sqlplus /nolog

conn / as sysdba
```

```oraclesqlplus
select instance_name from v$instance;

show user;
```
使用上述命令查出来的，就是所有可用的 “数据库名” 和 “用户名”


> 阿里的这个镜像，所有的密码都是统一的 **helowin**

system用户具有DBA权限，但是没有SYSDBA权限。平常一般用该帐号管理数据库。
而sys用户是Oracle数据库中权限最高的帐号，具有“SYSDBA”和“SYSOPER”权限，一般不允许从外部登录

使用system连接测试：

```yml
url: jdbc:oracle:thin:@localhost:1521:ORCL
username: system
password: helowin
```
实际使用过程中，可以创建用户，不同的用户表是相互隔离的。


**配置防火墙**

防火墙要允许 1521 端口，外部的数据库管理工具才能连的上


```bash
#打开防火墙
systemctl start firewalld
service firewalld status
#查询端口状态
firewall-cmd --query-port=1521/tcp

#永久性开放端口
firewall-cmd --permanent --zone=public --add-port=1521/tcp

#重启防火墙
firewall-cmd --reload

firewall-cmd --query-port=1521/tcp
```

## rabbitMQ

**下载**

```bash
docker pull rabbitmq:3.10.7-management
```

**启动**

```bash
docker run -di --name rabbitmq -p 5672:5672 -p 15672:15672  rabbitmq:3.10.7-management
```

```bash
docker run -d --name rabbitmq3.9.3 -p 5672:5672 -p 15672:15672 -v `pwd`/data:/var/lib/rabbitmq --hostname  -e RABBITMQ_DEFAULT_VHOST=/  -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin rabbitmq:3.10.7-management
```

如果不想特别说明带上版本信息 management 也可以直接拉取最新的，然后启动容器：

```bash
docekr pull rabbitmq
docker run -d --hostname my-rabbitmq --name rabbitmq -p 15672:15672 -p 5672:5672 rabbitmq
```

然后可以进入容器后执行以下命令启动界面程序：

（1）通过界面进入容器内部bash控制台

（2）通过 `docker ps -a` 查看部署的mq容器id，在通过 `docker exec -it 容器id /bin/bash `进入容器内部

```bash
rabbitmq-plugins enable rabbitmq_management
```
访问界面程序

```text
http://192.168.80.81:15672/#/
guest/guest
```
验证启动服务

```bash
curl -v  127.0.0.1:5672
```

https://www.modb.pro/db/392483

https://blog.csdn.net/weixin_39609953/article/details/110235188


## ElasticSearch 7.8.0

**下载**

```bash
docker pull elasticsearch:7.8.0
```

创建映射目录

```bash
mkdir -p /opt/elk/elasticsearch/config
mkdir -p /opt/elk/elasticsearch/data
mkdir -p /opt/elk/elasticsearch/plugins
chmod -R 777 /opt/elk/elasticsearch
```

**启动**

```bash
echo "http.host: 0.0.0.0" >> /docker/elk/elasticsearch/config/elasticsearch.yml

docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300  -e "discovery.type=single-node" \
-e ES_JAVA_OPTS="-Xms128m -Xmx128m" \
-v /docker/elk/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
-v /docker/elk/elasticsearch/data:/usr/share/elasticsearch/data \
-v /docker/elk/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
elasticsearch:7.8.0
```
其中 `elasticsearch.yml` 是挂载的配置文件，data是挂载的数据，plugins是es的插件，如ik，而数据挂载需要权限，需要设置data文件的权限为可读可写,需要下边的指令。
`chmod -R 777 /opt/elk/elasticsearch/data`

-e “discovery.type=single-node” 设置为单节点

-v 是挂载磁盘映射宿主机

注意：

-e ES_JAVA_OPTS="-Xms256m -Xmx256m" 测试环境下，设置ES的初始内存和最大内存，否则导致过大启动不了ES

重复执行需要先删除容器 `docker rm -f elasticsearch`

验证启动

```text
# 查看启动日志
docker logs -f elasticsearch
# 查看结果
curl http://192.168.10.184:9200
```

其他如果涉及防火墙，开启9200、9300端口


## docker 安装 ELK

### 安装 ES 7.17.0

拉取镜像

```
docker pull docker.io/elasticsearch:7.17.0
docker pull elasticsearch:7.17.0
```

映射目录卷

```bash
# Elasticsearch配置文件
mkdir -p /opt/docker/elk/elasticsearch/config
# 数据目录
mkdir -p /opt/docker/elk/elasticsearch/data
# 插件目录
mkdir -p /opt/docker/elk/elasticsearch/plugins
```

启动容器

```bash
docker run -d --name elasticsearch717 -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms512m -Xmx512m" \
-v /opt/docker/elk/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
-v /opt/docker/elk/elasticsearch/data:/usr/share/elasticsearch/data \
-v /opt/docker/elk/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
--privileged --network es-network  elasticsearch:7.17.0

```


安装 ik 分词器

```bash
docker exec -it es bash ./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.17.0/elasticsearch-analysis-ik-7.17.0.zip
```

设置密码

开启 X-Pack
```yaml
cluster.name: "docker-cluster-01"
network.host: 0.0.0.0
http.cors.enabled: true
http.cors.allow-origin: "*"
# 此处开启xpack
xpack.security.enabled: true 
```

重新启动elasticsearch。
```bash
docker restart elasticsearch717
```

### ES密码相关


进入docker中的elasticsearch中，设置密码，执行

```bash
docker exec -it es bash ./usr/share/elasticsearch/bin/x-pack/setup-passwords interactive
```

验证

```bash
curl localhost:9200 -u elastic
```

ES 修改密码

```bash
POST _xpack/security/user/_password
POST _xpack/security/user/<username>/_password
# 将用户elastic  密码改为elastic
curl -u elastic -H "Content-Type: application/json" -X POST "localhost:9200/_xpack/security/user/elastic/_password" --data '{"password":"elastic"}'
# 测试是否修改成功
curl localhost:9200 -u elastic
```


### 安装 logstash 7.17.0

> 看dockerhub 里有 7.17.5 可以试试 

```bash
# 拉取镜像
docker pull logstash:7.17.0

#logstash 配置文件
mkdir -p /opt/docker/elk/logstash/config
mkdir -p /opt/docker/elk/logstash/conf.d

docker run -it -d -p 5044:5044 --name logstash  --net host \
-v /opt/docker/elk/logstash/config/logstash.yml:/usr/share/logstash/config/logstash.yml \
-v /opt/docker/elk/logstash/conf.d/:/usr/share/logstash/conf.d/ logstash:7.17.0

```
生产环境建议使用 --restart="always"

### 安装 kibana 7.17.0


```bash
# 拉取镜像
docker pull kibana:7.17.0

# kibana 配置文件
mkdir -p /opt/docker/elk/kibana/config
```

查询 elasticsearch717 的容器IP地址，关键词 `IPADRESS`

```bash
 docker inspect elasticsearch717
```

```bash
# 此处的IP换成ES的IP
docker run -d --name kibana717 -e ELASTICSEARCH_HOSTS=http://106.52.148.109:9200 -p 5601:5601 \
-v /opt/docker/elk/kibana/config/kibana.yml:/usr/share/kibana/config/kibana.yml \
kibana:7.17.0
```

访问验证 http://192.168.80.81:5601

**修改配置**
```bash
# 进入容器内部
docker exec -it kibana /bin/bash		
# 挂载目录
vi /opt/docker/elk/kibana/config/kibana.yml		
```

在 `kibana.yml` 文件添加 es 地址、用户名密码
```yaml
#
# ** THIS IS AN AUTO-GENERATED FILE **
#

# Default Kibana configuration for docker target
server.host: "0"
server.shutdownTimeout: "5s"
elasticsearch.hosts: [ "http://172.17.0.3:9200" ]
monitoring.ui.container.elasticsearch.enabled: true
i18n.locale: "zh-CN"
# 此处设置elastic的用户名和密码
elasticsearch.username: elastic
elasticsearch.password: elastic
```

重启Kibana

```bash
docker restart kibana
```

## ELK常见问题

第一点：KB、ES版本不一致（网上大部分都是这么说的）

    解决方法：把KB和ES版本调整为统一版本

第二点：kibana.yml中配置有问题（通过查看日志，发现了Error: No Living connections的问题）

    解决方法：将配置文件kibana.yml中的elasticsearch.url改为正确的链接，默认为: http://elasticsearch:9200

    改为http://自己的IP地址:9200,如果存在密码的话需要加上下面两句话

    elasticsearch.username: "elastic"

    elasticsearch.password: "123456"

第三点：浏览器没有缓过来

    解决方法：刷新几次浏览器。

第四点： es磁盘空间满会导致只读 问题（ES 写索引报错 FORBIDDEN/12/index read-only / allow delete (api）

    解决方案： https://blog.csdn.net/zheng45/article/details/92383323 

第5点：es密码不能包含`@`符号的因为连接的时候其实是拼接的url,会导致冲突   http://user:pass@localhost:9200  

    第6点；es加了密码验证之后命令基本带上-u elastic   ，只有7.3版本以上的免费使用密码认证


