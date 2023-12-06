---
title: "Git创建用户key"
date: "2017-07-21"
lastMod: "2017-07-21"
categories: ["it"]
tags: ["git", "bash", "ssh-keygen"]
---

## Git创建用户key

以Git bash ssh为例：

进入Git bash ssh目录，一般为`C:\Users\[用户]\.ssh`或`/home/[用户]/.ssh`
```bash
ssh-keygen -t rsa -b 4096 -C "name@domain.com"
```

注：生成文件名的时候最好不要修改文件名，会导致提示输入密码（即失败）

**复制用户key（windows）**
```bash
clip < ~/.ssh/id_rsa.pub
或
cat ~/.ssh/id_ed25519.pub | clip
```