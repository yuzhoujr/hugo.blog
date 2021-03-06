+++
date = "2013-10-09T20:29:40+08:00"
menu = "main"
tags = ["gobyexample", "golang"]
title = "gobyexample - 9 - Slice"

+++

在Go语言中Slice是一个主要的数据结构，你将会经常用到，它提供了比数组更强大的接口

	package main

	import "fmt"

	func main() {
		// 还记得数组的类型是由什么决定么？数组的长度+数组包含的元素类型
		// 与数组不同，Slice的类型只由它包含的元素决定
		// 通常我们使用make来创建一个Slice
		// 此处是用make创建了一个包含3个string的Slice
		// 没有初始化，所以是3个string的零值（即3个空字符串）
		s := make([]string, 3)
		fmt.Println("emp:", s)

		// 就像数组一样对Slice进行存取操作
		s[0] = "a"
		s[1] = "b"
		s[2] = "c"
		fmt.Println("set:", s)
		fmt.Println("get:", s[2])

		// len返回Slice的长度
		fmt.Println("len:", len(s))

		// 除了上面提到的基本操作之外，Slice还支持一些其他方法
		// 比如内置的：append，append会返回一个包含了一个或多个新元素的
		// 新的（其实也有可能是原来的Slice，也可能是新的）Slice
		// 所以append之后我们要接收其返回的新Slice，请Python/Java码农注意
		s = append(s, "d")
		s = append(s, "e", "f")
		fmt.Println("apd:", s)

		// Slice可以使用copy方法做拷贝
		// 此处我们先创建了一个与s相同长度的空Slice c
		// 然后把s的内容拷贝到了c中
		c := make([]string, len(s))
		copy(c, s)
		fmt.Println("cpy:", c)

		// Slice支持切片操作（称之为reslice），使用这样的语法：slice[low:high]
		// 比如下面这个例子，我们得到一个新的Slice l，里边包含s[2], s[3], s[4]三个元素
		// 注意，包含slice[low]不包含slice[high]
		l := s[2:5]
		fmt.Println("sl1:", l)

		// 如果冒号前面的数字不写，那就是从0开始
		l = s[:5]
		fmt.Println("sl2:", l)

		// 如果冒号后面的数字不写，那就是Slice的长度
		// 即从low往后的所有元素
		l = s[2:]
		fmt.Println("sl3:", l)

		// 就像数组，我们可以在一行内声明并初始化一个slice
		t := []string{"g", "h", "i"}
		fmt.Println("dcl:", t)

		// Slice可以被组合成多维数据结构。
		// 与多维数组不同的是：内层的Slice可以具有不同的长度
		twoD := make([][]int, 3)
		for i := 0; i < 3; i++ {
			innerLen := i + 1
			twoD[i] = make([]int, innerLen)
			for j := 0; j < innerLen; j++ {
				twoD[i][j] = i + j
			}
		}
		fmt.Println("2d: ", twoD)
	}

以上程序的运行结果：

	$ go run slices.go
	emp: [  ]
	set: [a b c]
	get: c
	len: 3
	apd: [a b c d e f]
	cpy: [a b c d e f]
	sl1: [c d e]
	sl2: [a b c d e]
	sl3: [c d e f]
	dcl: [g h i]
	2d:  [[0] [1 2] [2 3 4]]

虽说Slice和Array是不同的数据结构，但是通过`fmt.Println()`打印的效果是一样的

关于Slice的设计和实现更详细的内容，请参看Go语言团队的 [这篇文章](http://blog.golang.org/2011/01/go-slices-usage-and-internals.html)

