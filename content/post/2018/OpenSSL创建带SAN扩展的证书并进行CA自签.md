---
title: "OpenSSL创建带SAN扩展的证书并进行CA自签"
date: "2018-01-11"
lastMod: "2020-03-18"
categories: ["it"]
tags: ["linux", "ssl", "https", "证书"]
---

## 2020-03-18
苹果2019年调整新策略，按之前的方式生成证书会有问题（目前发现Chrome 80会报证书不可信，Safari可用），最长证书不超过825天

**以下操作，最好用root用户，不建议sudo**

### 创建CA
openssl genrsa -des3 -out ca.key 4096
openssl req -sha256 -new -x509 -days 825 -key ca.key -out ca.crt \
    -subj "/C=CN/ST=GuangDong/L=ShenZhen/O=CEM/OU=www.cem-tech.cn/CN=CEM Tech"

### 创建证书私钥
私钥密码是必须的
`openssl genrsa -des3 -out server.key 4096`

去掉私钥密码
`openssl rsa -in server.key -out server.key`

### 创建证书请求文件
```bash
openssl req -new \
    -sha256 \
    -key server.key \
    -subj "/C=CN/ST=GuangDong/L=ShenZhen/O=CEM/OU=www.cem-tech.cn/CN=CEM Tech" \
    -reqexts SAN \
    -config <(cat /etc/pki/tls/openssl.cnf \
        <(printf "extendedKeyUsage=id-kp-serverAuth\n[SAN]\nsubjectAltName=DNS:192.168.1.10,IP:192.168.1.10")) \
    -out server.csr
```

### 初始化文件
当前操作系统如果是第1次执行：为了能使证书可以工作还需要创建两个文件。  
建议用root！！第1行可以sudo，第2行必须root

`touch /etc/pki/CA/index.txt`

`echo 01 > /etc/pki/CA/serial`

### 签名
```bash
openssl ca -in server.csr \
    -days 825 \
    -md sha256 \
    -keyfile ca.key \
    -cert ca.crt \
    -extensions SAN \
    -config <(cat /etc/pki/tls/openssl.cnf \
        <(printf "extendedKeyUsage=id-kp-serverAuth\n[SAN]\nsubjectAltName=DNS:192.168.1.10,IP:192.168.1.10")) \
    -out server.crt
```

## 基本概念

OpenSSL相当于SSL的一个实现，如果把SSL规范看成OO中的接口，那么OpenSSL则认为是接口的实现。接口规范本身是安全没问题的，但是具体实现可能会有不完善的地方，比如之前的"心脏出血"漏洞，就是OpenSSL中的一个bug。

### 扩展名
- .crt certificate的缩写，证书文件,可以是DER(二进制)编码的，也可以是PEM(ASCII(Base64))编码的，在类unix系统中比较常见；
- .cer 也是证书，常见于Windows系统。编码类型同样可以是DER或者PEM的，windows下有工具可以转换crt到cer；
- .csr 是Certificate Signing Request的缩写，这不是证书。证书签名请求文件，一般是生成请求以后发送给CA（权威的证书颁发机构），然后CA会给你签名并发回证书；
- .key 通常指私钥，一般公钥或者密钥都会用这种扩展名，可以是DER编码的或者是PEM编码的。查看DER编码的(公钥或者密钥)的文件的命令为：openssl rsa -inform DER -noout -text -in xxx.key。查看PEM编码的(公钥或者密钥)的文件的命令为：openssl rsa -inform PEM -noout -text -in xxx.key；
- .p12证书文件,包含一个X509证书和一个被密码保护的私钥。

### 证书类型
x509的证书编码格式有两种:
- PEM(Privacy-enhanced Electronic Mail)是明文格式的，以 --BEGIN CERTIFICATE--开头，已--END CERTIFICATE--结尾。中间是经过base64编码的内容，apache需要的证书就是这类编码的证书。查看这类证书的信息的命令为：openssl x509 -noout -text -in server.pem。其实PEM就是把DER的内容进行了一次base64编码
- DER是二进制格式的证书，查看这类证书的信息的命令为：openssl x509 -noout -text -inform der -in server.der

### 自签名类型
- 自签名证书
- 私有CA签名证书
	自签名证书的Issuer和Subject是相同的。

区别:
自签名的证书无法被吊销，私有CA签名的证书可以被吊销。
如果你的规划需要创建多个证书，那么使用私有CA签名的方法比较合适，因为只要给所有的客户端都安装相同的CA证书，那么以该CA证书签名过的证书，客户端都是信任的，也就只需要安装一次就够了。
如果你使用用自签名证书，你需要给所有的客户端安装该证书才会被信任。如果你需要第二个证书，则需要给所有客户端安装第二个CA证书才会被信任。

参考：
- <http://blog.csdn.net/babydavic/article/details/8442817>
- <http://blog.csdn.net/liuchunming033/article/details/48470575>
- <http://blog.csdn.net/fyang2007/article/details/6180361>
- <https://support.apple.com/en-us/HT210176>
- <https://blog.csdn.net/qq_41453285/article/details/99558821>
- <https://blog.gslin.org/archives/tag/oid/>
- <https://blog.csdn.net/hatmen2/article/details/81265593>