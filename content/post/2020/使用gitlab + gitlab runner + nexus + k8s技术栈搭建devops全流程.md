---
title: "使用gitlab + gitlab runner + nexus + k8s技术栈搭建devops全流程"
date: "2020-04-09 18:45:03"
lastMod: "2020-04-14 18:50:03"
categories: ["it"]
tags: ["gitlab", "gitlab runner", "nexus", "k8s", "CICD"]
---

### 版本清单

- gitlab：12.8.1-ce.0
- gitlab runner：v12.9.0
- nexus：3.21.1
- k8s：1.17.4
- istio：1.5.1

### 前提条件

- maven镜像未使用官方的，而是基于JDK镜像自己创建，原因是官方的镜像对于配置全局mirror没有找到简单方便的办法
- kubectl官方没有镜像，当前使用的是：bitnami/kubectl

### 安装过程

#### gitlab

- 使用compose安装：

```yml
  gitlab:
    image: gitlab/gitlab-ce:12.8.1-ce.0
    ports:
    - '22:22'
    - '80:80'
    - '443:443'
    volumes:
    - /data/gitlab/config:/etc/gitlab
    - /data/gitlab/logs:/var/log/gitlab
    - /data/gitlab/data:/var/opt/gitlab
    container_name: gitlab
    restart: unless-stopped
```

- gitlab.rb配置文件修改：

```ruby
external_url 'https://ip'

gitlab_rails['time_zone'] = 'Asia/Shanghai'

gitlab_rails['gitlab_email_enabled'] = true
gitlab_rails['gitlab_email_from'] = 'xxx@xxx.xxx'
gitlab_rails['gitlab_email_display_name'] = 'xxx'

gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.xxx.xxx.xxx"
gitlab_rails['smtp_port'] = 465
gitlab_rails['smtp_user_name'] = "xxx@xxx.xxx"
gitlab_rails['smtp_password'] = "********"
gitlab_rails['smtp_domain'] = "xxx.xxx"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = true

user['git_user_name'] = "GitLab"
user['git_user_email'] = "xxx@xxx.xxx"

nginx['enable'] = true
nginx['redirect_http_to_https'] = true
nginx['redirect_http_to_https_port'] = 80

nginx['ssl_certificate'] = "/etc/gitlab/trusted-certs/xxx.crt"
nginx['ssl_certificate_key'] = "/etc/gitlab/trusted-certs/xxx.key"
```

- 常规配置，管理员登录

1. 管理中心，外观，自定义Logo/登录注册页面等
2. 管理中心，设置，网络，外发请求，勾选：<u>允许Webhook和服务对本地网络的请求</u> 和 <u>允许系统钩子向本地网络发送的请求</u>

- 添加kubernetes集群，可选global，group，project范围，以group为例：找到group配置页面，选择Kubernetes，添加，填入：集群名、API URL，CA证书，Token等，保存即可

```bash
# 获取API URL
kubectl cluster-info | grep 'Kubernetes master' | awk '/http/ {print $NF}'

# 获取ca证书
kubectl get secret <secret name> -n <namespace> -o jsonpath="{['data']['ca\.crt']}" | base64 --decode > ca.pem

# 获取token，名称空间：istio-test
kubectl -n <namespace> describe secret $(kubectl -n <namespace> get secret | grep gitlab-service-account | awk '{print $1}')
```

- **重要事项**：添加或配置k8s集群时，切记不要勾选<u>GitLab-managed cluster</u>，https://docs.gitlab.com/ee/user/project/clusters/#custom-namespace

#### gitlab-runner

- 使用compose安装：

```yaml
  gitlab-runner:
    image: gitlab/gitlab-runner:v12.9.0
    volumes:
    - ~/gitlab-runner/config:/etc/gitlab-runner
    - /var/run/docker.sock:/var/run/docker.sock
    - ~/.docker/config.json:/root/.docker/config.json:ro
    - ~/certs/ca.crt:/etc/gitlab-runner/certs/192.168.1.10.crt:ro
    container_name: gitlab-runner
    restart: unless-stopped
```

- 配置文件修改：

```toml
# 设置并发数量
concurrent = 8

# 映射宿主机docker.sock、config.json
# 持久化数据，如maven本地仓库，当容器重新创建时也能重复使用，加快构建速度
volumes = ["/cache", "/var/run/docker.sock:/var/run/docker.sock", "/home/cem/.docker/config.json:/root/.docker/config.json", "/home/cem/.m2:/root/.m2"]
```

- 注册到gitlab略！网上一堆文档，命令：

```bash
docker exec -it gitlab-runner gitlab-runner register
```

- 重要说明

1. 使用docker方式，映宿主机docker.sock文件，这样在容器内执行的docker操作与在宿主机一致，比如在容器内创建的镜像，等同于在宿主机创建的镜像（因此有一定风险性）
2. 映射宿主机config.json，这样在宿主机执行登录过docker registry的情况下，容器内可以不用做登录操作，就可以直接pull/push，这种方式个人感觉比在容器中每次在pull/push前执行login命令更简单，更安全（只需要在宿主机登录，密码并未泄露；反之，还需要在gitlab中配置docker registry地址，访问的用户名和密码）
3. 对于自签名证书的问题，参考：
   https://docs.gitlab.com/runner/configuration/tls-self-signed.html
   https://docs.gitlab.com/runner/install/docker.html#installing-trusted-ssl-server-certificates

- 遗留问题

1. gitlab-runner执行产生的容器，在某些情况下好像并不会自动删除（有可能是构建失败的情况下）
2. gitlab-runner运行过程中可能会执行docker build，导致宿主机会产生大量未使用的image，后期考虑定期执行docker image prune -y，加入cron
3. 网上说不建议gitlab与gitlab-runner安装在同一台服务器，个人猜测原因可能是ip相同；公司服务器是多网卡的，后期考虑激活第2个网卡，gitlab-runner绑定到第2个网卡的IP上（需要把gitlab绑定到第1个网卡，因为有评书限制），参考：https://docs.gitlab.com/runner/configuration/advanced-configuration.html#the-session_server-section

### nexus

- 使用compose安装

```yaml
  nexus:
    image: sonatype/nexus3:3.21.1
    ports:
    - '8081:8081'
    - '5000:5000'
    - '5001:5001'
    environment:
    - INSTALL4J_ADD_VM_PARAMS=-Xms4g -Xmx8g -XX:MaxDirectMemorySize=4g
    volumes:
    - /data/nexus:/nexus-data
    container_name: nexus
    restart: unless-stopped
```

- 常规配置

1. Docker仓库需要：Security -> Realms：添加Docker Bearer Token Realms
2. 匿名访问：Security -> Anonymous Access
3. 添加开发都角色（能上传）：Security -> Roles -> Privileges，添加权限nx-repository-view-*-*-*以及nx-anonymous角色；再添加一个用户到设置为这个角色

- 端口说明

8081：nexus默认web访问端口
5000：docker registry端口
5001：docker proxy端口

### 参考

- [GitLab CI/CD Pipeline Configuration Reference](https://docs.gitlab.com/ee/ci/yaml/README.html)
- [Building Docker images with GitLab CI/CD](https://docs.gitlab.com/ee/ci/docker/using_docker_build.html)
- [Predefined environment variables reference](https://docs.gitlab.com/ce/ci/variables/predefined_variables.html)
- [Environments and deployments](https://docs.gitlab.com/ee/ci/environments.html)
- [Advanced configuration](https://docs.gitlab.com/runner/configuration/advanced-configuration.html)

