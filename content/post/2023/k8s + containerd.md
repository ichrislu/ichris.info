---
title: "安装k8s基于containerd"
date: "2023-11-09"
lastMod: "2023-11-09"
categories: ["it"]
tags: ["k8s", "containerd"]
---

上次安装k8s，3年多了；今天尝试再安装一次k8s，不同的是这次不再用docker，改为containerd。  
得益于以前写的ansible脚本，很有参考价值，虽然3年多过去了，很多东西还对得上。

## 一、前期准备
### 1、网络规划
本次安装开发环境，配置1个Master节点，2个Worker节点，在安装的时候即可配置好。

|节点|主机名|IP|
|---|---|---|
|Master|k8s-master|192.168.2.231/22|
|Worker1|k8s-worker1|192.168.2.232/22|
|Worker2|k8s-worker2|192.168.2.233/22|

### 2、关闭系统功能
所有节点都执行

```bash
# 关闭防火墙
sudo systemctl stop firewalld
sudo systemctl disable firewalld

# 关闭SELINUX
sudo setenforce 0

sudo vim /etc/selinux/config
# SELINUX=disabled

# 关闭swap
sudo swapoff -a

# 注释交换分区的挂载
sudo vim /etc/fstab

# 配置网桥
sudo vim /etc/sysctl.conf

net.bridge.bridge-nf-call-ip6tables	= 1
net.bridge.bridge-nf-call-iptables     = 1
net.ipv4.ip_forward                 = 1
vm.swappiness                         = 0

# 加载以下2个模块
sudo modprobe ip_vs_rr
sudo modprobe br_netfilter

# 生效配置
sudo sysctl -p
```

*以下不确定是否需要操作*
```bash
# 安装时间同步服务，确保三台服务器时间准确
sudo yum -y install chrony
sudo systemctl start chronyd
sudo systemctl enable chronyd

# 修改hosts
vi /etc/hosts
192.168.2.231 k8s-master
192.168.2.232 k8s-worker1
192.168.2.233 k8s-worker2

```

### 3、*安装基础工具
可选步骤，个人喜好

```bash
sudo yum install -y vim telnet wget unzip epel-release
```

## 二、安装containerd
所有节点都执行

### 1、下载安装
```bash
wget https://github.com/containerd/containerd/releases/download/v1.7.8-linux-amd64.tar.gz

sudo tar Cxzvf /usr/local containerd-1.7.8-linux-amd64.tar.gz
```

### 2、修改配置
```bash
sudo mkdir /etc/containerd
sudo containerd config default > /etc/containerd/config.toml
sudo vim /etc/containerd/config.toml

#SystemdCgroup的值改为true
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  ...
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true

#由于国内下载不到registry.k8s.io的镜像，修改sandbox_image的值为：
sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.8"
```

### 3、启动服务
```bash
wget https://raw.githubusercontent.com/containerd/containerd/main/containerd.service

sudo mkdir -p /usr/lib/systemd/system
sudo mv containerd.service /usr/lib/systemd/system

systemctl daemon-reload
systemctl enable --now containerd
```

### 4、验证
```bash
ctr version
```

## 三、安装runc
所有节点都执行

下载地址：<https://github.com/opencontainers/runc/releases>

```bash
sudo install -m 755 runc.amd64 /usr/local/sbin/runc
```

## 四、安装CNI插件
所有节点都执行

下载地址：<https://github.com/containernetworking/plugins/releases>

```bash
sudo mkdir -p /opt/cni/bin
sudo tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.3.0.tgz
```

## 五、安装集群
所有节点都执行

```bash
# 添加阿里云的kubernetes源
sudo cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

# 安装最新版
sudo yum install -y kubeadm kubelet kubectl

# 开机自启动
sudo systemctl enable kubelet

# 验证安装
kubeadm version
```

## 六、初始化集群
仅Master节点执行

```bash
sudo kubeadm init --apiserver-advertise-address 192.168.2.231 --image-repository registry.aliyuncs.com/google_containers --kubernetes-version v1.28.3 --pod-network-cidr=10.244.0.0/16

# 参数说明
# API Server广播地址
--apiserver-advertise-address 192.168.2.231
# 指定镜像地址使用阿里云的，默认会使用谷歌镜像
--image-repository registry.aliyuncs.com/google_containers
# 指定当前的kubernetes的版本
--kubernetes-version v1.28.3
# flannel网络的固定地址范围
--pod-network-cidr=10.244.0.0/16
```

执行完成后，还有一系列操作，待做！

## 七、安装网络插件flannel
仅Master节点执行

```bash
# 下载安装文件
wget https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml

# 安装网络插件
sudo kubectl apply -f kube-flannel.yml

# 验证
kubectl get node
kubectl get cs
kubectl get ns
kubectl get pods -n kube-flannel
```

## 八、加入集群
Worker节点执行

```bash
# 加入集群
TODO...

# 验证
TODO...
```

九、安装crictl工具
可选

下载地址：<https://github.com/kubernetes-sigs/cri-tools>
```base
# 解压安装
sudo tar zxvf crictl-v1.28.0-linux-amd64.tar.gz -C /usr/local/bin

# 创建配置文件
sudo cat <<EOF | sudo tee /etc/crictl.yaml
> runtime-endpoint: unix:///run/containerd/containerd.sock
> image-endpoint: unix:///run/containerd/containerd.sock
> timeout: 2
> debug: false
> pull-image-on-create: false
> disable-pull-on-run: false
> EOF

# 验证
crictl pods
```


## 遗留问题
/run/containerd/containerd.sock一直报没有权限，不确定原因

## 参考
- <https://github.com/containerd/containerd/blob/main/docs/getting-started.md>
- <https://developer.aliyun.com/article/1291616>
- <https://kubernetes.io/zh-cn/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/>
- <https://zhuanlan.zhihu.com/p/612051521>