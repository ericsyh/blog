---
title: kubeadm 初探
date: 2019-11-25 18:56:27
tags: 
    - k8s
    - kubeadm
---

[kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/) 是目前 k8s 用于部署单主集群的主要工具，仅需要 `init` 和 `join` 两个命令就能完成 k8s 集群的部署。下面在 Azure 上的三台 VM 为实验环境，进行 kubeadm 的相关部署操作：

* 所有节点上部署 Docker 和 kubeadm 
* 通过 kubeadm 部署 k8s master 节点
* 通过 kubeadm 部署 k8s worker 节点
* 部署 k8s 网络插件 calico
* 部署 k8s Dashboard
* 部署 k8s 存储插件 rook

### 安装 Docker

按照 [Docker 部署文档](https://docs.docker.com/install/linux/docker-ce/centos/) 进行 Docker 的安装。

1. 依赖包安装

```bash
sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```

2. 添加 yum 源

```bash
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

3. 安装 Docker

```bash
sudo yum install docker-ce docker-ce-cli containerd.io
```

4. 启动 Docker 进程

```bash
sudo systemctl start docker
```

### 安装 kubeadm

```bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

# Set SELinux in permissive mode (effectively disabling it)
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

systemctl enable --now kubelet
```

### 部署 k8s master 节点

在 master 节点上执行如下的命令

```bash
kubeadm init
```

在 master 节点初始化完成后，会提示执行如下的配置命令

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

以及 worker 节点加入集群时候锁需要的 Join 命令

### 部署 k8s worker 节点

在 worker 节点上执行如下的命令

```bash
kubeadm join 10.0.0.4:6443 --token 6f46sk.du12egqecd9ix11z \
    --discovery-token-ca-cert-hash sha256:6366e915e608763e31cb9007c258ed3c366ea4ef3f6560b5cf88a730dbec1067
```

### 部署网络插件 Calico

```bash
kubectl apply -f https://docs.projectcalico.org/v3.8/manifests/calico.yaml
```

查看各个系统 Pod 的状态，是否都已经启动

```bash
kubectl get pods --all-namespaces

NAMESPACE     NAME                                      READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-55754f75c-2c7lp   1/1     Running   0          79s
kube-system   calico-node-mxhv8                         1/1     Running   0          79s
kube-system   calico-node-nm2jr                         1/1     Running   0          79s
kube-system   calico-node-vc7c6                         1/1     Running   0          79s
kube-system   coredns-5644d7b6d9-hc98f                  1/1     Running   0          15m
kube-system   coredns-5644d7b6d9-k4vn5                  0/1     Running   0          15m
kube-system   etcd-k8s-master                           1/1     Running   0          14m
kube-system   kube-apiserver-k8s-master                 1/1     Running   0          14m
kube-system   kube-controller-manager-k8s-master        1/1     Running   1          14m
kube-system   kube-proxy-f2cs8                          1/1     Running   0          15m
kube-system   kube-proxy-q5pcp                          1/1     Running   0          11m
kube-system   kube-proxy-qmjw6                          1/1     Running   0          11m
kube-system   kube-scheduler-k8s-master                 1/1     Running   1          14m
```

### 部署 Dashboard

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta6/aio/deploy/recommended.yaml
```
