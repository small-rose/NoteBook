---
layout: default
title: MySQL Install windows
parent: MySQL
grand_parent: Database
nav_order: 81
---

# MySQL Install windows
{: .no_toc }

MySQL 相关的文档
{: .fs-6 .fw-300 }

## 安装方式

> MySQL 8.4 是 LTS 长期支持版，8.4.13 为当前稳定小版本。

官方下载页：https://dev.mysql.com/downloads/mysql/8.4.html

选择 Windows (x86, 64-bit)：

|包类型  |文件名  |适用场景|
|-------|-------|-------|
|MSI Installer |mysql-8.4.13-winx64.msi |新手 / 快速安装
|ZIP Archive |mysql-8.4.13-winx64.zip |开发环境 / 自定义路径

> ⚠️ 系统要求：Windows 10/11 64 位，管理员权限，路径不要含中文、空格、特殊符号。

## 方式一：ZIP 免安装版（推荐）

### 1、解压目录规划

```
D:\mysql\mysql-8.4.13-winx64
```
最终结构：

```
mysql-8.4.13-winx64
├── bin\
├── lib\
├── share\
├── my.ini      ← 手动创建
└── data\       ← 初始化后自动生成
```
### 2、创建 my.ini 配置文件

在解压根目录新建 my.ini

> 在解压根目录新建 my.ini，编码用 ANSI 或 UTF-8 无 BOM（别用带 BOM 的 UTF-8，否则可能报 unknown variable）。

```ini
[mysqld]
# 基础路径（按实际解压路径修改）
basedir=D:/mysql/mysql-8.4.13-winx64
datadir=D:/mysql/mysql-8.4.13-winx64/data

# 网络
port=3306
bind-address=127.0.0.1

# 字符集
character-set-server=utf8mb4
collation-server=utf8mb4_general_ci

# 存储引擎
default-storage-engine=INNODB

# 连接
max_connections=200

# Windows 开发环境建议（避免表名大小写坑）
lower_case_table_names=1

# 安全（开发环境可关，生产环境务必开启）
# require_secure_transport=ON

[client]
port=3306
default-character-set=utf8mb4

[mysql]
default-character-set=utf8mb4
```
> 💡 datadir 目录不要手动建，初始化时会自动生成；如果已存在非空 data，先删掉再初始化。

### 3、初始化数据目录（关键一步）

右键「开始菜单」→ CMD(管理员), 进入 bin 目录：

```bat
cd D:\mysql\mysql-8.4.13-winx64\bin

mysqld --initialize --console
```

执行后会看到类似输出：

> [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: Ks9#xQ2!aBz

⚠️ 务必复制保存这个临时密码（如 Ks9#xQ2!aBz），第一次登录要用。

    如果想让 root 初始无密码（仅本地开发），可用：

    mysqld --initialize-insecure --console

### 4、 注册 Windows 服务

> mysqld --install MySQL84 --defaults-file=D:\mysql\mysql-8.4.13-winx64\my.ini

看到 Service successfully installed. 即成功。

- 服务名 MySQL84 可自定义
- 卸载服务命令：mysqld --remove MySQL84


### 5、启停服务

```bat
net start MySQL84

net stop MySQL84
```
也可在 services.msc 里手动管理

### 6、登录并修改 root 密码

> mysql -u root -p

修改密码：

```sql
ALTER USER 'root'@'localhost'
IDENTIFIED BY 'MyNewPass@123456';

FLUSH PRIVILEGES;
```
> MySQL 8.4 默认使用 caching_sha2_password，兼容性更好也更安全，不建议退回到 mysql_native_password。

### 7、配置环境变量（可选但推荐）

系统变量新建：

> MYSQL_HOME = D:\mysql\mysql-8.4.13-winx64

Path 追加：

> %MYSQL_HOME%\bin

验证：

> mysql --version

输出应为：

> mysql  Ver 8.4.13 for Win64 on x86_64 (MySQL Community Server - GPL)


### 8、创建开发用普通账号（生产/规范做法）

不要长期使用root跑业务

```sql
CREATE USER 'dev'@'localhost' IDENTIFIED BY 'Dev@123456';
GRANT ALL PRIVILEGES ON *.* TO 'dev'@'localhost';
FLUSH PRIVILEGES;
```

## 方式二：MSI 图形化安装（快速版）

### 1、双击 mysql-8.4.13-winx64.msi
   以管理员身份运行。
### 2、安装类型
   选 Custom 或 Developer Default。
### 3、关键配置项

- Type and Networking：默认端口 3306
- Authentication Method：选 Use Strong Password Encryption（默认）
- Accounts and Roles：
   - 设置 root 密码（别太简单，如 Admin@123456）
   - 可顺手建一个 dev 用户

Windows Service：

- 勾选 "Configure MySQL Server as a Windows Service"
- 服务名默认 MySQL84
- 勾选 "Start at system startup"

### 4、完成验证

```
mysql -u root -p
SELECT VERSION();
```

使用连接工具（Navicat / IDEA / DBeaver）测试。



## 常见坑与排查

1、mysqld --initialize 报错 unknown variable

> my.ini 编码问题 → 改成 ANSI
> 
> 路径用了反斜杠 \` 且未转义 → 建议用 / 或 \`

2. 服务启动失败：系统找不到指定文件

> basedir / datadir 写错
> 
> 没用管理员 CMD
> 
>之前装过旧服务：先 mysqld --remove MySQL84 再重装

3. 3306 端口占用

> netstat -ano | findstr 3306
>
>改 my.ini 里的 port=3307 然后重启服务。

4. 登录提示 Access denied

> 临时密码复制少了字符
>
> 用 --initialize-insecure 重来（开发环境最快）

5. 表名大小写敏感
 
>建表时 User 和 user 被认为不同 → my.ini 加：
>
> > lower_case_table_names=1

这个参数只能在初始化前设置，改了要删 data 重新初始化。