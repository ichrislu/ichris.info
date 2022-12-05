---
title: "MacOS常见问题"
date: "2017-06-13 12:24:25"
lastMod: "2017-06-13 12:24:25"
categories: ["it"]
tags: ["macOS"]
---

### 地址栏显示/隐藏完整路径
1. 显示完整路径
defaults write com.apple.finder _FXShowPosixPathInTitle -bool TRUE;killall Finder

2. 隐藏路径
defaults delete com.apple.finder _FXShowPosixPathInTitle;killall Finder

### 安装brew
官网：https://brew.sh/
先安装brew：/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

### 支持rar解压
brew安装rar：brew install unrar
解压：unrar x 文件名.rar

### .DS_Store文件
.DS_Store是Mac OS保存文件夹的自定义属性的隐藏文件，如文件的图标位置或背景色，相当于Windows的desktop.ini。
1. 禁止.DS_store生成：
打开 “终端” ，复制黏贴下面的命令，回车执行，重启Mac即可生效。
defaults write com.apple.desktopservices DSDontWriteNetworkStores -bool TRUE

2. 恢复.DS_store生成：
defaults delete com.apple.desktopservices DSDontWriteNetworkStores