---
layout: default
title: Kubernetes Use
parent: Container
nav_order: 15
---

# Kubernetes  Cluster
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}


k8s 使用
--------------------------------------------


# 一、基本环境
 
 
# 删除 POD 

```bash
# 找到所有的pod，部署的pod比较多就不推荐这种方法
kubectl get pod -A

# 找到所有的命名空间
kubectl get namespaces

#找到要删除的pod
kubectl get pod -n your_namespace

#删除 pod，
kubectl delete pod your_pod_name -n your_namespace --force --grace-period=0

# 删除 deployment
kubectl get deployment -n  your_namespace
kubectl delete deployment your_deployment_name -n your_namespace


# 再次查看验证
kubectl get deployment -n your_namespace
kubectl get pod -n your_namespace
```

## 1、k8s 安装 dashboard

>注意，版本要和k8s版本匹配，具体参考：https://github.com/kubernetes/dashboard/releases。

安装部署：

```bash
wget -O https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml

# 目前最新版本 

vim recommended.yaml

#在kind 为 service 部分，增加了 type: NodePort 。`nodePort: 30001` 是在指定pod容器端口，不指定就是随机端口,由于 NodePort 限制，默认限制端口范围只能为 30000-32767。

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

创建访问账户区分旧版和新版。

- 旧版创建

创建账户`dashboard-admin`，获取token

```bash
kubectl create serviceaccount dashboard-admin -n kubernetes-dashboard
```

授权,这里是创建集群管理员角色

```bash
kubectl create clusterrolebinding dashboard-admin-rb --clusterrole=cluster-admin --serviceaccount=kubernetes-dashboard:dashboard-admin
```

获取token

```bash
kubectl get secrets -n kubernetes-dashboard | grep dashboard-admin
kubectl describe secrets dashboard-admin-token-xxxxx -n kubernetes-dashboard
```

不知道端口号就查一下服务，或者直接在`recommended.yaml`文件中自定义端口：
```
kubectl  get svc -n kubernetes-dashboard
```

https://192.168.147.130:3xxxx/#/login

输入token登录、

- k8s 1.25.x 新版创建账户

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

查一下 pod 端口
```bash
[root@k8s-node01 ~]# kubectl  get svc -n kubernetes-dashboard
NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGE
dashboard-metrics-scraper   ClusterIP   10.104.67.199    <none>        8000/TCP        16h
kubernetes-dashboard        NodePort    10.109.157.223   <none>        443:31038/TCP   16h
[root@k8s-node01 ~]# 
```

https://192.168.147.130:31038/#/login

输入token登录、

token 是会过期的。


删除服务账户和集群角色绑定：

```bash
kubectl -n kubernetes-dashboard delete serviceaccount dashboard-admin
kubectl -n kubernetes-dashboard delete clusterrolebinding dashboard-admin
```  


[k8s-rabc权限控制](https://blog.csdn.net/BigData_Mining/article/details/88849696)


