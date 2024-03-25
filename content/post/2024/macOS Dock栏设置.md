---
title: "macOS Dock栏设置"
date: "2024-03-25"
lastMod: "2024-03-25"
categories: ["it"]
tags: ["macOS", "Dock"]
---

接扩展显示器的时候，Dock栏会在不同显示器之间切换展现，忍受这个问题很久了，今天花了一点时间找解决方案，最终还是无解，但是找到了另一种方式。

正常情况下，隐藏的Dock栏要显示出来，需要放到Dock显示位置1-2秒，这个等待很无聊，找到了设置时长的方法：

```zsh
defaults write com.apple.Dock autohide-delay -float 0 && killall Dock
```

经测试，Dock栏展现的等待时长的问题解决了，其实Dock栏就可以默认隐藏了，也就不会存在之前提前在不同显示器之间切换的问题了。