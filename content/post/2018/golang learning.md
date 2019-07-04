---
title: "golang learning"
date: "2018-10-17 16:21:00"
lastMod: "2018-10-19 14:28:00"
tags: ["go"]
---

##### 一个有关Golang变量作用域的坑

对于使用:=定义的变量，如果新变量p与那个同名已定义变量 (这里就是那个全局变量p)不在一个作用域中时，那么golang会新定义这个变量p，遮盖住全局变量p

https://studygolang.com/articles/2215

##### cannot find package golang.orgxtext... 或者golang.orgxnet...的解决方法

1. 在github上，找golang的托管代码，应该是在github.com/golang下，找一下text或者net的目录。
2. 下载下来，拷贝到$GOROOT/src/golang.org/x下面对应的目录下，就ok了

> 参考
>
> https://blog.csdn.net/dong_beijing/article/details/79653327
> https://blog.csdn.net/qq_35191331/article/details/79655839