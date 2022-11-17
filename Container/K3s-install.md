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

![单节点的k3s架构](../Assets/images/k3s-Danti.png)

1）k3s server节点是运行k3s server命令的机器（裸机或者虚拟机），而k3s Agent 节点是运行k3s agent命令的机器。

2）单点架构只有一个控制节点（在 K3s 里叫做server node，相当于 K8s 的 master node），而且K3s的数据存储使用 sqlite 并内置在了控制节点上

3）在这种配置中，每个 agent 节点都注册到同一个 server 节点。K3s 用户可以通过调用server节点上的K3s API来操作Kubernetes资源。


2.1、高可用的K3S架构

![高可用的K3S架构](../Assets/images/k3s-HA.png)

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
{: .tips }
> 注意，生产环境如果不能关闭防火墙，跟k8s一样,请依次开放所需默认端口，如（master）6443、2379-2380、2379-2380、10250、10257、10259,(worker)10250、30000-32767，也可以全部都自定义。


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




# 三、AirGap安装 k3s 【下载镜像离线安装】


## 1、基础环境准被

（1）这里基础环境跟前面一样。

（2）安装 containerd 【所有节点】 (喜欢 docker 或其他容器运行时的都可以)

（3）server节点访问其他节点的免密设置

## 2、准备安装文件


（1）下载镜像包 https://github.com/k3s-io/k3s/releases/

 (2) 下载 k3s 二进制文件

（3）下载安装脚本


```
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

## 3、执行安装

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

## 4、验证安装

```bash
systemctl status k3s -l
systemctl status k3s-agent -l
kubectl get node
kubectl get node -o wide
kubectl get svc
kubectl get pod -n kube-system
```


```bash
[root@k3s-agent2 k3s]# kubectl get node
NAME         STATUS   ROLES                  AGE   VERSION
k3s-server   Ready    control-plane,master   15h   v1.25.3+k3s1
k3s-agent1   Ready    <none>                 14h   v1.25.3+k3s1
k3s-agent2   Ready    <none>                 6m    v1.25.3+k3s1
[root@k3s-agent2 k3s]# kubectl get node -o wide
NAME         STATUS   ROLES                  AGE     VERSION        INTERNAL-IP       EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION           CONTAINER-RUNTIME
k3s-server   Ready    control-plane,master   15h     v1.25.3+k3s1   192.168.147.140   <none>        CentOS Linux 7 (Core)   3.10.0-1160.el7.x86_64   containerd://1.6.8-k3s1
k3s-agent1   Ready    <none>                 14h     v1.25.3+k3s1   192.168.147.141   <none>        CentOS Linux 7 (Core)   3.10.0-1160.el7.x86_64   containerd://1.6.8-k3s1
k3s-agent2   Ready    <none>                 6m20s   v1.25.3+k3s1   192.168.147.142   <none>        CentOS Linux 7 (Core)   3.10.0-1160.el7.x86_64   containerd://1.6.8-k3s1
[root@k3s-agent2 k3s]# kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.43.0.1    <none>        443/TCP   15h
[root@k3s-agent2 k3s]# kubectl get pod -n kube-system
NAME                                      READY   STATUS      RESTARTS      AGE
helm-install-traefik-crd-nx952            0/1     Completed   0             15h
helm-install-traefik-tbkcn                0/1     Completed   2             15h
traefik-9c6dc6686-vnxb5                   1/1     Running     1 (83m ago)   15h
svclb-traefik-4fe23bf5-6zmwv              2/2     Running     2 (83m ago)   15h
coredns-75fc8f8fff-8qxqw                  1/1     Running     1 (83m ago)   15h
local-path-provisioner-5b5579c644-rf8hk   1/1     Running     2 (82m ago)   15h
metrics-server-5c8978b444-qvx67           1/1     Running     2 (82m ago)   15h
svclb-traefik-4fe23bf5-99755              2/2     Running     2 (40m ago)   14h
svclb-traefik-4fe23bf5-v87ck              2/2     Running     0             14m
```

可以看到组件，如果不想用某个组件，比如不想用`traefik`，安装的时候添加` --disable traefik`即可 。



# 四、卸载 k3s

如果您使用安装脚本安装了 K3s，那么在安装过程中会生成一个卸载 K3s 的脚本。

>卸载 K3s 会删除集群数据和所有脚本。要使用不同的安装选项重新启动集群，请使用不同的标志重新运行安装脚本。

要从 server 节点卸载 K3s，请运行：

```bash
/usr/local/bin/k3s-uninstall.sh
```

要从 agent 节点卸载 K3s，请运行：

```bash
/usr/local/bin/k3s-agent-uninstall.sh
```


# 四、安装 rancher

## 安装 helm

### 1、自动安装

下载安装脚本

```bash
curl -fsSL -o install_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3

#设置权限
chmod 700 install_helm.sh

#执行脚本
./install_helm.sh
```

### 2、手动安装（针对网络不稳定）

下载

```bash
cd /opt/k3s-plugins/
wget https://get.helm.sh/helm-v3.2.4-linux-amd64.tar.gz


# 解压

tar -zxvf helm-v3.2.4-linux-amd64.tar.gz

# 移动 或者使用软连接

mv helm-v3.2.4-linux-amd64/helm  /usr/local/bin/helm
ln -s /opt/k3s-plugins/helm-v3.2.4-linux-amd64/helm  /usr/local/bin/helm
```

### 3、配置仓库

添加Helm Chart仓库

```bash
helm repo add rancher-stable https://releases.rancher.com/server-charts/stable
```


## 2、安装证书

由于Rancher Server 默认需要 SSL/TLS 配置来保证访问的安全性。

- 1.**Rancher 生成的自签名证书**： 在这种情况下，您需要在集群中安装cert-manager。 Rancher 利用cert-manager签发并维护证书。Rancher 将生成自己的 CA 证书，并使用该 CA 签署证书。然后，cert-manager负责管理该证书。
- 2.**Let's Encrypt**： Let's Encrypt 选项也需要使用cert-manager。但是，在这种情况下，cert-manager与特殊的 Issuer 结合使用，cert-manager将执行获取 Let's Encrypt 发行的证书所需的所有操作（包括申请和验证）。此配置使用 HTTP 验证（HTTP-01），因此负载均衡器必须具有可以从公网访问的公共 DNS 记录。
- 3.**使用自签名证书**： 此选项使您可以使用自己的权威 CA 颁发的证书或自签名 CA 证书。 Rancher 将使用该证书来保护 WebSocket 和 HTTPS 流量。在这种情况下，您必须上传名称分别为tls.crt和tls.key的 PEM 格式的证书以及相关的密钥。如果使用私有 CA，则还必须上传该证书。这是由于您的节点可能不信任此私有 CA。 Rancher 将获取该 CA 证书，并从中生成一个校验和，各种 Rancher 组件将使用该校验和来验证其与 Rancher 的连接。

由于我是虚拟机内网，使用自签名证书。

官方生成证书脚本 [一键生成 ssl 自签名证书脚本](https://docs.rancher.cn/docs/rancher2/installation/resources/advanced/self-signed-ssl/_index/)

```bash
#!/bin/bash -e

help ()
{
    echo  ' ================================================================ '
    echo  ' --ssl-domain: 生成ssl证书需要的主域名，如不指定则默认为www.rancher.local，如果是ip访问服务，则可忽略；'
    echo  ' --ssl-trusted-ip: 一般ssl证书只信任域名的访问请求，有时候需要使用ip去访问server，那么需要给ssl证书添加扩展IP，多个IP用逗号隔开；'
    echo  ' --ssl-trusted-domain: 如果想多个域名访问，则添加扩展域名（SSL_TRUSTED_DOMAIN）,多个扩展域名用逗号隔开；'
    echo  ' --ssl-size: ssl加密位数，默认2048；'
    echo  ' --ssl-cn: 国家代码(2个字母的代号),默认CN;'
    echo  ' 使用示例:'
    echo  ' ./create_self-signed-cert.sh --ssl-domain=www.test.com --ssl-trusted-domain=www.test2.com \ '
    echo  ' --ssl-trusted-ip=1.1.1.1,2.2.2.2,3.3.3.3 --ssl-size=2048 --ssl-date=3650'
    echo  ' ================================================================'
}

case "$1" in
    -h|--help) help; exit;;
esac

if [[ $1 == '' ]];then
    help;
    exit;
fi

CMDOPTS="$*"
for OPTS in $CMDOPTS;
do
    key=$(echo ${OPTS} | awk -F"=" '{print $1}' )
    value=$(echo ${OPTS} | awk -F"=" '{print $2}' )
    case "$key" in
        --ssl-domain) SSL_DOMAIN=$value ;;
        --ssl-trusted-ip) SSL_TRUSTED_IP=$value ;;
        --ssl-trusted-domain) SSL_TRUSTED_DOMAIN=$value ;;
        --ssl-size) SSL_SIZE=$value ;;
        --ssl-date) SSL_DATE=$value ;;
        --ca-date) CA_DATE=$value ;;
        --ssl-cn) CN=$value ;;
    esac
done

# CA相关配置
CA_DATE=${CA_DATE:-3650}
CA_KEY=${CA_KEY:-cakey.pem}
CA_CERT=${CA_CERT:-cacerts.pem}
CA_DOMAIN=cattle-ca

# ssl相关配置
SSL_CONFIG=${SSL_CONFIG:-$PWD/openssl.cnf}
SSL_DOMAIN=${SSL_DOMAIN:-'www.rancher.local'}
SSL_DATE=${SSL_DATE:-3650}
SSL_SIZE=${SSL_SIZE:-2048}

## 国家代码(2个字母的代号),默认CN;
CN=${CN:-CN}

SSL_KEY=$SSL_DOMAIN.key
SSL_CSR=$SSL_DOMAIN.csr
SSL_CERT=$SSL_DOMAIN.crt

echo -e "\033[32m ---------------------------- \033[0m"
echo -e "\033[32m       | 生成 SSL Cert |       \033[0m"
echo -e "\033[32m ---------------------------- \033[0m"

if [[ -e ./${CA_KEY} ]]; then
    echo -e "\033[32m ====> 1. 发现已存在CA私钥，备份"${CA_KEY}"为"${CA_KEY}"-bak，然后重新创建 \033[0m"
    mv ${CA_KEY} "${CA_KEY}"-bak
    openssl genrsa -out ${CA_KEY} ${SSL_SIZE}
else
    echo -e "\033[32m ====> 1. 生成新的CA私钥 ${CA_KEY} \033[0m"
    openssl genrsa -out ${CA_KEY} ${SSL_SIZE}
fi

if [[ -e ./${CA_CERT} ]]; then
    echo -e "\033[32m ====> 2. 发现已存在CA证书，先备份"${CA_CERT}"为"${CA_CERT}"-bak，然后重新创建 \033[0m"
    mv ${CA_CERT} "${CA_CERT}"-bak
    openssl req -x509 -sha256 -new -nodes -key ${CA_KEY} -days ${CA_DATE} -out ${CA_CERT} -subj "/C=${CN}/CN=${CA_DOMAIN}"
else
    echo -e "\033[32m ====> 2. 生成新的CA证书 ${CA_CERT} \033[0m"
    openssl req -x509 -sha256 -new -nodes -key ${CA_KEY} -days ${CA_DATE} -out ${CA_CERT} -subj "/C=${CN}/CN=${CA_DOMAIN}"
fi

echo -e "\033[32m ====> 3. 生成Openssl配置文件 ${SSL_CONFIG} \033[0m"
cat > ${SSL_CONFIG} <<EOM
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = clientAuth, serverAuth
EOM

if [[ -n ${SSL_TRUSTED_IP} || -n ${SSL_TRUSTED_DOMAIN} || -n ${SSL_DOMAIN} ]]; then
    cat >> ${SSL_CONFIG} <<EOM
subjectAltName = @alt_names
[alt_names]
EOM
    IFS=","
    dns=(${SSL_TRUSTED_DOMAIN})
    dns+=(${SSL_DOMAIN})
    for i in "${!dns[@]}"; do
      echo DNS.$((i+1)) = ${dns[$i]} >> ${SSL_CONFIG}
    done

    if [[ -n ${SSL_TRUSTED_IP} ]]; then
        ip=(${SSL_TRUSTED_IP})
        for i in "${!ip[@]}"; do
          echo IP.$((i+1)) = ${ip[$i]} >> ${SSL_CONFIG}
        done
    fi
fi

echo -e "\033[32m ====> 4. 生成服务SSL KEY ${SSL_KEY} \033[0m"
openssl genrsa -out ${SSL_KEY} ${SSL_SIZE}

echo -e "\033[32m ====> 5. 生成服务SSL CSR ${SSL_CSR} \033[0m"
openssl req -sha256 -new -key ${SSL_KEY} -out ${SSL_CSR} -subj "/C=${CN}/CN=${SSL_DOMAIN}" -config ${SSL_CONFIG}

echo -e "\033[32m ====> 6. 生成服务SSL CERT ${SSL_CERT} \033[0m"
openssl x509 -sha256 -req -in ${SSL_CSR} -CA ${CA_CERT} \
    -CAkey ${CA_KEY} -CAcreateserial -out ${SSL_CERT} \
    -days ${SSL_DATE} -extensions v3_req \
    -extfile ${SSL_CONFIG}

echo -e "\033[32m ====> 7. 证书制作完成 \033[0m"
echo
echo -e "\033[32m ====> 8. 以YAML格式输出结果 \033[0m"
echo "----------------------------------------------------------"
echo "ca_key: |"
cat $CA_KEY | sed 's/^/  /'
echo
echo "ca_cert: |"
cat $CA_CERT | sed 's/^/  /'
echo
echo "ssl_key: |"
cat $SSL_KEY | sed 's/^/  /'
echo
echo "ssl_csr: |"
cat $SSL_CSR | sed 's/^/  /'
echo
echo "ssl_cert: |"
cat $SSL_CERT | sed 's/^/  /'
echo

echo -e "\033[32m ====> 9. 附加CA证书到Cert文件 \033[0m"
cat ${CA_CERT} >> ${SSL_CERT}
echo "ssl_cert: |"
cat $SSL_CERT | sed 's/^/  /'
echo

echo -e "\033[32m ====> 10. 重命名服务证书 \033[0m"
echo "cp ${SSL_DOMAIN}.key tls.key"
cp ${SSL_DOMAIN}.key tls.key
echo "cp ${SSL_DOMAIN}.crt tls.crt"
cp ${SSL_DOMAIN}.crt tls.crt
```


复制官方提供的代码另存为create_self-signed-cert.sh或者其他您喜欢的文件名。


在自己的hosts文件里添加 192.168.147.140  rancher.demo.com


常用参数：

```
--ssl-domain: 生成ssl证书需要的主域名，如不指定则默认为www.rancher.local，如果是ip访问服务，则可忽略；
--ssl-trusted-ip: 一般ssl证书只信任域名的访问请求，有时候需要使用ip去访问server，那么需要给ssl证书添加扩展IP，多个IP用逗号隔开；
--ssl-trusted-domain: 如果想多个域名访问，则添加扩展域名（TRUSTED_DOMAIN）,多个TRUSTED_DOMAIN用逗号隔开；
--ssl-size: ssl加密位数，默认2048；
--ssl-cn: 国家代码(2个字母的代号),默认CN；
```

使用示例:
./create_self-signed-cert.sh --ssl-domain=rancher.demo.com --ssl-trusted-domain=rancher.demo.com \
--ssl-trusted-ip=192.168.147.140,192.168.147.141,192.168.147.142 --ssl-size=2048 --ssl-date=3650
```

验证生疏

## 3、安装rancehr


{: .important }
>Rancher 需要安装在受支持的 Kubernetes 版本上。要了解你的 Rancher 版本支持哪些版本的 Local Kubernetes，请参考[rancher 版本选择参考矩阵]（https://www.suse.com/suse-rancher/support-matrix/all-supported-versions/rancher-v2-6-9/) 或 [Rancher Release](https://github.com/rancher/rancher/releases)。
>
> 例如：Rancher v2.5.12 支持的 Kubernetes 版本包括 1.20、1.19、1.18 和 1.17。所以你的 Local Kubernetes 集群可以选择 1.20、1.19、1.18 或 1.17 版本。


在k3s中创建rancher的名名空间（Namespace）

```bash
kubectl create namespace myrancher-system
```
安装并rancher  如果您使用的是私有 CA 证书，请在命令中增加 --set privateCA=true。

```bash
helm install rancher rancher-latest/rancher \
      --namespace small-rancher \
      --set hostname=rancher.demo.com \
      --set ingress.tls.source=secret
```


验证安装：

```bash
kubectl -n small-rancher rollout status deploy/rancher
```

安装完成后使用域名访问 rancher : https://rancher.demo.com

