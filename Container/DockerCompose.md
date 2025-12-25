---
layout: default
title: Docker Compose
parent: Container
nav_order: 10
---

# Docker Compose Install
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---


Docker Compose 安装
---------------------------


## 使用仓库进行安装 docker-compose v1

1、安装仓库

此时可以安装Docker Compose。 通过执行以下命令下载当前版本：

```bash
# 下载指定版本的 Docker Compose V1
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# 赋予执行权限
sudo chmod +x /usr/local/bin/docker-compose

# 创建软链接（可选）
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

# 验证安装
docker-compose --version
```

> 网络不好就手工下载上传一下

```bash
tar -zxvf docker-compose-linux-x86_64.tar.gz
cp docker-compose-1.29.2  /usr/local/bin/docker-compose
# 赋予执行权限
sudo chmod +x /usr/local/bin/docker-compose

#检查Docker Compose版本
docker-compose -v
```


创建一个简单的 docker-compose.yml 文件

```bash
cat > docker-compose.yml << 'EOF'
version: '3.8'
services:
  nginx:
    image: nginx:alpine
    ports:
      - "8080:80"
EOF
```

常用命令：

```bash
# 启动服务
docker-compose up -d

# 查看服务状态
docker-compose ps

# 停止服务
docker-compose down
```

卸载  docker-compose v1

```bash
# 删除二进制文件
sudo rm /usr/local/bin/docker-compose

# 删除软链接
sudo rm /usr/bin/docker-compose

# 卸载 pip 安装的版本
sudo pip uninstall docker-compose
```

## 使用仓库进行安装 docker-compose v2

Docker Compose V1 和 V2 在核心功能上基本一致，但 V2 是更现代化的版本，推荐使用 V2。

主要区别

| 特性  | V1 | V2 |
|-----|---|---|
| 命令格式 |  docker-compose | docker compose（作为 Docker CLI 插件）|
|性能 |相对较慢 |启动速度更快，资源占用更少|
|兼容性 |需要单独安装 |集成在 Docker Desktop 中，Linux 需单独安装|
|功能 |基础功能 |新增 docker compose watch等新功能|
|版本 |1.x 系列 |2.x 系列|

两个版本可以共存。V2 的命令是 docker compose（注意没有连字符），V1 是 docker-compose。


命令安装

```bash
mkdir -p /opt/docker/cli-plugins
curl -SL "https://github.com/docker/compose/releases/download/v2.21.0/docker-compose-linux-x86_64" -o  /opt/docker/cli-plugins/
chmod +x /opt/docker/cli-plugins/docker-compose
docker compose version
```


系统插件安装
```bash
# 安装 Docker Compose 插件
sudo yum install docker-compose-plugin

# 验证安装
docker compose version
```

二进制安装

```bash
# 下载最新版 Docker Compose V2
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# 赋予执行权限
sudo chmod +x /usr/local/bin/docker-compose

# 创建软链接（可选）
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

# 验证安装
docker compose version
```

常用命令


```bash
# 启动服务
docker compose up -d

# 查看服务状态
docker compose ps

# 停止服务
docker compose down
```

卸载

```bash
# 卸载插件
sudo yum remove docker-compose-plugin

# 删除二进制文件
sudo rm /usr/local/bin/docker-compose
```