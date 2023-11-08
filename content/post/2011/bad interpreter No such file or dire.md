---
title: "/bin/sh^M: bad interpreter: No such file or dire"
date: "2011-01-26"
lastMod: "2011-01-26"
categories: ["it"]
tags: ["linux", "shell", "编码"]
---

在Linux中执行shell脚本，异常/bin/sh^M: bad interpreter: No such file or directory。 

### 分析：
这是不同系统编码格式引起的：在windows系统中编辑的.sh文件可能有不可见字符，所以在Linux系统下执行会报以上异常信息。 

### 解决：
1. 在windows下转换：
利用一些编辑器如UltraEdit或EditPlus等工具先将脚本编码转换，再放到Linux中执行。转换方式如下（UltraEdit）：File-->Conversions-->DOS->UNIX即可。

2. 在Linux中转换：
```shell
chmod a+x <filename> # 首先要确保文件有可执行权限
vim <filename> # 然后修改文件格式
:set ff # 查看文件格式，或:set fileformat
```
可以看到如下信息：
fileformat=dos 或 fileformat=unix

利用如下命令修改文件格式
```shell
:set ff=unix #或:set fileformat=unix
:x 保存退出
```