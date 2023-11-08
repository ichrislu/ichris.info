---
title: "sonarqube+gitlab"
date: "2021-03-29"
lastMod: "2021-03-29"
categories: ["it"]
tags: ["sonarqube", "gitlab"]
---

### 目标

安装SonarQube，在gitlab-runner上运行SonarScanner，SonarQube与Gitlab的账户打通

### 安装SonarQube

很简单，参考官方文档即可

```yaml
  sonarqube:
    image: sonarqube:8.9.0-community
    ports:
    - '9000:9000'
    volumes:
    - /cem/sonarqube/conf:/opt/sonarqube/conf
    - /cem/sonarqube/data:/opt/sonarqube/data
    - /cem/sonarqube/extensions:/opt/sonarqube/extensions
    - /cem/sonarqube/cacerts:/opt/java/openjdk/lib/security/cacerts:ro
    environment:
      SONAR_JDBC_URL: jdbc:postgresql://postgresql:5432/sonarqube
      SONAR_JDBC_USERNAME: sonarqube
      SONAR_JDBC_PASSWORD: *********
    container_name: sonarqube
    restart: unless-stopped
```

### SonarScanner

则是通过gitlab-ci.yml实现，官方文档并不详细，遇到一个小问题
因为我没有在项目中写sonar-project.properties文件，通过gitlab-ci.yml中variables和添加script的-D参数实现的，官方并没有说哪些需要用variables中映射环境变量实现，哪些需要通过-D参数传过去。慢慢试，得出以下：

```yaml
  variables:
    SONAR_HOST_URL: "http://192.168.1.10:9000"
    SONAR_TOKEN: "606a5553c22e3a960ce35463989e8ce58077abd6"
    SONAR_USER_HOME: "${CI_PROJECT_DIR}/.sonar"
    GIT_DEPTH: "0"
  cache:
    key: "${CI_JOB_NAME}"
    paths:
      - .sonar/cache
  script:
    - sonar-scanner -Dsonar.qualitygate.wait=true -Dsonar.projectKey=${CI_PROJECT_NAMESPACE}:${CI_PROJECT_NAME} -Dsonar.exclusions=pb/*
```

### SonarQube+Gitlab

虽然SonarQube可以匿名上传分析报告，但是允许匿名登录则游客可以浏览项目代码，不安全！

看到官方文档有写可以通过Gitlab登录，不需要分配账号，节省了很多管理麻烦，使用也更方便。但实现过程却遇到了问题，记录如下

1. 第1个问题则是TLS证书问题：安全起见，SonarQube回调地址必须是https，但SonarQube部署是在本地。于是想了一个主意：在公网公司官方上加了一个子链接，如www.xxx.com/sonar，NG上配置该地址跳转到本地，这个骚操作不错哈～～～
   
   ```nginx
                   location ~ /sonar {
                           rewrite ^/sonar/(.*) http://192.168.1.10:9000/$1 permanent;
                   }
   ```

2. 本来以为可以了，结果测试发现跳不过去，一直报错，查Sonar日志：unable to find valid certification path to requested target。后面想明白了，公网跳本地是正常的。但这个错误本地调用Gitlab验证的时候报错了，因为本地Gitlab是https的，但证书却是自签名的，Java不认！
    网上找资料，进容器中测试，发现把自己的ca导入java默认的ca文件中，wget测试通过！于是
   
   1. 先复制自签名的ca.crt到容器，再在容器中把自己的ca文件导入java默认的ca中：
      
      ```shell
      # 进入容器运行
      cd /opt/java/openjdk/lib/security/
      keytool -import -alias <自定义别名> -keystore cacerts -file ca.crt -trustcacerts
      ```
   
   2. 将容器中/opt/java/openjdk/lib/security/cacerts文件复制到宿主机中
      
      ```shell
      docker cp sonarqube:/opt/java/openjdk/lib/security/cacerts .
      ```
   
   3. 修改docker-compose.yml文件，将上述文件映射替换容器内相应的文件
      
      ```yaml
      volumes:
      - /cem/sonarqube/cacerts:/opt/java/openjdk/lib/security/cacerts:ro
      ```

### 配置Gitlab

管理员账号进入Applications，添加一个Application

Name：SonarQube，只是标识作用
URI：https://domain.com/sonar/oauth2/callback/gitlab
Scopes：api/read_user

### 配置SonarQube

1. ALM集成
   1. GitLab：启用GitLab认证，并配置GitLab URL，Application ID，Secret，勾选：Allow users to sign-up/Synchronize user groups
2. 配置 -> 通用：Server Base URL：https://domain.com/sonar，以及可选的邮件配置
3. 配置 -> 权限：Force user authentication：勾选

### 参考

1. https://stackoverflow.com/questions/55617053/sonarqube-remove-code-view-for-anyone-group