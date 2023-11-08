---
title: "Travis使用AWS短信实现构建部署完成后短信通知"
date: "2019-01-09"
lastMod: "2019-01-09"
categories: ["it"]
tags: ["travis", "AWS", "SNS"]
---

使用scp将构建完成的文件上传至服务器，部署后使用AWS的SNS发短信，考虑过的方案：

- 官方仅提供两种语言的SDK：Java和.Net，而且还要写代码，所以放弃
- Travis的通知并不支持sms和接口调用，只能放弃

后面查AWS的SNS文档，找到有关于AWS CLI的内容，联想到这可能是个更好方案，跟着官方文档安装：

#### 初始系统已经带了Python 2.7，这一步可以略过，直接安装AWS CLI：
```shell
curl https://s3.amazonaws.com/aws-cli/awscli-bundle.zip -o awscli-bundle.zip
unzip awscli-bundle.zip
sudo ./awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws
```
安装完，可以用aws help测试一下

#### IAM中新建子用户、组，添加权限
注意，如果需要发短信，需要添加所有权限（AmazonSNSFullAccess），记下用户的密钥和私钥

#### 继续跟着文档，配置添加配置文件，以Linux为例：
```properties
$ cat ~/.aws/credentials
[default]
aws_access_key_id=<AWS 访问密钥>
aws_secret_access_key=<AWS 私有密钥>

$ cat ~/.aws/config
[default]
region=<AWS 区域>
```
将第2步的用户密钥与私钥替换配置文件中内容，并修改替换自己的AWS区域

#### 接口调用
```bash
aws sns publish --phone-number 086159****1291 --message "在这里输入短信内容"
```
注意：手机号码需要加086

> 参考

> - https://docs.aws.amazon.com/zh_cn/cli/latest/userguide/install-bundle.html
> - https://docs.aws.amazon.com/zh_cn/cli/latest/userguide/cli-configure-files.html
> - https://docs.aws.amazon.com/zh_cn/cli/latest/userguide/cli-usage-help.html
> - https://docs.aws.amazon.com/cli/latest/reference/sns/publish.htm
