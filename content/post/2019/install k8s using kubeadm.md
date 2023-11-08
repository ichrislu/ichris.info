---
title: "kubeadm安装k8s集群"
date: "2019-06-10"
lastMod: "2019-06-26"
categories: ["it"]
tags: ["kubeadm", "k8s"]
---

coredns是k8s-dns的替代者，Kubernetes 1.11的默认选项

## 关于CentOS科学上网
## 安装shadowsocks
```shell
# 先安装扩展包，方便安装后面的pip等：
sudo yum -y install epel-release

# 安装pip
sudo yum -y install python-pip

# 通过pip安装shadowsocks
sudo pip install shadowsocks

# 启动测试
sslocal -c shadowsocks.json

# 开机服务
```
### 配置文件：shadowsocks.json
```json
{
    "server":"代理地址",
    "server_port":代理端口,
    "local_port":1080,
    "password":"密码",
    "timeout":600,
    "method":"aes-256-cfb",
    "fast_open": false,
    "workers": 4
}
```

## 安装Docker
```
略
```

## 转socks为http代理
```shell
# 安装
sudo yum install -y privoxy

# 编辑配置文件
sudo vim /etc/privoxy/config
# 添加配置socks代理信息
forward-socks5  /       127.0.0.1:1080  .

# 启用服务
sudo systemctl enable privoxy
# 启动程序
sudo systemctl start privoxy
```

## 为docker配置代理
```shell
# 编辑配置文件
sudo vim /etc/systemd/system/docker.service.d/http-proxy.conf

# 添加如下内容
[Service]
Environment="HTTP_PROXY=http://192.168.1.201:8118" "NO_PROXY=localhost,127.0.0.1,docker.io,docker.com"

# 应用生效
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## 添加k8s仓库
```shell
# 添加仓库
sudo vim /etc/yum.repos.d/kubernetes.repo

[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg

# 查看源提供的版本
yum list kubeadm
```

## 为k8s仓库配置代理
```shell
# 为k8s的仓库配置代理
sudo vim /etc/yum.repos.d/kubernetes.repo

# 添加如下内容
proxy=http://127.0.0.1:8118

```

## 离线安装kubeadm/kubectl/kubelet
```shell
# 下载安装包和其依赖包
sudo yum install --downloadonly --downloaddir=. kubelet-1.14.3-0 kubeadm-1.14.3-0 kubectl-1.14.3-0

# 强制忽略依赖安装
rpm -ivh --force --nodeps *.rpm

# 查看需要使用哪些镜像
kubeadm config images list
参考：
https://www.jianshu.com/p/f9a54e553ce4
https://www.cnblogs.com/xieyifeng/p/9383236.html
```

## 启用systemd
```shell
# 修改docker配置文件：
sudo vim /etc/docker/daemon.json

# 添加内容
"exec-opts" : ["native.cgroupdriver=systemd"]

# 重启docker生效
```

## 群集初始化
```shell
# 这里貌似必须要root
$ sudo kubeadm init \
   --kubernetes-version=v1.14.0 \
   --pod-network-cidr=10.244.0.0/16 \
   --apiserver-advertise-address=10.151.30.57 \

# 后续工作
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## 加入集群
```shell
kubeadm join 192.168.1.201:6443 --token <token> \
    --discovery-token-ca-cert-hash sha256:<ca-cert-hash>
```

## 安装pod网络（on master）
```
$ wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
$ kubectl apply -f  kube-flannel.yml
```

## 集群重置
```shell
kubeadm reset
```

## TODO 遗留问题：安装网络时，无法使用走代理，暂时通过docker pull先下载镜像解决
```shell
# 以下不能正常走代理，不知道什么原因
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
kubectl apply -f kube-flannel.yml

# 临时解决办法：
docker pull quay.io/coreos/flannel:v0.11.0-amd64
再执行就好了
```

---

### ~~内网其他机器使用代理~~
应当是不需要了，因为rpm和docker镜像已经下载好了
```shell
# Docker
# 编辑配置文件
sudo vim /etc/systemd/system/docker.service.d/http-proxy.conf

# 添加如下内容
[Service]
Environment="HTTP_PROXY=http://192.168.1.201:8118" "NO_PROXY=localhost,127.0.0.1,docker.io,docker.com"

# 应用生效
sudo systemctl daemon-reload
sudo systemctl restart docker
---
# yum
# 编辑配置文件
sudo vim /etc/yum.conf

# 添加一行
proxy=http://192.168.200.65:8118
```

---
警告：
```
[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
[WARNING Hostname]: hostname "vm1" could not be reached
[WARNING Hostname]: hostname "vm1": lookup vm1 on 192.168.1.1:53: no such host
```

下面问题解决办法，文档描述不正确，至少1.14.0版该systemd路径是：/usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf

报错：
```
Unfortunately, an error has occurred:
	timed out waiting for the condition

This error is likely caused by:
	- The kubelet is not running
	- The kubelet is unhealthy due to a misconfiguration of the node in some way (required cgroups disabled)

If you are on a systemd-powered system, you can try to troubleshoot the error with the following commands:
	- 'systemctl status kubelet'
	- 'journalctl -xeu kubelet'

Additionally, a control plane component may have crashed or exited when started by the container runtime.
To troubleshoot, list all containers using your preferred container runtimes CLI, e.g. docker.
Here is one example how you may list all Kubernetes containers running in docker:
	- 'docker ps -a | grep kube | grep -v pause'
	Once you have found the failing container, you can inspect its logs with:
	- 'docker logs CONTAINERID'
error execution phase wait-control-plane: couldn't initialize a Kubernetes cluster
```