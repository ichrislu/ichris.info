---
title: "转：go语言内置函数new()和struct{}初始化的区别"
date: "2019-07-15"
lastMod: "2019-07-16"
categories: ["it"]
tags: ["golang", "new", "struct"]
---

**new()** 
这是一个用来分配内存的内置函数，它的参数是一个类型，不是一个值，它的返回值是一个指向新分配的 t 类型的零值的指针。

在golang的代码定义如下：

```go
func new(t Type) *Type
```

**strut{}** 
直接使用struct{} 来初始化strut时，返回的是一个struct类型的值，而不是指针。

两者对比代码如下：

```go
type Student struct{
	id int
	name string
}

func main(){
    var s_1 *Student = new(Student) # 注：这里经常习惯使用go的类型推断，而写成s_1 := new(Student)，这很容易让人忽略new()返回的是指针而非该类型的值
	s_1.id = 100
	s_1.name = "cat"
	var s_2 Student = Student{id:1,name:"tom"}
	fmt.Println(s_1,s_2)
}

# 输出结果
&{100 cat} {1 tom}
```

结果中就可以看出s_1的类型为指针，s_2为一个Student类型



> 原文参考：https://blog.csdn.net/happinessaflower/article/details/44658041