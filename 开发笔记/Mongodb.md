
## Mongodb 快速入门

### Mongodb 安装

> 在 centos 7 系统操作验证，其他环境未知。

#### 在线安装

1、配置MongoDB的yum源

创建yum源文件：
```bash
cd /etc/yum.repos.d 

vim mongodb-org-4.0.repo 
```
添加阿里云的源镜像

```
[mongodb-org-4.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.0/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-4.0.asc
```

这里可以修改 gpgcheck=0, 省去gpg验证

安装之前先更新所有包 ：

```bash
yum update
```

2、安装MongoDB


安装命令：

```bash
yum -y install mongodb-org
```

安装完成后

查看mongo安装位置 
```bash
whereis mongod
```

修改配置文件 ： 

```bash
vim /etc/mongod.conf
```

修改内容 `bindIp: 172.0.0.1`  改为 `bindIp: 0.0.0.0` 表示允许任意网址连接，如果固定几个IP就指定IP即可。

3、MongoDB 常用命令 

```bash
#启动mongodb
systemctl start mongod.service

#停止mongodb
systemctl stop mongod.service

#查到mongodb的状态：
systemctl status mongod.service
```

4、开放端口

```bash
# 开放27017端口
firewall-cmd --permanent --add-port=27017/tcp

# 移除端口
firewall-cmd --permanent --remove-port=27017/tcp

#重启防火墙(修改配置后要重启防火墙)
firewall-cmd --reload

#查看开放端口
firewall-cmd --list-ports
```

如果是云服务器，需要在控制台的安全组里添加一下需要开放的端口，包括出站和入站。


5、设置开机启动


./mongod  --dbpath=/usr/local/mongodb/dbs/ --logpath==/usr/local/mongodb/logs --fork

```bash
systemctl enable mongod.service
```


#### 离线安装

操作步骤：

```bash
# 下载
wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-4.0.13.tgz

# 解压
tar zxf mongodb-linux-x86_64-4.0.13.tgz

# 移动
mv ./mongodb-linux-x86_64-4.0.13/ /usr/local/mongodb

# 创建全局链接
cd /usr/local/mongodb/
ln -s /usr/local/mongodb/bin/mongo /usr/local/bin/mongo
ln -s /usr/local/mongodb/bin/mongod /usr/local/bin/mongod




# 进入根目录创建 data 文件夹，在 data 中再创建 db 文件夹和 log 文件夹
mkdir -p /data/db
mkdir -p /data/log

# 启动mongodb
mongod --dbpath /data/db --fork --logpath-/data/log/mongodb.log

#创建帐号
mongo admin --eval "db.createUser({ user: 'root', pwd: 'ancheng@123', roles: [ { role: 'root', db: 'admin' } , 'readWriteAnyDatabase'] })"

```

关闭mongodb 服务


```bash
ps -ef|grep mongo 

root 7220 1    0 17:25 ? 00:00:01 mongod --dbpath /data/db --fork --logpath /data/log/mongodb.log 
root 6268 3220 0 17:26 pts/1 00:00:00 grep --color=auto mongo

kill 7220
```

创建mongod.conf配置
```bash
vi /usr/local/mongodb/bin/mongod.conf
```
内容

```
systemLog:
  destination: file
  logAppend: true
  path: /data/log/mongodb.log # 日志文件
storage:
  dbPath: /data/db  # 数据目录
  journal:
    enabled: true
processManagement:
  fork: true  # fork and run in background
  timeZoneInfo: /usr/share/zoneinfo
net:
  port: 27017      # 端口
  bindIp: 0.0.0.0  # 绑定IP

```

创建mongodb服务
```bash
vi /lib/systemd/system/mongodb.service
```
内容：

```
[Unit]

Description=mongodb 
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
ExecStart=/usr/local/mongodb/bin/mongod --config /usr/local/mongodb/bin/mongod.conf
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/usr/local/mongodb/bin/mongod --shutdown --config /usr/local/mongodb/bin/mongod.conf
PrivateTmp=true

[Install]
WantedBy=multi-user.target

```

服务启停：
```bash
# 启动服务：
systemctl start mongodb.service

#停止服务：
systemctl stop mongodb.service

#开机启动：
systemctl enable mongodb.service
```


### Mongodb Shell

1、连接


查看版本

```bash
mongo --version
```
或者连接之后使用

```
db.version();
```

首次无密码连接：

```bash
mongo
```
进入shell终端就可以执行相关命令了。

带密码是连接：

**方式一**

```bash
mongo
use admin
db.auth('admin', '123456')
```

**方式二**


```bash
# 基本格式
mongo [database] -u [username] -p [password]

# 示例
mongo admin -u admin -p 123456
```


2、查看数据库：

```bash
show dbs
```

3、进入admin数据库

如果数据库不存在，则创建数据库，否则切换到指定数据库。

```bash
use admin

use test_demo
```

删除数据库

删除数据库，需要先切换到要删除的库下。

```bash
use test_demo
db.dropDatabase()
```

4、创建账户

创建管理员账户：

```bash
db.createUser({ user: "admin", pwd: "password", roles: [{ role: "userAdminAnyDatabase", db: "admin" }] })
```

mongodb中的用户是基于身份role的，该管理员账户的 role是 userAdminAnyDatabase。admin用户用于管理账号，不能进行关闭数据库等操作。


创建root超级用户

```bash
db.createUser({user: "root",pwd: "password", roles: [ { role: "root", db: "admin" } ]})
```

创建完admin管理员，创建一个超级管理员root。角色：root。root角色用于关闭数据库。

```bash
db.shutdownServer()
```

MongoDB 数据库默认角色

   - 数据库用户角色：read、readWrite
   - 数据库管理角色：dbAdmin、dbOwner、userAdmin
   - 集群管理角色：clusterAdmin、clusterManager、clusterMonitor、hostManager
   - 备份恢复角色：backup、restore
   - 所有数据库角色： readAnyDatabase、readWriteAnyDatabase、userAdminAnyDatabase、
    dbAdminAnyDatabase
   - 超级用户角色：root 


5、创建自己的库

创建一个自己的库，并指定角色，用户名，密码。

```bash
use xiaocai   # 跳转到需要添加用户的数据库,库不存在则会创建

db.createUser({ user: "user_ac", pwd: "AC_SF_123", roles: [{ role: "dbOwner", db: "bp" }] })
db.createUser({user:"user_bp",pwd:"bp123Ac",roles:[{role:"readWrite",db:"bp"}]})


db.createUser({
  user: 'small.cai',  # 用户名
  pwd: '123456',  # 密码
  roles:[{
    role: 'dbOwner',  # 角色
    db: 'xiaocai'  # 数据库名
  }]
})

show users

# 测试数据保存
db.xiaocai.insert({"id":1, "name":"测试数据保存"})


# 查询数据
db.xiaocai.find({"id":1})
```

role: "dbOwner"代表数据库所有者角色，拥有最高该数据库最高权限。比如新建索引等当账号管理员和超级管理员，可以为自己的数据库创建用户了。这时候一定要注意，如果想在`xiaocai`库上创建用户，一定要切换到所在数据库上去`use xiaocai`，再进行创建用户，不然创建的用户还是属于admin。


客户端连接测试：

浏览器访问

http://10.2.6.76:27017/

```
mongodb://user_ac:AC_SF_123@10.2.6.76:27017/bp
mongodb://small.cai:123456@server_ip:27017/bp
```


如果连接需要验证，则需要在配置中`vim /etc/mongod.conf`开启验证：

将

```bash
#security:
```

改为：

```bash
security:
  authorization: enabled
```  

创建完成之后使用 `show dbs` 查看，会看不到，需要插入一条数据才能看得到。

```bash
db.xiaocai.insert({"id":1,"name":"张小彩"})
```

6、管理账户


```bash
show users
```

删除用户

删除用户必须由账号管理员来删，所以，需要切换到admin角色。

```bash
use admin

# 登录、密码认证
db.auth('admin', '654321')  

# 修改用户密码
db.updateUser('admin', {pwd: '654321'})

# 删除
db.dropUser('testadmin')    


```


MongoDB 4.X 用户和角色权限管理

默认情况下，MongoDB实例启动运行时是没有启用用户访问权限控制的，也就是说，在实例本机服务器上都可以随意登录实例进行各种操作，MongoDB不会对连接客户端进行用户验证，可以想象这是非常危险的。为了强制开启用户访问控制(用户验证)，则需要在MongoDB实例启动时使用选项--auth或在指定启动配置文件中添加选项auth=true。

