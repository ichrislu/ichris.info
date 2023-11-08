---
title: "使用Nexus3搭建docker registry"
date: "2018-10-11"
lastMod: "2019-07-19"
categories: ["it"]
tags: ["nexus", "docker registry"]
---

### Nexus 3.0 的三种docker仓库：

1. docker (proxy)      代理和缓存远程仓库 ，只能pull
2. docker (hosted)    托管仓库 ，私有仓库，可以push和pull
3. docker (group)      将多个proxy和hosted仓库添加到一个组，只访问一个组地址即可，只能pull



**前提：安装配置好了Java运行环境**

安装：下载Nexus 3，解压，./bin/nexus run 或 ./bin/nexus start就能运行

### 配置：

#### 一、CentOS 7开机启动服务，编辑/etc/systemd/system/nexus.service

```
[Unit]
Description=nexus service
After=network.target

[Service]
Type=forking
LimitNOFILE=65536
ExecStart=/data/nexus-3.9.0-01/bin/nexus start
ExecStop=/data/nexus-3.9.0-01/bin/nexus stop
User=devops
Restart=on-abort

[Install]
WantedBy=multi-user.target
```

注：

1、LimitNOFILE=65536，为Nexus建议配置，不然界面会有警告提示，

2、User=devops，运行用户

#### 二、使用keytool为Nexus生成自签名的证书

进入<Nexus 3 Dir>/etc/ssl，执行

keytool -genkeypair -keystore keystore.jks -storepass password -keypass password -alias jetty -keyalg RSA -keysize 2048 -validity 5000 -dname "CN=*.${NEXUS_DOMAIN}, OU=Example, O=Sonatype, L=Unspecified, ST=Unspecified, C=US" -ext "SAN=DNS:${NEXUS_DOMAIN},IP:${NEXUS_IP_ADDRESS}" -ext "BC=ca:true"



说明 ：

${NEXUS_DOMAIN}：Nexus所在主机域，我填的主机名可以

${NEXUS_IP_ADDRESS}：Nexus对外IP



编辑nexus-default.properties

添加application-port-ssl=8443

添加,${jetty.etc}/jetty-https.xml到nexus-args后面



编辑<Nexus 3 Dir>/etc/jetty-https.xml

​    <Set name="KeyStorePath"><Property name="ssl.etc"/>/keystore.jks</Set>

​    <Set name="KeyStorePassword">password</Set>

​    <Set name="KeyManagerPassword">password</Set>

​    <Set name="TrustStorePath"><Property name="ssl.etc"/>/keystore.jks</Set>

​    <Set name="TrustStorePassword">password</Set>



重启Nexus生效



输出证书：

keytool -printcert -sslserver ${NEXUS_DOMAIN}:${SSL_PORT} -rfc > ${FILE_NAME}.cer



说明 ：

1、${NEXUS_DOMAIN}填IP也可以

2、服务器将打印证书信息



复制证书信息，保存为证书文件*.cer



CentOS 7安装证书：

sudo cp *.crt /etc/pki/ca-trust/source/anchors/myregistrydomain.com.crt sudo update-ca-trust

重启Docker生效

Windows安装证书：

右键：选择安装证书

存储位置：本地计算机

浏览，选择：受信任的根证书颁发机构



#### 三、docker登录

大坑：一般来说，走完1-2步，就可以了，网上都是这么说的，但docker一直登录不上

报错：Error response from daemon: login attempt to https://ip:5443/v2/ failed with status: 401 Unauthorized



查了2天，最终发现问题在于没有配置Nexus的Realms，默认只有Local Authenticating Realm、Local Authorlzing Realm，将Docker Bearer Token Realm添加上就好了



#### 四、docker proxy仓库

非公网环境，建议取消“Disable to allow anonymous pull (Note: also requires Docker Bearer Token Realm to be activated)”，允许匿名pull



#### 五、关于权限

普通开发人员使用Nexus，不必要使用admin账号，建议创建新账号

普通开发人员日常工作只涉及到浏览仓库，提交程序包版本到库，因此只需要添加以下权限即可：

nx-repository-view-*-*-add

nx-repository-view-*-*-browse

nx-repository-view-*-*-edit

nx-repository-view-*-*-read



或者，其实nx-repository-view-*-*-* + nx-anonymous可能更有道理！



参考文档：



中文安装，基本是以下官方参考文档的翻译，还有其他3项仓库的示例

<http://www.cnblogs.com/wzy5223/p/5410990.html>

<https://www.cnblogs.com/wzy5223/p/5414965.html>



官方参考文档：Using Self-Signed Certificates with Nexus Repository Manager and Docker Daemon

<https://support.sonatype.com/hc/en-us/articles/217542177>



导入证书

<https://docs.docker.com/registry/insecure/#docker-still-complains-about-the-certificate-when-using-authentication>



Nexus启动SSL

<https://help.sonatype.com/display/NXRM3/Configuring+SSL#ConfiguringSSL-InboundSSL-ConfiguringtoServeContentviaHTTPS>