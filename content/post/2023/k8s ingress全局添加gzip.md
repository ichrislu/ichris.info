---
title: "k8s ingress全局添加gzip"
date: "2023-12-05"
lastMod: "2023-12-05"
categories: ["it"]
tags: ["k8s", "ingress", "nginx", "gzip"]
---

阿里云官方推荐，SLB做4层负载，只做流量转发。所以SSL和gzip都只能放在ingress中配置了。
SSL之前已经配置好了，今天测试一下gzip。

尝试了一下，修改ingress注解，直接报错：`/server-snippet annotation cannot be used. Snippet directives are disabled by the Ingress administrator`
经过一翻搜索，修改命名空间`kube-system`中`nginx-configuration`配置项，添加键值`use-gzip=true`，测试ok！响应报头显示了`Content-Encoding: gzip`

而且全局生效，所有的ingress都支持gzip了。