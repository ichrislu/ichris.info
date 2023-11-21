---
title: "linux命令行不显示当前路径"
date: "2014-11-13"
lastMod: "2014-11-13"
categories: ["it"]
tags: ["linux"]
---

### linux命令行不显示当前路径
编辑：`~/.bash_profile`，添加如下内容：`export PS1='[\u@\h \w]\$'`

### 以下方式，待验证发行版兼容性
cp /etc/skel/.{bash_profile,bashrc} ~
source ~/.bashrc