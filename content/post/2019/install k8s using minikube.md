---
title: "使用minikube安装kubernetes单结点集群"
date: "2019-07-20"
lastMod: "2019-08-04"
categories: ["it"]
tags: ["minikube", "k8s"]
---

从网上查到的资料来看，使用minikube安装kubernetes单结点集群相对kubeadm安装kubernetes集群简单多了，但是实践过程还是遇到一些问题，记录下来，用作交流：

### 说明

本次安装环境：

操作系统：CentOS Linux release 7.6.1810 (Core)
kubectl：v1.15.0
minikube：v1.2.0

因为Linux的先天优势，偷懒便没有安装VM插件。目的是搭建小团队的集成开发环境，所以安装问题可以忽略，使用—vm-driver=none参数

过程和说明会比较简单，建议有一定基础再阅读

下载镜像以及相关软件需要翻墙，以及配置代理，不在本篇文章作说明，可以参见之前的文章相关部分：[kubeadm安装k8s集群](/2019/install-k8s-using-kubeadm/)

### 下载

因为是计划后期做成与kubeadm一样的ansible playbook，所以事先将安装程序下载准备好

- kubectl，官方参考：https://k8smeetup.github.io/docs/tasks/tools/install-kubectl/
- minikube，官方：https://github.com/kubernetes/minikube

### 准备工作

- ~~配置docker cgroup：同样参见：[kubeadm安装k8s集群](/2019/install-k8s-using-kubeadm/)~~
- 禁用交换分区：同样参见：[kubeadm安装k8s集群](/2019/install-k8s-using-kubeadm/)
- 开8443 10250端口

### 步骤

- 启动集群，自动下载镜像

```bash
sudo /usr/local/bin/minikube start --vm-driver=none
```

- 按提示迁移配置，生效登录ssh生效！

```bash
sudo mv /root/.kube /root/.minikube $HOME
sudo chown -R $USER $HOME/.kube $HOME/.minikube
```

- 修改配置文件：

```bash
vim ~/.kube/config

# 改以下3项路径为当前用户路径
# certificate-authority
# client-certificate
# client-key
```



#### 优化方案

可以不用迁移配置，以及修改配置文件

```bash
export MINIKUBE_WANTUPDATENOTIFICATION=false && \
	export MINIKUBE_WANTREPORTERRORPROMPT=false && \
	export MINIKUBE_HOME=$HOME && \
	export CHANGE_MINIKUBE_NONE_USER=true && \
	export KUBECONFIG=$HOME/.kube/config && \
	http_proxy=<proxy-host>:<proxy-port> sudo -E /usr/local/bin/minikube start --vm-driver=none
```



### 遗留问题

- 按官方以及其他文章建议，运行sudo minikube start --vm-driver=none，一直报错：没权限？不能确定了。改为sudo /usr/local/bin/minikube start --vm-driver=none，可以运行，很奇怪！

### TODO

- 定制版本
  
  ```bash
  # 查阅版本
  minikube get-k8s-versions
  # 选择版本启动
  minikube start --kubernetes-version v1.7.3 --vm-driver=none
```
  
- 镜像加速：

  ```base
  minikube start --registry-mirror=https://registry.docker-cn.com
  ```

- kubectl自动补全
  ```
  echo "source <(kubectl completion bash)" >> ~/.bashrc
  ```