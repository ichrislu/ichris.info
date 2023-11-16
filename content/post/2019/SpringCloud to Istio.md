---
title: "从Spring Cloud到Istio，架构升级（更新中……）"
date: "2019-07-15"
lastMod: "2019-07-25"
categories: ["it"]
tags: ["Spring Cloud", "k8s"]
---

## 架构从SpringCloud升级到Istio

### 优势

1. 自动伸缩
2. 健康检查和自愈能力
3. kubernetes生态好，解决方案多；监控告警、网络等都有成熟方案和插件，有问题也很容易找到解决方案
4. 不局限于docker，同样也支持RKT（当前docker为ce版本）
5. 官方自带dashboard，可进行简单操作，大部分操作可不必使用命令行
6. Spring Cloud限定了开发语言为Java，Kubernetes天然支持异构
7. 单独管理服务的CPU、内存、磁盘和网络

### 前置条件

#### 境外代理

与kubernetes相关所有软件安装包、容器镜像都在google的服务器，因为GFW的存在，需要翻墙访问。

为安装部署各个环境，所以需要购买一台境外服务器，安装翻墙代理软件。

以AWS为例

- 低配置的预留实例每年费用不到1000人民币（不含流量和磁盘费用），而且新用户第1年免费
- 流量大概0.12美元/G，首G免费
- 磁盘0.12美元/G-月，仅装翻墙代理，只需要系统盘6-8G足够

注：流量和磁盘费用不同区域价格可能会有少量差异

一般来说，折合每月费用不会超过3美元

### 实践过程

#### 组件删减替代方案

- eureka -> etcd
- config -> map secret

#### 开发调试

对于云原生十二要素之9：Disposability，可快速启动和优雅关闭，可能未完全实现

扩展要素之1：优先考虑API设计，可能未完全满足（要求增量添加接口，高度向前兼容）

其他条件都已具备

#### 部署方法
通过自动化运维工具Ansible，初始化操作系统、安装和配置防火墙、安装Docker、配置Dockers、安装kubernetes集群、初始化群集等操作

注：有一个弊端，目前Ansible主控端并不支持Windows，仅支持macOS和Linux

##### 开发环境

minikube

##### 测试环境

minikube+kubeadm

##### 生产环境

kubeadm+原生部署

### 对人员要求

#### 开发人员

#### 部署人员

## 降低内存占用

- 拆解公共包为：功能、配置、常量、公共接口调用、实体类
- 所有跨模块接口调用在公共包中实现，由各模块调用；由统一环境变量决定模块间接口调用是走Feign，还是本地调用（待考虑实现细节）



---

> > [服务迁移之路 | Spring Cloud向Service Mesh转变](https://juejin.im/post/5ce26e266fb9a07eb67d619f)
> >
> > [Kubernetes和Spring Cloud哪个部署微服务更好？](https://www.kubernetes.org.cn/1057.html)