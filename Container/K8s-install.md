---
layout: default
title: Kubernetes Cluster
parent: Container
nav_order: 10
---

# Kubernetes  Cluster V1.25.3 Install
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}


k8s 1.25.x 集群安装
--------------------------------------------


# 一、基本环境

官方基本要求

- 一台兼容的 Linux 主机。Kubernetes 项目为基于 Debian 和 Red Hat 的 Linux 发行版以及一些不提供包管理器的发行版提供通用的指令。
- 每台机器 2 GB 或更多的 RAM（如果少于这个数字将会影响你应用的运行内存）。
- CPU 2 核心及以上。
- 集群中的所有机器的网络彼此均能相互连接（公网和内网都可以）。
- 节点之中不可以有重复的主机名、MAC 地址或 product_uuid。请参见这里了解更多详细信息。
- 开启机器上的某些端口。请参见这里了解更多详细信息。
- 禁用交换分区。为了保证 kubelet 正常工作，你必须禁用交换分区。

官方提供的常见故障排查：

[官方-故障排查文档](https://kubernetes.io/zh-cn/docs/setup/production-environment/tools/kubeadm/troubleshooting-kubeadm/)

本次换警在虚拟机进行的基本配置：

内存：2GB

CPU : 2核CPU

硬盘 : 40GB

本次环境说明：

操作系统： CentOS 7.9

内核版本： 3.10.0-1160.el7.x86_64

[Kubernetes](https://kubernetes.io/zh-cn/releases/download/) : 1.25.3 （2022年10月 最新版）

containerd 1.6.9

runc 1.1.4


# 二、基础环境准备


三台机器，最好固定IP，相互可以ping。

```
192.168.147.130  k8s-master
192.168.147.131  k8s-node01
192.168.147.132  k8s-node02
```

## 1、分别修改hosts文件

为的是统一服务器的hostname，也可以不设置，但是主机名不能一样，我是虚拟机克隆的，必须要改。

修改 k8s-master 主机名

```bash
hostnamectl set-hostname k8s-master
```

修改 k8s-node01 主机名

```bash
hostnamectl set-hostname k8s-node01
```

修改 k8s-node02 主机名

```bash
hostnamectl set-hostname k8s-node02
```

在**所有节点** `/etc/hosts` 添加主机名映射

```
192.168.147.130  k8s-master
192.168.147.131  k8s-node01
192.168.147.132  k8s-node02
```

然后分别在其中一台 ping 一下另外两台：

```
ping   k8s-master
ping   k8s-node01
ping   k8s-node02
```

> 如果没有特别说明在哪个节点执行，相关命令需要在所有节点执行。


## 2.关闭防火墙和selinux

关闭防火墙

```bash
systemctl stop firewalld && systemctl disable firewalld && iptables -F
```

{: .tips }
> 注意，生产环境请依次开放所需默认端口，如（master）6443、2379-2380、2379-2380、10250、10257、10259,(worker)10250、30000-32767，也可以全部都自定义。

关闭selinux

```bash
sed -i 's/enforcing/disabled/' /etc/selinux/config && setenforce 0
```

## 3. 关闭swap分区

临时关闭

```bash
swapoff -a
```

永久关闭swap

```bash
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
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
```

```bash
sysctl --system
```


系统优化内核参数，选其一即可。

```bash
cat > /etc/sysctl.d/k8s_better.conf << EOF
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables=1
net.ipv4.ip_forward=1
vm.swappiness=0
vm.overcommit_memory=1
vm.panic_on_oom=0
fs.inotify.max_user_instances=8192
fs.inotify.max_user_watches=1048576
fs.file-max=52706963
fs.nr_open=52706963
net.ipv6.conf.all.disable_ipv6=1
net.netfilter.nf_conntrack_max=2310720
EOF

# 开启 ipvs 转发
modprobe br_netfilter
lsmod |grep conntrack
modprobe ip_conntrack
sysctl -p /etc/sysctl.d/k8s_better.conf
```

> 启用 bridge-nf-call-iptables 这个内核参数 (置为 1)，表示 bridge 设备在二层转发时也去调用 iptables 配置的三层规则 (包含 conntrack)

## 5.加载ip_vs内核模块

> kubernetes里kube-proxy支持三种模式，在v1.8之前我们使用的是iptables 以及 userspace两种模式，在kubernetes 1.8之后引入了ipvs模式，并且在v1.11中正式使用，其中iptables和ipvs都是内核态也就是基于netfilter，只有userspace模式是用户态。
>
> - **userspace模式**
>
>   在k8s v1.2后就已经被淘汰了，userspace的作用就是在proxy的用户空间监听一个端口，所有的svc都转到这个端口，然后proxy的内部应用层对其进行转发。proxy会为每个svc随机监听一个端口，并增加一个iptables规则，从客户端到 ClusterIP:Port 的报文都会被重定向到 Proxy Port，Kube-Proxy 收到报文后，通过 Round Robin (轮询) 或者 Session Affinity（会话亲和力，即同一 Client IP 都走同一链路给同一 Pod 服务）分发给对应的 Pod。所有流量都是在用户空间进行转发的，虽然比较稳定，但是效率不高。
>
> - **Iptables 模式** 
>
>   iptables这种模式是从kubernetes1.2开始并在v1.12之前的默认模式。在这种模式下proxy监控kubernetes对svc和ep对象进行增删改查。并且这种模式使用iptables来做用户态的入口，而真正提供服务的是内核的Netilter，Netfilter采用模块化设计，具有良好的可扩充性。其重要工具模块IPTables从用户态的iptables连接到内核态的Netfilter的架构中，Netfilter与IP协议栈是无缝契合的，并允许使用者对数据报进行过滤、地址转换、处理等操作。这种情况下proxy只作为Controller。Kube-Proxy 监听 Kubernetes Master 增加和删除 Service 以及 Endpoint 的消息。对于每一个 Service，Kube Proxy 创建相应的 IPtables 规则，并将发送到 Service Cluster IP 的流量转发到 Service 后端提供服务的 Pod 的相应端口上。并且流量的转发都是在内核态进行的，所以性能更高更加可靠。
>
>   在这种模式下缺点就是在大规模的集群中，iptables添加规则会有很大的延迟。因为使用iptables，每增加一个svc都会增加一条iptables的chain。并且iptables修改了规则后必须得全部刷新才可以生效。
>
> - **IPVS 模式**
>   kubernetes从1.8开始增加了IPVS支持，IPVS相对于iptables来说效率会更加高，使用ipvs模式需要在允许proxy的节点上安装ipvsadm，ipset工具包加载ipvs的内核模块。并且ipvs可以轻松处理每秒 10 万次以上的转发请求。
>
>   当proxy启动的时候，proxy将验证节点上是否安装了ipvs模块。如果未安装的话将回退到iptables模式。
>
>   并在Kubernetes 1.12成为kube-proxy的默认代理模式。ipvs模式也是基于netfilter，对比iptables模式在大规模Kubernetes集群有更好的扩展性和性能，支持更加复杂的负载均衡算法(如：最小负载、最少连接、加权等)，支持Server的健康检查和连接重试等功能。ipvs依赖于iptables，使用iptables进行包过滤、SNAT、masquared。ipvs将使用ipset需要被DROP或MASQUARED的源地址或目标地址，这样就能保证iptables规则数量的固定，我们不需要关心集群中有多少个Service了。
>
>   ————————————————
>   版权声明：本文为CSDN博主「噜噜噜的博客」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。原文链接：https://blog.csdn.net/qq_22543991/article/details/99563898
>
>   ipvs 为负载均衡算法提供了更多选项，例如：
>
>   - rr ：轮询调度
>   - lc ：最小连接数
>   - dh ：目标哈希
>   - sh ：源哈希
>   - sed ：最短期望延迟
>   - nq ： 不排队调度

如果kube-proxy 模式为ip_vs则必须加载，如果没有加载，就会退回到 iptables 模式.

```bash
modprobe ip_vs
modprobe ip_vs_rr
modprobe ip_vs_wrr
modprobe ip_vs_sh
modprobe nf_conntrack_ipv4
```

如果没有安装就安装一下

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

{: .important }
> 😅 特别注意，linux kernel 4.19版本已经将 nf_conntrack_ipv4 更新为 nf_conntrack， 而 kube-proxy 1.13 以下版本，强依赖 nf_conntrack_ipv4。 我的环境内核版本3.10 所以写是 nf_conntrack_ipv4，如果机器内核版本 >= 4.19 换成 nf_conntrack 即可。另外也选择可以升级 kube-proxy 1.13+，或者降级内核版本。😌



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

# 三、运行环境准备

为了在 Pod 中运行容器，Kubernetes 使用 容器运行时（Container Runtime）。默认情况下，Kubernetes 使用 容器运行时接口（Container Runtime Interface，CRI） 来与你所选择的容器运行时交互。如果你不指定运行时，kubeadm 会自动尝试通过扫描已知的端点列表来检测已安装的容器运行时。如果检测到有多个或者没有容器运行时，kubeadm 将抛出一个错误并要求你指定一个想要使用的运行时。


> k84 在 1.24+ 版本之后默认移除 dockershim，如果非要使用docker，需要安装一个支持CRI标准的shim（垫片）中间层组件—— `cri-dockerd` 一头通过CRI跟kubelet交互，另一头跟docker api交互，从而间接的实现了kubernetes以docker作为容器运行时。但是这种架构缺点也很明显，调用链更长，效率更低。
> - docker 作为 k8s 容器运行时，调用关系为：   kubelet --> dockershim （在 kubelet 进程中）  --> dockerd --> containerd
> - containerd 作为 k8s 容器运行时，调用关系为： kubelet --> cri plugin（在 containerd 进程中） --> containerd

[k8s + docker 运行时](https://kubernetes.io/docs/tasks/administer-cluster/migrating-from-dockershim/migrate-dockershim-dockerd/)

就是需要额外安装一下第三方适配插件 cri-docker .

## 1、安装containerd [所有节点安装]

创建 /etc/modules-load.d/containerd.conf 配置文件:

```bash
cat << EOF > /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF


#加载文件系统模块  # 加载 转发模块
modprobe overlay && modprobe br_netfilter
```

> br_netfiler作用：br_netfilter模块可以使 iptables 规则可以在 Linux Bridges 上面工作，用于将桥接的流量转发至iptables链。在基本使用过程中，如果没有加载br_netfilter模块，那么并不会影响不同node上的pod之间的通信，但是会影响同node内的pod之间通过service来通信。


获取阿里云YUM源

```bash
wget -O /etc/yum.repos.d/docker-ce.repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

在线安装：

```bash
yum install -y containerd.io
```
离线安装参考 [ containerd ]() 篇

## 2、配置containerd [所有节点安装]

生成containerd的配置文件

```bash
mkdir /etc/containerd -p && containerd config default > /etc/containerd/config.toml
```

编辑配置文件

```bash
vim /etc/containerd/config.toml
```

修改内容：

(1) `SystemdCgroup = false` 改为 `SystemdCgroup = true`

(2) `sandbox_image` 的镜像仓库地址。
```
# sandbox_image = "k8s.gcr.io/pause:3.6"
```
改为：
```
sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.6"
```
{: .warning }
>旧的版本是 `k8s.gcr.io/pause:3.6` 新的版本是：`registry.k8s.io/pause:3.7` 特别说明一下，依赖里显示的是3.8，但是安装的时候阿里云还没有3.8，只有3.7

替换命令：

```bash
sed -i 's#k8s.gcr.io#registry.aliyuncs.com/google_containers#g' /etc/containerd/config.toml

sed -i 's#registry.k8s.io#registry.aliyuncs.com/google_containers#g' /etc/containerd/config.toml
sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml 
```

{: .warning }
>不知道是什么缘故，我执行sed命令无法成功，已检查过语法没有问题。

重启运行时容器：

```bash 
systemctl enable --now containerd && systemctl restart containerd
```

安装验证

```bash
ctr images ls  && ps -ef | grep containerd
ctr version && runc --version
```

{: .note }
> <1>`ctr`是`containerd`自带的CLI命令行工具。`crictl`是k8s中CRI（容器运行时接口）的客户端，k8s使用该客户端和containerd进行交互。所以就也安装一下吧。
>
> <2> containerd 相比于docker , 多了namespace概念, 每个image和container 都会在各自的namespace下可见，crictl 使用命名空间 k8s.io 。

## 3、安装 crictl [所有节点安装]

暂时没有发现在线安装。

离线安装 crictl

```bash
VERSION="v1.25.0"
wget https://github.com/kubernetes-sigs/cri-tools/releases/download/$VERSION/crictl-$VERSION-linux-amd64.tar.gz
tar zxvf crictl-$VERSION-linux-amd64.tar.gz -C /usr/local/bin
rm -f crictl-$VERSION-linux-amd64.tar.gz
```
或者下载好tar包，上传后解压一下。

```bash
crictl -v
crictl --help
```
如果执行crictl命令是报错，提示如下信息

```
WARN[0000] image connect using default endpoints: [unix:///var/run/dockershim.sock unix:///run/containerd/containerd.sock unix:///run/crio/crio.sock unix:///var/run/cri-dockerd.sock]. As the default settings are now deprecated, you should set the endpoint instead. 
ERRO[0000] unable to determine image API version: rpc error: code = Unavailable desc = connection error: desc = "transport: Error while dialing dial unix /var/run/dockershim.sock: connect: no such file or directory"  
```

就创建配置文件，指定一下调用端点`endpoint`:
- 使用 `Docker` 运行时Unix 域套接字是: `unix:///var/run/dockershim.sock`
- 使用 `cri-dockerd` 运行时Unix 域套接字是: `unix:///var/run/cri-dockerd.sock`
- 使用 `containerd` 运行时Unix 域套接字是:	`unix:///var/run/containerd/containerd.sock`
- 使用 `CRI-O` 运行时Unix 域套接字是:	`unix:///var/run/crio/crio.sock`

```bash
cat <<EOF > /etc/crictl.yaml 
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 5
debug: false
EOF
```

重启一下 `containerd`

```bash
systemctl restart containerd
```

# 四、安装k8s 1.25.3

## 1、添加安装的yum源

```bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

更新下缓存

```bash
yum clean all && yum makecache
# do this
yum makecache fast
```

## 2、安装kubeadm，kubelet和kubectl

生产环境记得校验文件。

在每台机器上安装以下的软件包：

  - kubeadm：用来初始化集群的指令。
  - kubelet：在集群中的每个节点上用来启动 Pod 和容器等。
  - kubectl：用来与集群通信的命令行工具。

```bash
yum install -y kubectl kubelet kubeadm
yum install -y kubectl kubelet kubeadm --nogpgcheck -y

systemctl enable --now kubelet
```

准备k8s1.25.3 所需要的镜像

```bash
kubeadm config images list --kubernetes-version=v1.25.3
```

## 3、集群初始化

使用kubeadm init命令初始化，在 **k8s-master** 上执行

```bash
kubeadm init --kubernetes-version=v1.25.3 --pod-network-cidr=10.224.0.0/16 --apiserver-advertise-address=192.168.147.130 --image-repository registry.aliyuncs.com/google_containers
```

参数说明：

- --apiserver-advertise-address 集群通告地址
- --image-repository 由于默认拉取镜像地址k8s.gcr.io国内无法访问，这里指定阿里云镜像仓库地址
- --kubernetes-version K8s版本，与上面安装的一致
- --service-cidr 集群内部虚拟网络，Pod统一访问入口
- --pod-network-cidr Pod网络，，与下面部署的CNI网络组件yaml中保持一致

出现以下内容说明初始化成功：

```
To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.147.130:6443 --token albnsj.7axslw1i5bv1byno \
        --discovery-token-ca-cert-hash sha256:647a98e0c91fdb307bb15638b4590344b3fff446f4e49318d437dbf1368f3e06
 
```

查看节点验证：

```bash
kubectl get node 
```

现在只有1个主节点。

{: .warning }
>有可能会初始化失败。如果失败参考最后一个故障排查章节找一下解决办法.

## 4、其他节点加入集群

在 **k8s-node01** 和 **k8s-node02** 节点执行： 

```
kubeadm join 192.168.147.130:6443 --token 7rehau.x9g0q32zfqh0fhb7 \
        --discovery-token-ca-cert-hash sha256:3e43951c1145d283951beac3d576c681f22de3fe674c568b0db5b1ed100c2d77
```

> 注意命令是在一行。

再次查询节点验证：

```bash
kubectl get node 
```

## 5、集群部署网络插件

Kubernetes跨主机容器之间的通信组件，目前主流的是flannel和calico，二选一即可，不必都安装。


### 5.1 安装 calico 。

> Calico是一个纯三层的数据中心网络方案，Calico支持广泛的平台，包括Kubernetes、OpenStack等。
>
> Calico 在每一个计算节点利用 Linux Kernel 实现了一个高效的虚拟路由器（ vRouter） 来负责数据转发，而每个 vRouter 通过 BGP 协议负责把自己上运行的 workload 的路由信息向整个 Calico 网络内传播。
>
> 此外，Calico 项目还实现了 Kubernetes 网络策略，提供ACL功能。
>

下载Calico

```bash
wget https://docs.projectcalico.org/manifests/calico.yaml --no-check-certificate

vim calico.yaml
...
- name: CALICO_IPV4POOL_CIDR
  value: "10.244.0.0/16"
...
```

通常默认情况下，PodIP地址范围 `--pod-network-cid=10.244.0.0/16`，它拥有`16^2=256*256=65536`个地址（包括网络地址+广播地址）可拆分成独立子网。

Calico为每个节点都会创建一个独立子网，即从CIDR大的地址池中划分较小范围的地址池给到每个节点。

Calico可以通过修改配置blockSize块大小来设置每个节点分配的独立子网的范围池大小。这边默认值IPV4=26，IPV6=122。二进制掩码26（11111111 11111111 11111111 11000000）转换成十进制掩码即=255.255.255.192，即每个节点的子网可以有64个IP地址，减去广播地址和网络地址，可为Pod分配的有效IP地址有62个。

> 用户在创建私有网络时，需要用 [CIDR（无类别域间路由）](https://cloud.tencent.com/document/product/215/18509#2) 作为私有网络指定 IP 地址组。

> ### 无类别域间路由
>
> CIDR（Classless Inter-Domain Routing）即无类别域间路由，由您指定的独立网络空间地址块，通过 IP 和掩码结合，实现对网络的整体划分。以`10.1.0.0/16`为例，其中`10.1.0.0`为网络块的 IP，`16`为网络块的掩码。通过设定掩码的大小，可以调整网络块的大小设定。网络块包括的 IP 数 = 2 ^( 32 - 掩码)，因此`10.1.0.0/16`网络块最多包含65536个 IP 地址。

- **10.0**.0.0 - **10.255**.255.255（**掩码范围需在12 - 28**之间）
- **172.16**.0.0 - **172.31**.255.255（**掩码范围需在12 - 28**之间）
- **192.168**.0.0 - **192.168**.255.255 （**掩码范围需在16 - 28**之间)

参考： https://cloud.tencent.com/document/product/215/20046

设置好CIDR之后，启动网络组件

```bash
kubectl apply -f calico.yaml
```

查看节点：

```bash
  kubectl get pods -n kube-system
  kubectl get node 
```

### 5.2 安装 flannel 

```bash
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

```
"Network": "10.224.0.0/16"
```

修改完网络配置后启动

```bash
kubectl apply -f kube-flannel.yaml
```

查看部署的结果

```bash
kubectl -n kube-system get pods -o wide
```



## 6、部署dashborad


>注意，版本要和k8s版本匹配，具体参考：https://github.com/kubernetes/dashboard/releases。

部署操作：

```bash
wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml

# 目前最新版本 
vim recommended.yaml

#在kind 为 service 部分，增加了 type: NodePort 

----
spec:
  ports:
    - port: 443
      targetPort: 8443
      nodePort: 30001
  type: NodePort
  selector:
    k8s-app: kubernetes-dashboard
----

# 启动pod
kubectl apply -f recommended.yaml

#验证 pod 运行
kubectl get pod -n kubernetes-dashboard -o wide

# 验证 service 运行
kubectl get svc -n kubernetes-dashboard
```

https://192.168.147.130:30001/#/login

然后就是创建访问账户生成token了。

创建访问账户

```bash
# 创建访问账号，准备一个yaml文件
[root@k8s-master01 ~]# vi dashboard-admin.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: dashboard-admin
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: dashboard-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: dashboard-admin
  namespace: kubernetes-dashboard
 
[root@k8s-master01 ~]# kubectl apply -f dashboard-admin.yaml
```

获取访问token

```bash
[root@k8s-master k8s-images]# kubectl -n kubernetes-dashboard create token dashboard-admin
eyJhbGciOiJSUzI1NiIsImtpZCI6InVLWGVwWTU0akNZdk9WbHRNZkhfRDZ4WEUtTWJRclhCVUw1Qk5KYV9JbE0ifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNjY4MjM5NTUxLCJpYXQiOjE2NjgyMzU5NTEsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsInNlcnZpY2VhY2NvdW50Ijp7Im5hbWUiOiJkYXNoYm9hcmQtYWRtaW4iLCJ1aWQiOiI5ZWE3YTQxMi02NDViLTQwYTktOWEwMC1mYWFhZjBjZDlmNWUifX0sIm5iZiI6MTY2ODIzNTk1MSwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmVybmV0ZXMtZGFzaGJvYXJkOmRhc2hib2FyZC1hZG1pbiJ9.pRqwTb9pNcX4YxLIbxUZ2wzflY1EmwKb9PQngvad1HvDfZUnIWlSQhte5KnJpLdJ5WaGLj0TqEA-fknI7_Pj_ROCgIdTLn6tglQXNoD7nRbOpe_dfeO26sa92N0GK6P07RRXXgW5i0_GeWbHsjmhZKDXB8uW-Ru0j2dW_--lwx3ivEQKVG3MubpkW_HpGN-NQgt9aO4xdr3o2-G7HMKXYVwjDSf-G9GUPhoP6RcmI-3TH81g8NhfnSRNhO-s4mTmKxky3CcYs9byuXPVYKrHJqKt8A7W3mGnmXrvI-E4aW5ygfhYu6hb_rOShyaaur9tJtUMW-rTXCgHXFcdaI-lkg
[root@k8s-master k8s-images]# 
```

帖入token登录即可。

~~token 默认是 15 分钟过期，可以修改 `--token-ttl`的值进行修改。~~


## 7、检查状态

查看所有命名空间的pod 或者 节点状态。

```bash
#查节点
kubectl get nodes -o wide
[root@k8s-master ~]# kubectl get nodes -o wide
NAME         STATUS     ROLES           AGE   VERSION   INTERNAL-IP       EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION           CONTAINER-RUNTIME
k8s-master   Ready      control-plane   37h   v1.25.3   192.168.147.130   <none>        CentOS Linux 7 (Core)   3.10.0-1160.el7.x86_64   containerd://1.6.9
k8s-node01   Ready      <none>          35h   v1.25.3   192.168.147.131   <none>        CentOS Linux 7 (Core)   3.10.0-1160.el7.x86_64   containerd://1.6.9
k8s-node02   NotReady   <none>          13h   v1.25.4   192.168.147.132   <none>        CentOS Linux 7 (Core)   3.10.0-1160.el7.x86_64   containerd://1.6.9

# 查所有的pod
kubectl get pod -A
kubectl get pod --all-namespaces

# 看k8s相关的服务pod
kubectl get pods -n kube-system
```

等待一段时间全部节点就会处于Ready状态即可。

也可以通过获取集群状态的方法，检查是否已恰当的配置了 kubectl：

```bash
kubectl cluster-info
```

如果返回一个 URL，则意味着 kubectl 成功的访问到了你的集群。

如果命令 kubectl cluster-info 返回了 url，但还不能访问集群，那可以用以下命令来检查配置是否妥当：

```bash
kubectl cluster-info dump
```

## 8、kubectl 的可选配置和插件

### 8.1 启用 shell 自动补全功能c

安装 bash-completion

```bash
yum install bash-completion

#ubantu
apt-get install bash-completion  
```

可以用命令 `type _init_completion` 检查 `bash-completion` 是否已安装。

启动补全功能。

针对系统全局启动

```bash
kubectl completion bash | sudo tee /etc/bash_completion.d/kubectl > /dev/null
```

如果 kubectl 有关联的别名，你可以扩展 Shell 补全来适配此别名：

```bash
echo 'alias k=kubectl' >>~/.bashrc
echo 'complete -o default -F __start_kubectl k' >>~/.bashrc
```

重新加载shell

```bash
exec bash
```

# 五、kubectl 集群故障排查

## 1、官方文档

- [集群故障排查](https://kubernetes.io/zh-cn/docs/tasks/debug/debug-cluster/)

- [官方-故障排查文档](https://kubernetes.io/zh-cn/docs/setup/production-environment/tools/kubeadm/troubleshooting-kubeadm/)

## 2、其他

- **kubelet 参数查看**

```bash
# kubelet配置文件
cat /etc/sysconfig/kubelet
```
> 这个配置文件是kubelet可以向对外提供参数的地方，我们想加参数的时候可以在这里加；

```bash
vi /etc/sysconfig/kubelet
KUBELET_EXTRA_ARGS=--container-runtime=remote --container-runtime-endpoint=/run/containerd/containerd.sock --cgroup-driver=systemd
```

```
#查看运行时参数
kubelet --help | grep runtime
```
- **kubeadm 集群初始化失败 [master]**

:collision: 常见错误1:

```
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory “/etc/kubernetes/manifests”. This can take up to 4m0s
[kubelet-check] Initial timeout of 40s passed.
```
❤️‍🩹 通常是镜像下载问题，可以先用国内镜像源拉下来，然后再改tag 
```
k8s.gcr.io/XXXX
registry.k8s.io/xxx
```
然后可以删除旧镜像，该过程中最好使用 crictl 命令，会默认tag 到 k8s.io 命名空间下。
也可能国内就是没有同步到最新的镜像，就降低一下镜像的小版本，可以到docker仓库找一下，也可以降低一下k8s版本。
我本次安装就是最新的 v1.25.3,依赖pause:3.8,但是docker仓库只有3.7，没有3.8版本，所以调整一下版本后也可以启动。

```
ctr  image pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.25.3
ctr  image pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager:v1.25.3
ctr  image pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.25.3
ctr  image pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.25.3
ctr  image pull registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.7
ctr  image pull registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.5.4-0
ctr  image pull registry.cn-hangzhou.aliyuncs.com/google_containers/coredns/coredns:v1.9.3
crictl  image tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.25.3 
crictl  image tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager:v1.25.3 registry.k8s.io/kube-controller-manager:v1.25.3
crictl  image tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.25.3 registry.k8s.io/kube-scheduler:v1.25.3
crictl  image tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.25.3 registry.k8s.io/kube-proxy:v1.25.3
crictl  image tag registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.8 registry.k8s.io/pause:3.7
crictl  image tag registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.5.4-0 registry.k8s.io/etcd:3.5.4-0
crictl  image tag registry.cn-hangzhou.aliyuncs.com/google_containers/coredns/coredns:v1.9.3 registry.k8s.io/coredns/coredns:v1.9.3
```

另一种办法就是下载镜像包，上传之后手工导入。[k8s 组件镜像包下载](https://www.downloadkubernetes.com/)

```
crictl  image import kube-apiserver 
crictl  image import kube-controller-manager 
crictl  image import kube-scheduler
...
```

:collision: 常见错误2

主节点获取节点信息报错如下:
```
[root@k8s-master ~]# kubectl get node 
The connection to the server localhost:8080 was refused - did you specify the right host or port?
```

解决参考 https://blog.csdn.net/CEVERY/article/details/108753379

> 环境变量原因：kubernetes master没有与本机绑定，集群初始化的时候没有绑定，此时设置在本机的环境变量即可解决问题。

❤️‍🩹 可以通过设置环境变量解决

```bash
#具体根据情况，此处记录linux设置该环境变量
#方式一：编辑文件设置
vim /etc/profile
#在底部增加新的环境变量
export KUBECONFIG=/etc/kubernetes/admin.conf

#方式二:直接追加文件内容
echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> /etc/profile

# 使之生效
source /etc/profile	
```

执行结果：

```bash
[root@k8s-master ~]# echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> /etc/profile
[root@k8s-master ~]# source /etc/profile
[root@k8s-master ~]# kubectl get nodes
NAME         STATUS     ROLES           AGE    VERSION
k8s-master   NotReady   control-plane   141m   v1.25.3
k8s-node01   NotReady   <none>          11m    v1.25.3
[root@k8s-master ~]# 
```


:collision: 常见错误3:

❤️‍🩹 主要就是 端口被占用  和 一些k8s组件的yaml文件已存在的错误。`kubeadm init` 失败 需要`kubeadm reset` 

```bash
kubeadm reset && rm -fr  $HOME/.kube/config
# 重新初始化集群
kubeadm init  --apiserver-advertise-address=192.168.147.130 --image-repository registry.aliyuncs.com/google_containers
```


:collision: 常见错误4(所有节点)

```
WARN[0000] image connect using default endpoints: [unix:///var/run/dockershim.sock unix:///run/containerd/containerd.sock unix:///run/crio/crio.sock unix:///var/run/cri-dockerd.sock]. As the default settings are now deprecated, you should set the endpoint instead. 
ERRO[0000] unable to determine image API version: rpc error: code = Unavailable desc = connection error: desc = "transport: Error while dialing dial unix /var/run/dockershim.sock: connect: no such file or directory"  
```

❤ ‍🩹 安装一下 crictl

```bash
cat <<EOF > /etc/crictl.yaml 
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 5
debug: false
EOF
```
然后重启一下 `systemctl restart containerd` 。


- **kubectl join 报错**

:collision: 其他node加入集群时报错：

```bash
[root@k8s-node02 k8s-images]# kubeadm join 192.168.147.130:6443 --token e7g3g9.e1y6bdqd53takj8r --discovery-token-ca-cert-hash sha256:628d4e2345640d2b98c6b3ef7d7c71fa5125e489fcb9037e0605993bba403e6d
[preflight] Running pre-flight checks
error execution phase preflight: couldn't validate the identity of the API Server: could not find a JWS signature in the cluster-info ConfigMap for token ID "e7g3g9"
To see the stack trace of this error execute with --v=5 or higher
```

❤️‍🩹 解决参考： https://blog.csdn.net/qq_49530779/article/details/122037992

(1)k8s api server不可达

此时需要检查和关闭**所有节点**的 `firewalld`和`selinux`。

```bash
[root@k8s-master ~]#setenforce 0
[root@k8s-master ~]#sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
[root@k8s-master ~]#systemctl disable firewalld --now
```

(2)token 过期

此时需要通过kubedam在master主机上重新生成token, 然后根据token输出添加命令

```bash
[root@k8s-master ~]# kubeadm token generate
kv55to.xjn7lugwe25h21a0
[root@k8s-master ~]# kubeadm token create kv55to.xjn7lugwe25h21a0 --print-join-command --ttl=0
kubeadm join 192.168.147.130:6443 --token kv55to.xjn7lugwe25h21a0 --discovery-token-ca-cert-hash sha256:628d4e2345640d2b98c6b3ef7d7c71fa5125e489fcb9037e0605993bba403e6d 
[root@k8s-master ~]# 
```

Node 节点加入之后再次查询就有了（嗯，隔了几天不小心安装了一个1.25.4）

```bash
[root@k8s-master k8s-images]# kubectl  get node
NAME         STATUS     ROLES           AGE   VERSION
k8s-master   NotReady   control-plane   24h   v1.25.3
k8s-node01   NotReady   <none>          22h   v1.25.3
k8s-node02   NotReady   <none>          3s    v1.25.4
[root@k8s-master k8s-images]# 
```

- **非主节点使用kubectl命令**

:collision:  非主节点无法执行 kubectl 命令

❤️‍🩹在node节点把主节点的`admin.conf`拷贝过来，添加配置即可。

```bash
cd /etc/kubernetes && scp root@192.168.147.130:/etc/kubernetes/admin.conf .
echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> ~/.bash_profile
source ~/.bash_profile
```

- **pod 出现 CrashLoopBackOff 状态**

💥 pod 状态出现CrashLoopBackOff，无法访问了 

```bash
[root@k8s-master k8s-images]# kubectl get pod -n kubernetes-dashboard
NAME                                         READY   STATUS             RESTARTS      AGE
dashboard-metrics-scraper-598b75c6fd-kkfzh   1/1     Running            0             2m42s
kubernetes-dashboard-784f89dbd-l58qr         0/1     CrashLoopBackOff   4 (52s ago)   2m43s
[root@k8s-master k8s-images]# 
``` 

❤️‍🩹  咋处理 ?
