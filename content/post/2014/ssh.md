---
title: "ssh登录"
date: "2014-11-13"
lastMod: "2018-09-14"
categories: ["it"]
tags: ["linux", "ssh", "ssh-keygen", "ssh-copy-id"]
---

### SSH配置文件sshd_config常见修改

1. 禁止root用户远程登录  
PermitRootLogin项，值改为no，重启ssh服务生效

2. SSH登录慢  
UseDNS配置项，值为no，重启ssh服务生效  
修改配置后仍然慢，找到GSSAPIAuthentication配置项，值为no

3. 只允许指定主机连接  
在文件最后面另起一行添加`AllowUsers deploy@183.21.89.249`，即：指定deploy用户在183.21.89.249机器上连接

4. 仅允许特定的用户通过SSH登陆  
如果你想让一列用户都能通过SSH登陆，你可以在sshd_config文件中指定它们，例如：我想让用户anze、dasa、kimy能通过SSH登陆，在sshd_config文件的末尾我添加下面这样一行：`AllowUsers anze dasa kimy`

重启ssh服务，生效配置`/etc/rc.d/sshd restart`，新版本可能是`sudo systemctl restart sshd` 或者 `sudo service sshd restart`
### SSH密钥对生成
运行命令`ssh-keygen`，系统会提示输入私钥文件保存路径和文件名（默认为~/.ssh/id_rsa），以及私钥的密码（可选）。

### 复制公钥到远程服务器
`ssh-copy-id user@remote_host`