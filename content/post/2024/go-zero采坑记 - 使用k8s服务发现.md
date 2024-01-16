---
title: "go-zero采坑记 - 使用k8s服务发现"
date: "2024-01-16"
lastMod: "2024-01-16"
categories: ["it"]
tags: ["golang", "go-zero", "k8s", "服务发现", "微服务"]
---

# 前言
研发框架搭建得差不多了，最近在本地和生产测试环境一致性。go-zero官方以介绍使用etcd做为服务发现首选，可能更多人并没有选择部署在k8s上吧。为了快速进入开发，我也选择了etcd，过程还算顺利。

这几天在找资料解决如何相同配置文件适应不同环境的时候，又遇到了选择服务发现的问题，似乎k8s是一个更好的选择。服务本来就运行在k8s上，不必再单独部署etcd服务，减少外部依赖，增强健壮性。

# 过程：
1. 删除服务配置文件中关于etcd的配置，添加以下配置：
	```yaml
	IdRpc:
	Target: <Cluster IP>:<Service Port> # 开发环境使用，端口为service映射的端口，在集群外部通过集群地址访问
	#   Target: k8s://<namespace>/<service>:<port> # 生产环境使用，在集群内部通过命名空间+服务名访问
	```

2. 创建集群访问权限
  因为官方提供的链接已经是404状态了，通过找github历史版本记录能找回，现记录如下：
  - ClusterRole，定义集群范围的权限角色，不受 namespace 控制
	```yaml
	apiVersion: rbac.authorization.k8s.io/v1
	kind: ClusterRole
	metadata:
		name: endpoints-reader
	rules:
		- apiGroups: [""]
		resources: ["endpoints"]
		verbs: ["get", "watch", "list"]
	```
  - ServiceAccount，定义 namespace 范围内的 service account
	```yaml
	apiVersion: v1
	kind: ServiceAccount
	metadata:
		name: endpoints-reader
		namespace: <namespace> # the namespace to create the ServiceAccount
	```
  - ClusterRoleBinding，将定义好的 ClusterRole 和不同 namespace 的 ServiceAccount 进行绑定
	```yaml
	apiVersion: rbac.authorization.k8s.io/v1
	kind: ClusterRoleBinding
	metadata:
		name: endpoints-reader
	subjects:
		- kind: ServiceAccount
		name: endpoints-reader
		namespace: <namespace> # the namespace that the ServiceAccount resides in
	roleRef:
		kind: ClusterRole
		name: endpoints-reader
		apiGroup: rbac.authorization.k8s.io
	```

3. 部署deployment的时候加上serviceAccountName 来指定使用哪个 ServiceAccount
    ```yaml
	apiVersion: apps/v1
	kind: Deployment
	metadata:
	name: alpine-deployment
	labels:
		app: alpine
	spec:
	replicas: 1
	selector:
		matchLabels:
		app: alpine
	template:
		metadata:
		labels:
			app: alpine
		spec:
		serviceAccountName: endpoints-reader #添加这一行
		containers:
		- name: alpine
			image: alpine
			command:
			- sleep
			- infinity
	```

# 参考
- <https://learnku.com/articles/60748>
- <https://github.com/zeromicro/go-zero/blob/v1.2.3/zrpc/internal/resolver/kube/deploy/serviceaccount.yaml>
- <https://github.com/zeromicro/go-zero/blob/v1.2.3/zrpc/internal/resolver/kube/deploy/clusterrolebinding.yaml>
- <https://github.com/zeromicro/go-zero/blob/v1.2.3/zrpc/internal/resolver/kube/deploy/clusterrole.yaml>
- <https://github.com/zeromicro/zero-examples/tree/main/discovery/k8s>