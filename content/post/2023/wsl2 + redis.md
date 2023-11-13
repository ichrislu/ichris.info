---
title: "在WSL上折腾Redis"
date: "2023-11-13"
lastMod: "2023-11-13"
categories: ["it"]
tags: ["windows", "wsl", "redis"]
---

就剩一台机器了，因为当时购买的时候没有注意看，华为用国产的供应商，一大堆硬件找不到Linux驱动，只能安装Windows。但是Redis没有Windows发行版，不想再用Docker了，试试WSL上安装Redis。于是就有了这个简单的问题，折腾了一天的故事。 

## 关于网络问题
最开始的问题是顺利启动，，本机用redis-cli可以访问，但其他内网机器无法访问，后来反应过来，这是WSL，跟虚拟机差不多，肯定是无法访问的。于是开始折腾网络模式，尝试在Hyper-V里新建交换机，在Windows用户主目录写配置文件`C:\Users\<UserName>\.wslconfig`，以及WSL的ubuntu中配置`/etc/wsl.config`，通通都失败！

最终找到一篇文章，是关于新版Windows WSL2 2.0.0的新特性：自带支持新的镜像网络，能解决所有网络相关问题，步骤：
- 更新23h2
- 更新WSL：`wsl --update --pre-release`
- 编辑文件%USERPROFILE%/.wslconfig，写入以下内容：
```
[experimental]
networkingMode=mirrored
dnsTunneling=true
firewall=true
autoProxy=true
```
全部搞定！效果：与主机IP相同，无额外的配置。

## 关于apt-get问题
最开始接触这个，还是在08年，密语：`本软件具有超级牛力`，有懂的同学吗？哈哈～～～～

后面十多年，几乎一直在用CentOS，今天遇上小难题了，忙活了半天。最开始安装redis-server，没有指定版本，给我初建了最新7.x，为了跟生产保持同步，得降版本到6.x。
卸载很容易，再重新安装就麻烦了，一直报依赖项redis-tools已经安装了更新的7.x版本，与当前依赖项版本不相符。尝试apt-get remove，但报错，说找不到该软件包，尝试更新源，仍然不行。最终偶然找到一篇文章可以解决，其实很简单，就是直接强制安装一次依赖的redis-tools，并指定版本为6.x，然后再安装软件就好了。