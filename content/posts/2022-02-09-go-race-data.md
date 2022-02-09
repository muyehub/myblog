---
title: "golang 中提防竞态条件的产生"
date: 2022-02-09T22:33:25+08:00
draft: true
---
我们来看下面这段代码：
```Go
package main

import (
	"time"
)

type testInt struct {
	Num int
}

func main() {
	a := make([]int, 0)
	for i := 1; i < 10; i++ {
		a = append(a, i)
	}
	t := testInt{Num: 1}

	for _, v := range a {
		go func(i int) {
			t.Num = i
			pTs(t)
		}(v)
	}
	time.Sleep(time.Second*3)
}

func pTs(t testInt) {
	println(t.Num)
}
```
你可以复制出来执行一下，会不会有问题呢？从执行的结果来看，是没问题的。
但是如果你要加上 -race 参数去执行，就会发现在 t.Num = i 这里报错了，这是为什么呢？
这就要说到一个 go 语言的编程常识了(可能不限语言)，就是在多个协程去处理一个变量的时候记得要加锁或者用 atomic 进行原子操作，不管结果是否正常。因为如果不这么做，有天出了问题可是不好排查的。

建议大家拿到陌生代码，都 -race 去检测一下，就是这样。

对于为什么不加锁执行起来也没有问题，我还没有想明白，希望有大明白能指点一二。

