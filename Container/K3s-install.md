---
layout: default
title: K3s Cluster
parent: Container
nav_order: 10
---

# K3s v1.25.3+k3s1 Install
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}


k3s v1.25.3+k3s1 集群安装
--------------------------------------------


# 一、k3s

## 1、k3s简介

中文网站：http://docs.rancher.cn/docs/k3s/quick-start/_index/

k3s是经过CNCF认证的由Rancher公司开发维护的一个轻量级的 Kubernetes 发行版，内核机制还是和 k8s 一样，但是剔除了很多外部依赖以及 K8s 的 alpha、beta 特性，同时改变了部署方式和运行方式，目的是轻量化 K8s，简单来说，K3s 就是阉割版 K8s，消耗资源极少。它主要用于边缘计算、物联网等场景。K3s 具有以下特点：

1）安装简单，占用资源少，只需要512M内存就可以运行起来；
2）apiserver 、schedule 等组件全部简化，并以进程的形式运行在节点上，把程序都打包为单个二进制文件，每个程序只需要占用100M内存；
3）使用基于sqlite3的轻量级存储后端作为默认存储机制。同时支持使用etcd3、MySQL 和PostgreSQL作为存储机制；
4）默认使用 local-path-provisioner 提供本地存储卷；
5）默认安装了Helm controller 和 Traefik Ingress controller；
6）所有 Kubernetes control-plane 组件的操作都封装在单个二进制文件和进程中，使 K3s 具有自动化和管理包括证书分发在内的复杂集群操作的能力。
7）减少外部依赖，操作系统只需要安装较新的内核（centos7.6就可以，不需要升级内核）以及支持cgroup即可，k3s安装包已经包含了containerd、Flannel、CoreDNS，非常方便地一键式安装，不需要额外安装Docker、Flannel等组件。


## 2. k3s 架构

2.1、单节点的k3s架构

1）k3s server节点是运行k3s server命令的机器（裸机或者虚拟机），而k3s Agent 节点是运行k3s agent命令的机器。

2）单点架构只有一个控制节点（在 K3s 里叫做server node，相当于 K8s 的 master node），而且K3s的数据存储使用 sqlite 并内置在了控制节点上

3）在这种配置中，每个 agent 节点都注册到同一个 server 节点。K3s 用户可以通过调用server节点上的K3s API来操作Kubernetes资源。


2.1、高可用的K3S架构

虽然单节点 k3s 集群可以满足各种用例，但对于 Kubernetes control-plane 的正常运行至关重要的环境，可以在高可用配置中运行 K3s。一个高可用 K3s 集群由以下几个部分组成：

1）K3s Server 节点：两个或者更多的server节点将为 Kubernetes API 提供服务并运行其他 control-plane 服务

2）外部数据库：外部数据存储（与单节点 k3s 设置中使用的嵌入式 SQLite 数据存储相反）


# k3s 安装部署

因为k3s 是k8s 轻量级版本，常规基础环境基本一致。要求的硬件配置可以低点。

环境准备：

|集群角色|IP地址|主机命名|安装组件|配置|
|-------|------|-------|-------|----|
|server节点| 192.168.147.140| k3s-server |k3s server | 2c2g|
|agent节点| 192.168.147.141| k3s-agent1 |k3s agent | 2c2g|
|agent节点| 192.168.147.142| k3s-agent2 |k3s agent | 2c2g|

1、怎么系统参数

## 1、分别修改hosts文件

为的是统一服务器的hostname，也可以不设置，但是主机名不能一样，我是虚拟机克隆的，必须要改。

修改 k3s-server 主机名

```bash
hostnamectl set-hostname k3s-server
```

修改 k3s-agent1 主机名

```bash
hostnamectl set-hostname k3s-agent1
```

修改 k3s-agent2 主机名

```bash
hostnamectl set-hostname k3s-agent2
```

在**所有节点** `/etc/hosts` 添加主机名映射

```
192.168.147.140  k3s-server
192.168.147.141  k3s-agent1
192.168.147.142  k3s-agent2
```

然后分别在其中一台 ping 一下另外两台：

```
ping   k3s-server
ping   k3s-agent1
ping   k3s-agent2
```

> 如果没有特别说明在哪个节点执行，相关命令需要在所有节点执行。


## 2.关闭防火墙和selinux

关闭防火墙

```bash
systemctl stop firewalld && systemctl disable firewalld && iptables -F
```

关闭selinux

```bash
# 临时关闭
setenforce 0

# 永久关闭
sed -i 's/enforcing/disabled/' /etc/selinux/config
```

## 3. 关闭swap分区

临时关闭

```bash
#临时关闭
swapoff -a
#永久关闭swap
sed -ri 's/.*swap.*/#&/' /etc/fstab
```

## 4.修改系统参数


验证机器UUID

```bash
cat /sys/class/dmi/id/product_uuid
# 云机器 修改在网络配置里修改uuid
vi /etc/sysconfig/network-scripts/ifcfg-eth0
# 虚拟机一般是
vi cat /etc/sysconfig/network-scripts/ifcfg-ens33
```


简单修改内核参数

```bash
cat > /etc/sysctl.d/k3s_better.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

sysctl --system
```

对于K3S，需要确保系统中的如下module是加载状态的，
 - `ip_conntrack`
 - `br_netfilter`

可以通过在 /etc/modules-load.d/ 目录下，建立一个配置文件， 比如： k3s.conf


```bash
cat >> /etc/modules-load.d/k3s.modules << EOF
#!/bin/bash
modprobe -- ip_conntrack
modprobe -- br_netfilter
EOF

chmod +x /etc/modules-load.d/k3s.modules
sh /etc/modules-load.d/k3s.modules

lsmod | grep -e ip_conntrack -e br_netfilter
```


IPVS

```bash
yum install ipset ipvsadm -y
ipvsadm -l -n
```

centos  设置下次开机自动加载

```bash
cat >> /etc/sysconfig/modules/ipvs.modules << EOF
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_sh
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- nf_conntrack_ipv4
EOF

chmod +x /etc/sysconfig/modules/ipvs.modules
sh /etc/sysconfig/modules/ipvs.modules

lsmod | grep -e ip_vs -e nf_conntrack_ipv4
```



## 6.修改时区，同步时间


```bash
yum install chrond -y
vim /etc/chrony.conf
---txt
ntpdate ntp1.aliyun.com iburst
---
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
echo 'Asia/Shanghai' > /etc/timezone
```


# 二、安装 k3s 【脚本在线安装】

>从 v1.17.4+k3s1 开始，K3s 支持[自动升级](https://www.w3cschool.cn/tedsy/tedsy-oryn3pjz.html)。

## 1、安装 containerd 【所有节点】

```bash
yum install -y yum-utils
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum install containerd -y
systemctl start containerd && systemctl enable containerd
```

2、节点访问免密设置 【所有server节点】

实现 k3s-server 到集群内任意agent节点免密访问。

```bash
cd /root/.ssh/

# 执行命令，一路回车
ssh-keygen -t rsa -b 2048

# 如果只有较少数量的 k3-server 或者 k3s节点，可以执行（与下面二选一）
ssh-copy-id -p 22 root@k3s-agent1
```

多个节点使用for循环：

```bash
# 如果有多个节点需要免密，请参考，手动粘贴密码即可（与上面二选一）
for i in  k3s-server2 k3s-server3 k3s-agent1 k3s-agent2 k3s-agent3;do ssh-copy-id -p 22 root@$i  ;done
```


## 3、安装k3s server  【所有server节点】

版本在release 页面选择：https://github.com/k3s-io/k3s/releases


```bash

#安装最新版， 默认就是stable ，
curl -sfL https://get.k3s.io | INSTALL_K3S_CHANNEL=latest sh -
#安装最新版
curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=v1.25.3+k3s1 sh -
# 在 k3s server 上执行
curl -sfL http://rancher-mirror.cnrancher.com/k3s/k3s-install.sh | INSTALL_K3S_CHANNEL=latest  INSTALL_K3S_MIRROR=cn sh -

curl -sfL https://rancher-mirror.oss-cn-hangzhou.aliyuncs.com/k3s/k3s-install.sh | INSTALL_K3S_CHANNEL=latest  INSTALL_K3S_MIRROR=cn sh -
```

{: .warning }
> 我执行的`get.k3s.io`域名的，使用国内镜像安装不了，可以访问 `http://rancher-mirror.cnrancher.com/k3s/k3s-install.sh` 试试能不能打开，可以看到脚本内容就说明可以使用。

更多安装参数参考[k3s安装参数](https://docs.rancher.cn/docs/k3s/installation/install-options/_index/)

验证安装（跟k8s一样）

```bash
[root@k3s-server ~]# kubectl get node
NAME         STATUS   ROLES                  AGE     VERSION
k3s-server   Ready    control-plane,master   6m49s   v1.25.3+k3s1
[root@k3s-server ~]# kubectl get node -o wide
NAME         STATUS   ROLES                  AGE   VERSION        INTERNAL-IP       EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION           CONTAINER-RUNTIME
k3s-server   Ready    control-plane,master   10m   v1.25.3+k3s1   192.168.147.140   <none>        CentOS Linux 7 (Core)   3.10.0-1160.el7.x86_64   containerd://1.6.8-k3s1
[root@k3s-server ~]# kubectl get pods -n  kube-system
NAME                                      READY   STATUS      RESTARTS   AGE
coredns-75fc8f8fff-8qxqw                  1/1     Running     0          7m7s
local-path-provisioner-5b5579c644-rf8hk   1/1     Running     0          7m7s
metrics-server-5c8978b444-qvx67           1/1     Running     0          7m7s
helm-install-traefik-crd-nx952            0/1     Completed   0          7m7s
helm-install-traefik-tbkcn                0/1     Completed   2          7m7s
svclb-traefik-4fe23bf5-6zmwv              2/2     Running     0          6m5s
traefik-9c6dc6686-vnxb5                   1/1     Running     0          6m6s
[root@k3s-server ~]#
```

## 4、k3s集群添加work节点

需要先在k3s-server上获取 join token

```bash
# 提取join token
[root@k3s-server ~]# cat /var/lib/rancher/k3s/server/node-token
K108b359c44cd8f494ac630a6978819044b83aa85fff25c2e5f07ad7d2103eb9598::server:f1a9bcb9262fccd7a602fd4dbf2e6c38
```

跟 k8s 类似，使用token，在k3s-agent上执行命令，加入节点到集群 `K3S_URL`就是 `k3s-server`d的 `192.168.147.140`.

```bash
# 在 agent 的节点执行，安装并加入集群
[root@k3s-agent1 ~]# curl -sfL http://rancher-mirror.cnrancher.com/k3s/k3s-install.sh  | INSTALL_K3S_MIRROR=cn K3S_URL=https://192.168.147.140:6443 K3S_TOKEN=K108b359c44cd8f494ac630a6978819044b83aa85fff25c2e5f07ad7d2103eb9598::server:f1a9bcb9262fccd7a602fd4dbf2e6c38 sh -

[root@k3s-agent1 ~]# curl -sfL https://get.k3s.io |  K3S_URL=https://192.168.147.140:6443 K3S_TOKEN=K108b359c44cd8f494ac630a6978819044b83aa85fff25c2e5f07ad7d2103eb9598::server:f1a9bcb9262fccd7a602fd4dbf2e6c38 sh -

```


验证集群状态：

```bash
systemctl status k3s-agent -l

kubectl get nodes
```


其他worker节点想执行 `kubectl` 命令，做法和kubernetes一样的。

在worker 节点执行：

```bash
mkdir -p /etc/rancher/k3s && cd /etc/rancher/k3s
scp root@192.168.147.140:/etc/rancher/k3s/k3s.yaml .


# 修改访问 server 节点的IP,我记得写的好像是 127.0.0.1，改成server节点的IP
vim k3s.yaml

---yaml
	server: https:192.168.147.140:6443
---

echo "export KUBECONFIG=/etc/rancher/k3s/k3s.yaml" >> ~/.bash_profile
source ~/.bash_profile
```


验证：

```bash
[root@k3s-agent1 rancher]# kubectl get node
NAME         STATUS   ROLES                  AGE   VERSION
k3s-server   Ready    control-plane,master   55m   v1.25.3+k3s1
k3s-agent1   Ready    <none>                 13m   v1.25.3+k3s1
```




# 二、AirGap安装 k3s 【下载镜像离线安装】

（1）这里基础环境跟前面一样。
（2）安装 containerd 【所有节点】 (喜欢 docker 或其他容器运行时的都可以)
（3）server节点访问其他节点的免密设置


1、准备安装文件


（1）下载镜像包 https://github.com/k3s-io/k3s/releases/

 (2) 下载 k3s 二进制文件

（3）下载安装脚本


```txt

# 镜像文件地址
https://github.com/k3s-io/k3s/releases/download/v1.25.3+k3s1/k3s-airgap-images-arm64.tar

# 下载 k3s 二进制
https://github.com/k3s-io/k3s/releases/download/v1.25.3+k3s1/k3s

# 安装脚本，下载后命名为 k3s-ag-install.sh
https://get.k3s.io
```

将 tar 文件放在​images​目录下

```bash
sudo mkdir -p /var/lib/rancher/k3s/agent/images/
sudo cp ./k3s-airgap-images-arm64.tar /var/lib/rancher/k3s/agent/images/
sudo cp k3s /usr/local/bin/k3s
chmod +x /usr/local/bin/k3s
```

载入镜像到本地镜像仓库，也可放自己的私有仓库。


```bash
# docker 容器运行时
docker load -i k3s-airgap-images-amd64.tar

# containerd 容器运行时
crictl -n=k8s.io image import k3s-airgap-images-amd64.tar
```

2、执行安装

[K3S 安装配置参考](https://docs.rancher.cn/docs/k3s/installation/install-options/server-config/_index)

在安装之前了解一些安装时参数，这里列举一部分：

 -  –datastore-endpoint value：指定 etcd、Mysql、Postgres 或 Sqlite（默认）数据源名称
 -  –docker：用docker代替containerd
 -  –no-deploy value：不需要部署的组件 (有效选项: coredns, servicelb, traefik, local-storage, metrics-server)
 -  –write-kubeconfig value：将管理客户端的kubeconfig写入这个文件
 -  –write-kubeconfig-mode value：用这种模式编写kubeconfig，例如：644
 -  –tls-san value：在 TLS 证书中添加其他主机名或 IP 作为主题备用名称
 -  –node-ip value：为节点发布的 IP 地址
 -  –node-taint value：用一组污点注册kubelet（默认情况下，k3s 启动 master 节点也同时具有 worker 角色，是可调度的，因此可以在它们上启动工作，可以采用此方式解决）


**k3s-server 节点安装**

- 单节点模式

```bash

# docker 为容器运行时 多了一个 --docker 参数
INSTALL_K3S_SKIP_DOWNLOAD=true INSTALL_K3S_EXEC='server --write-kubeconfig ~/.kube/config --write-kubeconfig-mode 644  --docker --node-ip 152.136.181.95' ./k3s-ag-install.sh

# containerd 为容器运行时
INSTALL_K3S_SKIP_DOWNLOAD=true INSTALL_K3S_EXEC='server --write-kubeconfig ~/.kube/config --write-kubeconfig-mode 644 --node-ip 192.168.147.140' ./k3s-ag-install.sh

```

- 高可用模式

```bash
# 外部mysql高可用
INSTALL_K3S_SKIP_DOWNLOAD=true INSTALL_K3S_EXEC='server --write-kubeconfig ~/.kube/config --write-kubeconfig-mode 644 --node-ip 192.168.147.140  --datastore-endpoint='mysql://username:password@tcp(hostname:3306)/database-name'' ./k3s-ag-install.sh

# ETCD
INSTALL_K3S_SKIP_DOWNLOAD=true INSTALL_K3S_EXEC='server --write-kubeconfig ~/.kube/config --write-kubeconfig-mode 644 --node-ip 192.168.147.140  --datastore-endpoint='https://ETCD_IP:2379'' ./k3s-ag-install.sh
```

**k3s-agent 节点安装**

获取 server 节点 Token：K3S_TOKEN 是 Server 端的，位于 `/var/lib/rancher/k3s/server/node-token` 下

```bash
cat /var/lib/rancher/k3s/server/node-token
```

上传 k3s 二进制文件：

```bash
cp k3s /usr/local/bin/k3s
chmod +x /usr/local/bin/k3s
```

加入 server 集群

```bash
# 新增临时变量的模式传参
export INSTALL_K3S_SKIP_DOWNLOAD=true

export K3S_TOKEN=K108b359c44cd8f494ac630a6978819044b83aa85fff25c2e5f07ad7d2103eb9598::server:f1a9bcb9262fccd7a602fd4dbf2e6c38

export K3S_URL=https://192.168.147.140:6443

INSTALL_K3S_EXEC=' --write-kubeconfig ~/.kube/config --write-kubeconfig-mode 644' ./k3s-install.sh
```

也可以整理出一条命令:

```bash
INSTALL_K3S_SKIP_DOWNLOAD=true \
INSTALL_K3S_EXEC='--server https://192.168.147.140:6443 --token K108b359c44cd8f494ac630a6978819044b83aa85fff25c2e5f07ad7d2103eb9598::server:f1a9bcb9262fccd7a602fd4dbf2e6c38  --write-kubeconfig ~/.kube/config --write-kubeconfig-mode 644' \
./k3s-install.sh
```

