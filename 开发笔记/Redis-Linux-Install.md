
## Redis-Linux 安装


离线安装

版本选择： 一般多用redis 5

如果要用redis 6 ,在执行 `gcc -v ` 检查一下 gcc 版本，redis 6 要求gcc 版本高于5。

选择一个你喜欢的版本：

```
http://download.redis.io/releases/
```

1.下载文件

```bash
wget http://download.redis.io/releases/redis-5.0.9.tar.gz

tar -zxvf redis-5.0.9.tar.gz

mv  redis-5.0.9  /usr/local/
```


2.检查gcc环境

```bash
whereis gcc

# 没有就安装
yum install -y gcc g++ gcc-c++
```


3.编译安装

```bash

cd /usr/local/redis-5.0.9

make

#如果make报错 就执行这个，否则跳过 
make MALLOC=libc

make  install PREFIX=/usr/local/redis-5.0.9

```



4、 处理配置

```bash
mkdir -p  /etc/redis

cd /usr/local/redis-5.0.9

cp redis.conf  /etc/redis/6379.conf
```



3、启动处理


复制启动脚本到 init.d
```bash
cd /usr/local/redis-5.0.9

cp utils/redis_init_script  /etc/init.d/redisd
```

改启动脚本参数

```bash
vi /etc/init.d/redisd
```


添加以下代码，:wq保存退出

添加
```
# chkconfig: 2345 10 90
# description: Start and Stop redisd
```

修改：
```
EXEC=/usr/local/bin/redis-server
CLIEXEC=/usr/local/bin/redis-cli
PIDFILE=/var/run/redis_${REDISPORT}.pid
```

修改之后：

```
#!/bin/sh
#
# chkconfig: 2345 10 90
# description: Start and Stop redisd
# Simple Redis ...

  ...

REDISPORT=6379

EXEC=/usr/local/redis-5.0.9/bin/redis-server
CLIEXEC=/usr/local/redis-5.0.9/bin/redis-cli

PIDFILE=/var/run/redis_${REDISPORT}.pid
CONF="/etc/redis/${REDISPORT}.conf"

 ...

```


增加脚本执行权限

```bash
chmod +x /etc/init.d/redisd
```

增加系统开机启动

```
chkconfig --add redisd

chkconfig --list redisd
```



测试
```bash
[root]# service redisd start
Starting Redis server...
15019:C 16 Aug 2021 20:49:34.165 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
15019:C 16 Aug 2021 20:49:34.165 # Redis version=5.0.9, bits=64, commit=00000000, modified=0, pid=15019, just started
15019:C 16 Aug 2021 20:49:34.165 # Configuration loaded
[root]# ps -aux | grep redis
```


如果可以正常启动之后，就可以配置后台守护进程启动。

配置后台运行

```bash
vim  /etc/redis/6379.conf
````
把守护模式设置成yes

```
daemonize  yes
```