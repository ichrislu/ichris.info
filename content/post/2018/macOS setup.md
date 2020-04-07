---
title: "macOS安装配置和软件"
date: "2018-12-26 21:44:55"
lastMod: "2020-04-07 10:54:25"
tags: ["macOS", "linux"]
---

### 系统设置

#### 程序坞
- 勾选放大，以及设置合适的大小
- 勾选将窗口最小化为应用程序图标
- 取消勾选在程序坞中显示最近使用的应用程序

#### 语言与地区
- 时间格式勾选24小时制

#### 安全性与隐私

#### 通知
- 勿扰模式：

#### 显示器
- 夜览：日落到日出

#### 键盘
- 快捷键：全键盘控制 -> 所有控制

#### 触控板
- 光标与点按
  - 勾选轻点来点按
  - 跟踪速度：倒数第2
- 更多手势：
  - 在全屏幕显示的应用之间轻扫：四指
  - 高度中心：四指向上
  - 应用Exporse：四指向下

#### 声音
- 输出：在菜单栏中显示音量

#### 用户与群组
- 关闭客人用户

#### 日期与时间
- 时钟：显示秒钟、闪动、使用24小时格式等

#### Finder
- 通用：开启新“仿达”窗口时打开：home
- 边栏：个人习惯
- 高级：将以下位置的文件夹保持在顶部：按名称排序时的窗口中
- 打开一个仿达窗口：点状态栏，显示路径栏和显示状态栏；点击查看显示选项：排序方式为名称

#### 辅助功能
- 鼠标与触控板
  - 启用拖移：使用拖移锁定

**关于系统设置，写到最后的感受，也许"迁移助理"是个很不错的工具和选择！**

---

### 软件

#### 常规软件
- 清歌五笔
- Alfred
- Typora
- ~~XMind Zen，幕布~~，已换为WPS
- AppCleaner
- Microsoft Remote Desktop（仅海外App Store有，不能下载，当前是下载的测试版）
- IINA
- ~~CotEditor~~，已换为MacVim + Noto
- ~~FreeDownloadManager~~，已换为Motrix
- The Unarchiver，eZip
- 有道云笔记

#### 常规开发工具
- Postman
- Goland/IntelliJ IDEA/PyCharm/VS Code
- Sequel Pro
- Sourcetree
- Git
- AnotherRedisDesktopManager
- ~~Termius~~，已换为iTerm2 + profile

#### ShadowsocksX-NG
以前一直用的是ShadowsocksX，这次重装改用了ShadowsocksX-NG，据[官网](https://github.com/shadowsocks/ShadowsocksX-NG)介绍，后者是用来替代前者的，同样遇到了小坑。

第1次安装完，启动发现间隙性不能使用，一直找不到原因，于是暂时先停掉ShadowsocksX-NG，换回了ShadowsocksX。
后来受[这篇文章] (https://www.twisted-meadows.com/shadowsocksx-ng/)启发，发日志发现果然是端口冲突。之前间隙性不能使用的原因很有可能是第1次运行可以，然后开启了ShadowsocksX打算导入配置而造成了端口冲突，因为同时侦听了1080的代理端口。停用和删除ShadowsocksX后，果然正常。
启动ShadowsocksX-NG，点击状态栏图标，选择“偏好设置”，修改常规中Socks5监听端口号：1080

#### 越来越好
导入配置就好了

#### MySQL
- 安装到最后一步，会弹出提示框，告之生成了密码，需要保存下来。
- 使用保存的密码登录，提示密码过期。
- 初始安装mysql工具没有加到PATH，可以手动添加，也可以直接启动：/usr/local/mysql/bin/mysql。参考：https://blog.csdn.net/sinat_26696701/article/details/49619207/
- 用mysql工具使用保存的密码登录，然后马上修改密码：ALTER USER USER() IDENTIFIED BY '密码写在这里'; 参考：https://majing.io/posts/10000005531181

#### homebrew
- 官网：https://brew.sh
- 安装telnet wget zssh

#### Alfred

##### 常规设置

- TODO

##### workflow

- Coversion：进制转换
- Hash：算Hash
- IP Address：获取IP
- Password Generator：生成随机密码
- Restart App：重启程序
- Terminal Finder：Terminal和Finder切换
- wechatTool：微信工具
- Youdao Translate：有道翻译

#### zsh

##### 切换默认Shell到Zsh
Mac OS X默认已经安装好了Zsh，你可以打开终端，输入zsh --version来确认，如果没有安装，请参考这个文档。
打开终端输入下面的命令，切换默认Shell为Zsh：
```shell
chsh -s /bin/zsh
```

关闭终端重新打开后，你将默认使用zsh作为终端Shell。然而你会发现，终端并没有变得多酷炫，接着往下走
##### 安装Oh My ZSH!
打开终端输入下面的命令：
```shell
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

耐心等待一会儿，然后你就会发现你的终端变成了彩色的了。

#### iTerm2
##### 配置
- 安装iTerm2之后，发现option+←和option+→这两组快捷键并不能实现光标按照单词快速移动，因为iTerm2对将这两个快捷键做了其他用途，修改如下：
cmd + ,进入设置，找到Profile，选择Profile，比如default，找到keys选项卡，在Key Mappings中分别找到option+←和option+→，选择Action为Send Escape Sequence，下面的ESC+分别输入b和f；修改完成重启即可
- 默认通过ssh连接服务器会超时，可以通过修改配置文件：/etc/ssh/ssh_config，在Host 下添加ServerAliveInterval 60即可，如：
```conf
Host *
        SendEnv LANG LC_*
        ServerAliveInterval 60
```
或修改用户级配置文件~/.ssh/config（没有就创建）
```conf
Host *
     ServerAliveInterval 60
```

> 官网：www.iterm2.com

#### finder和shell切换

以下两个软件被Alfred的workflow替代

##### ~~go2shell~~

- ~~官网：zipzapmac.com/go2shell~~
- ~~安装后打开app，设置并运行~~
- ~~注意事项：~~
  1. ~~在APP Store安装的不行，并不会弹出官网的设置界面，但是官网下载的可以~~
  2. ~~在安装iTerm2之前，不要打开APP设置（选择Terminal），否则在Finder中出现的图标为一个？号，测试环境Mojave 10.14.2。安装iTerm2后，选择iTerm2正常~~
  3. ~~如果出现Finder多余的？号，可以右南工具栏，选择自定义工具栏，将多余的？号删除即可（拉下来）~~

##### ~~FinderGo~~
- ~~官网：https://github.com/onmyway133/FinderGo~~
- ~~按官方使用说明，安装后打开，获取系统运行权限(右键打开更好)，然后接Cmd将程序图标拖入Finder工具栏即可；支持Terminal、iTerm、Hyper。~~
- ~~可替换go2shell了，因为相比较go2shell，体验稍好，软件在维护。可换图标和动态切换不同终端~~
