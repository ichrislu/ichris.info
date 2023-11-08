---
title: "k8s-cronjob"
date: "2021-04-06"
lastMod: "2021-04-06"
categories: ["it"]
tags: ["k8s", "cronjob"]
---

当前k8s版本：1.17，在使用cronjob的时候发现调度的时间不对，配置的是0 0 * * *，但实际运行时间为早上8点，推测是时区问题没跑了～
但是运行程序的容器已经处理过时区问题，而且调度是k8s发起了，问题在k8s那边。

经查，果然！找到k8s master机器，路径：/etc/kubernetes/manifests，可以看到4个文件：

1. etcd.yaml
2. kube-apiserver.yaml
3. kube-controller-manager.yaml
4. kube-scheduler.yaml

网上找到的方案是添加/etc/localtime卷映射，经测试正常

添加localtime的volumes

```yaml
containers:
  volumeMounts:
  - mountPath: /etc/localtime
    name: localtime
    readOnly: true
volumes:
- hostPath:
    path: /etc/localtime
  name: localtime
```

上述操作不需要重启，因为是静态pod

有个小插曲，很奇怪。前几天按上面的操作，除kube-controller-manager，其他都正常。
看到网上说可以添加一个TZ的环境变量，发现好像可以。
但过后再看又不行了，再重新进入，做成跟其他3个文件一样，莫名其妙又好了。现在只能怀疑因为是静态pod，可能k8s应用需要时间吧。。。

添加TZ环境变量（仅添加这个应该是不行的，未测试！）

```yaml
env:
- name: TZ
  value: "Asia/Shanghai"
```
