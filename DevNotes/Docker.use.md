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

docker 环境变量
--------------------------------------------
### 查看环境变量

```bash
docker inspect <CONTAINER-NAME> OR <CONTAINER-ID>
```

### 设置环境变量


(1) 在 Dockerfile 中使用 ENV 指令设置环境变量

格式有两种：

```yaml
    ENV key value
    ENV key1=value1 key2=value2
```
示例：

```text
1、key value
	ENV JAVA_VERSION 1.8_181
2、key=value
	ENV JAVA_VERSION=1.8_181  file.encoding=utf-8
3、换行
	ENV VERSION=1.0 DEBUG=on \
	    NAME="Happy Feet"
4、在 Dockerfile 中使用
	$JAVA_VERSION
    $file.encoding

```
(2)在 docker run 命令中设置环境变量

基本格式有：
- docker run --env -e
- docker run --env-file

示例1，直接在执行参数后添加：

```bash
docker run -e LANG=zh_cn.utf8 --env VAR2=value2  centos
docker run --env VAR1=value1 --env VAR2=value2 centos
```

示例2，先在本地配置好变量，执行时使用参数：

```bash
export LANG=zh_cn.utf8
export VAR2=value2
```

使用：

```yaml
$ docker run --env LANG --env VAR2 centos env | grep VAR
LANG=zh_cn.utf8
VAR2=value2
```
示例3 使用文件作为环境变量，`.env` 文件如下:

```
# This is a comment
LANG=zh_cn.utf8
VAR2=value2
USER  # which takes the value from the local environment
```
执行命令：

```bash
docker run --env-file .env centos
```

### Docker容器中文乱码问题

进容器

```bash
docker exec -it 44fc0f0582d9 /bin/bash
docker exec -it 44fc0f0582d9 /bin/sh
```

查看docker容器编码格式:

```bash
locale
```

查看容器所有语言环境

```bash
locale -a
```

中文支持的有：`C.UTF-8`、`zh_CN.utf8`、`zh_CN.UTF-8`,可能不同系统显示略有不同。

a.临时修改：

```text
export LANG=C.UTF-8
export LANG=zh_CN.utf8
```

b.在Dockerfile中添加一行环境变量参数后重新制作镜像

```yaml
  ENV LANG C.UTF-8
```

使用新的镜像启动容器后进入再次验证：`locale`.

c.增加 docker 启动参数

```bash
docker run -e LANG=zh_CN.utf8  <CONTAINER-ID>
docker run --env LANG=zh_CN.utf8  <CONTAINER-ID>
```
d.容器界面增加docker参数

在环境变量处增加键、值后重启容器：

```text
env  LANG=zh_CN.utf8
```

### Docker 启动报错问题

```text
docker: Error response from daemon: endpoint with name xxx-redis already exists in network bridge.
```

删除容器
```bash
docker rm xxx-redis
```

清理此容器的网络占用

格式：docker network disconnect --force 网络模式 容器名称

示例：
```bash
docker network disconnect --force bridge xxx-redis
```

 


Docker容器四种网络模式，自定义网络
--------------------------------------------

在没安装docker之前ifconfig命令是查看不到docker0的网卡的每运行一个容器就会生成一个veth对

- docker0：虚拟网关——>容器的网关，绑定物理网卡，负责做NAT 地址转换、端口映射
- loopback：回环网卡TCP/IP网卡是否生效
- veth对：一组互相连接的虚拟接口，用于连接两个网络/名称空间，网络协议栈1

### Docker四种网络模式

**host模式**

host 容器将不会虚拟出自己的网卡，配置自己的IP等，而是使用宿主机的IP和端口如果启动容器的使用host模式，那么这个容器将不会获得一个独立的Network Namespace，而是和宿主机共用一个Network Namespace。容器将不会虚拟出自己的网卡，配置自己的IP等，而是使用宿主机的IP和端口，但是，容器的其他方面，如文件系统、进程列表等还是和宿主机隔离的

使用host模式的容器也直接使用宿主机的IP地址与外界通信，容器内部的服务端口也可以使用宿主机的端口，不需要进行NAT，host最大的优势就是网络性能比较好，但是docker host 上已经使用的端口就不能再用了，网络隔离性不好

简单讲就是：和宿主机公用一个网段ip不重复，服务端口不要重复会冲突

**container模式**

container创建的容器不会创建自己的网卡、设置IP等个，而是和一个指定地容器共享IP、端口范围

这个模式指定新创建的容器和已经存在的一个容器共享一个 Network Namespace ，而是一个和宿主机共享，新创建的容器不会创建自己的网卡、配置自己的IP、而是和一个指定地容器共享IP、端口范围等。同样，两个容器除了网络方面，其他的如文件系统，进程列表还是隔离的。两个容器的进程可以通过lo网卡设备通信

简单讲就是：docker0为网关对接物理网卡然后对接docker0共用一个网段、端口范围，其他都隔离通过回环网卡通信，即多个容器共享网络名称空间

**None模式**

该模式关闭了容器的网络功能这种模式下容器只有回环网口，没有其他网卡none模式可以在容器创建时通过-network=none参数指定
这种类型的网络无法联网，但是封闭的网络能很好的保证容器的安全性

简单讲就是：单机，自己闹着玩吧隔离性绝对的强

**Bridge模式**

这个模式是默认的，它会为每一个容器分配、设置IP等，并将容器连接到一个Docker虚拟网桥，通过Docker0网桥及iptables的nat表配置与宿主机通信
当Docker进程启动时，会在主机上创建一个名为docker0的虚拟网桥，此主机上启动的Docker容器会连接到这个虚拟网桥上，虚拟网桥的工作方式和物理交换机类似，这样主机上的所有容器就通过交换机连在了一个二层网络中。

从docker0子网中分配一个IP给容器使用，并设置docker0的IP地址为容器的默认网关，在主机上创建一对虚拟网卡veth pair设备，docker将veth pair设备的一端放在新创建的容器并将这个网络设备加入到docker0网桥中，可以通过brctl show 命令查看

bridge模式时docker的默认网络模式，不写-net参数，就是bridge模式。使用docker run -p时，docker实际是在iptables做了DNAT规则，实现端口转发功能，可以使用iptables -t nat -vnL查看

默认模式 通过veth对连接容器与docker0网桥，网桥分配给容器IP，同时docker0作为docker内容器的网关，最后和宿主机网卡进行通讯

- host模式	-net=host	容器和宿主机共享Network namespace
- container模式	-net=container:NAME_or_ID	多个容器共享一个Network namespace
- none模式	-net=none	容器有独立的Network namespace，但并没有对其进行任何网络设置，如分配veth pair 和网桥连接，配置IP等
- bridge模式	-net=bridge	默认模式

以上不需要手动配置，真正需要配置的是自定义网络。


### Docker自定义网络

查看网络列表:

```bash
docker network ls
```
自定义网络固定IP
```bash
docker network create --subnet 172.18.0.0/16 mynetwork

docker run -itd --name test2 --net mynetwork --ip 172.18.0.100 centos:latest /bin/bash
```

在宿主机环境执行容器内命令 ` docker exec -it 容器ID /bin/bash -c 'nginx' `

-c后面跟的就是需要执行的命令

进入容器没有systemctl 命令解决：添加 `–privileged=true`（指定此容器是否为特权容器），使用此参数，则不能用attach
示例：
```bash
docker run -itd --name test3 --privileged=true centos /sbin/init
```

`/sbin/init` 内核启动时主动呼叫的第一个进程

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

# 给相关目录权限
chmod -R 777 /opt/docker/elk/elasticsearch

# es网络
docker network create es-network
# 内容
echo "http.host: 0.0.0.0" >> /opt/docker/elk/elasticsearch/config/elasticsearch.yml
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
docker exec -it elasticsearch717 bash 
./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.17.0/elasticsearch-analysis-ik-7.17.0.zip
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
如果找不到对应的IP，也可以直接加入 es-network 使用容器名称访问。


在 `kibana.yml` 文件添加 es 地址、用户名密码
```bash
vi   /opt/docker/elk/kibana/config/kibana.yml
```

添加配置：
```yaml
#
# ** THIS IS AN AUTO-GENERATED FILE **
#

# Default Kibana configuration for docker target
server.host: "0"
server.shutdownTimeout: "5s"
# IP访问
#elasticsearch.hosts: [ "http://172.17.0.3:9200" ]
# 容器名称访问
elasticsearch.hosts: [ "http://elasticsearch717:9200" ]
monitoring.ui.container.elasticsearch.enabled: true
i18n.locale: "zh-CN"
# 此处设置elastic的用户名和密码，有密码设置密码，没有就先忽略
#elasticsearch.username: elastic
#elasticsearch.password: elastic
```

**启动容器**

```bash
# 此处的IP换成ES的IP
docker run -d --name kibana717 -e ELASTICSEARCH_HOSTS=http://106.52.148.109:9200 -p 5601:5601 \
-v /opt/docker/elk/kibana/config/kibana.yml:/usr/share/kibana/config/kibana.yml \
--network es-network kibana:7.17.0
```

使用容器名称访问

```bash
# 此处的IP换成ES的IP
docker run -d --name kibana717 -e ELASTICSEARCH_HOSTS=http://elasticsearch717:9200 -p 5601:5601 \
-v /opt/docker/elk/kibana/config/kibana.yml:/usr/share/kibana/config/kibana.yml \
--network es-network kibana:7.17.0
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


