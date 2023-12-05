---
title: "GitLab CICD自动更新k8s中容器版本"
date: "2023-12-05"
lastMod: "2023-12-05"
categories: ["it"]
tags: ["gitlab", "gitlab runner", "k8s", "CICD"]
---

## 前提
之前使用Gitlab和Runner的版本是12.\*，现在已经到了16.\*，安装和配置跟之前相比几乎没有变化，所以按之前的文档操作下来，一路比较顺利。在更新k8s集群容器版本的时候出现问题了，原来Auto Deploy方式在14.\*版本被弃用，较新的方案是安装k8s代理，方案有2种：
1. Helm：挺麻烦的，开发环境不必这么麻烦和复杂
2. KPT：更麻烦，使问题更复杂化
确实不想再去搞Helm或KPT了，不知道是因为年龄大了，没有精力了；还是潜意识里业务进度大于技术研究已经慢慢渗入思想了。

已经几乎要放弃了，在解决构建容器时间不正确的时候，却发现了新大陆。
方案是：将安装有kubectl机器的`~/.kube/config`文件映射进Gitlab-Runner，给Runner Helper，然后在`.gitlab-ci.yml`文件中添加before_script，在kubectl容器执行更新Pod版本之前加载`.kube/config`文件，获得k8s权限。

**操作配置**

gitlab-runner compose文件重点部分
```bash
volumes
  - /home/lssy/.kube:/root/.kube:ro
```

gitlab-runner/config/config.toml 配置文件重点部分
```bash
[[runners]]
  [runners.docker]
    volumes = ["/cache", "/home/lssy/.kube/config:/build/kubeconfig/config", "/etc/localtime:/etc/localtime:ro", "/var/run/docker.sock:/var/run/docker.sock", "/home/lssy/.docker/config.json:/root/.docker/config.json"]
```

.gitlab-ci.yml 重点部分
```bash
k8s-deploy-dev:
  stage: deploy
  image:
    name: bitnami/kubectl:1.28.2
    entrypoint: [ "" ]
  before_script:
    - cp -fr /build/kubeconfig/config /.kube/config
  script:
    -  kubectl rollout restart -n www deployment/www
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
```

参考：
- <https://zhuanlan.zhihu.com/p/654029372>