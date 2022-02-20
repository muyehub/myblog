---
title: "由 append 引发的一个疑问"
date: 2022-02-20T18:02:55+08:00
draft: true
---

#### 由 append 引发的一个疑问

我们先来看一个问题
```Go
package main

import "fmt"

func main() {
 a := make([]int, 0, 5)
 AddElm(a, 5)
 fmt.Println(a)
}

func AddElm(a []int, i int) {
 a = append(a, i)
}
```
上面这一段代码中，a 这个切片会输出什么呢？可以试着运行一下

答案是空切片，为什么呢？首先我们知道 append 这个操作，在容量足够的情况下是不会新生成一个 slice 来进行扩容的，所以这里排除这种情况。

已知这个 slice 也是值传递的方式传入的，那么是不是因为他 copy 了一份数据到函数内部从而导致的函数内外不一致的情况呢？我们打印地址看一下

```Go
package main

import "fmt"

func main() {
 a := make([]int, 0, 5)
 fmt.Printf("before: %p\n", a)
 AddElm(a, 5)
 fmt.Printf("after: %p\n", a)
 fmt.Println(a)
}

func AddElm(a []int, i int) {
 a = append(a, i)
 fmt.Printf("under: %p\n", a)
}

// 输出如下：
// before: 0xc0000a8030
// under: 0xc0000a8030
// after: 0xc0000a8030
// []
```

可以发现，在申明开始-函数内-函数追加赋值结束时打印，这三个地址都是一样的。

那么为什么产生这样的原因呢，就需要从 slice 的底层结构说起了，slice 是一个结构体，运行时的结构体在 reflect.SliceHeader 中，SliceHeader 结构体如下：
```Go
type SliceHeader struct {
 Data uintptr
 Len  int
 Cap  int
}
```

这里的 Data 是一个指向底层数组的指针，所以我们上面的代码打印出来的地址，会不会是 Data 的值而不是 slice 这个数据本身真正的地址呢(因为按照我理解，这个值传递传递到函数内部肯定是和外面的 slice 地址不同的)？我们来试一下，打印一下 SliceHeader 结构体中 Data 的值看看
```Go
package main

import (
 "fmt"
 "reflect" "runtime" "unsafe")

func main() {
 runtime.GOMAXPROCS(1)
 a := make([]int, 0, 5)
 source := (*reflect.SliceHeader)(unsafe.Pointer(&a))
 fmt.Printf("before: %p, source: %x\n", a, source.Data)
 AddElm(a, 5)
 fmt.Printf("after: %p, source: %x\n", a, source.Data)
 fmt.Println(a)
}

func AddElm(a []int, i int) {
 a = append(a, i)
 source := (*reflect.SliceHeader)(unsafe.Pointer(&a))
 fmt.Printf("under: %p, source: %x\n", a, source.Data)
}

// before: 0xc00001c1b0, source: c00001c1b0
// under: 0xc00001c1b0, source: c00001c1b0
// after: 0xc00001c1b0, source: c00001c1b0
// []

```
%x 占位符是将 Data 中存储的数组地址转成 16 进制，我们可以看到通过 %p 占位符获取到的指针就是拿的 Data 的值，那如果我们想拿这个 slice 数据本身地址的呢？这里只需要将 a 变为 &a，就可以拿到了，a 其实是 slice 的数据地址，而 &a 才是 slice 本身的指针地址(有点绕？)。接下来我们就打印一下 &a 的值
```Go
package main

import (
 "fmt"
 "reflect" "runtime" "unsafe")

func main() {
 runtime.GOMAXPROCS(1)
 a := make([]int, 0, 5)
 source := (*reflect.SliceHeader)(unsafe.Pointer(&a))
 fmt.Printf("before: %p, source: %x\n", &a, source.Data)
 AddElm(a, 5)
 fmt.Printf("after: %p, source: %x\n", &a, source.Data)
 fmt.Println(a)
}

func AddElm(a []int, i int) {
 a = append(a, i)
 source := (*reflect.SliceHeader)(unsafe.Pointer(&a))
 fmt.Printf("under: %p, source: %x\n", &a, source.Data)
}

// before: 0xc00000c048, source: c00001c1e0
// under: 0xc00000c060, source: c00001c1e0
// after: 0xc00000c048, source: c00001c1e0
// []
```
这下就不一样了，说明 append 确实新生成了一个 slice，但我们的 slice 容量足够，为什么会重新生成呢？我们再把 len 和 cap 一同打印出来
```Go
package main

import (
 "fmt"
 "reflect" "runtime" "unsafe")

func main() {
 runtime.GOMAXPROCS(1)
 a := make([]int, 0, 5)
 source := (*reflect.SliceHeader)(unsafe.Pointer(&a))
 fmt.Printf("before: %p, source: %x, len: %d, cap: %d\n", &a, source.Data, source.Len, source.Cap)
 AddElm(a, 5)
 fmt.Printf("after: %p, source: %x, len: %d, cap: %d\n", &a, source.Data, source.Len, source.Cap)
 fmt.Println(a)
}

func AddElm(a []int, i int) {
 a = append(a, i)
 source := (*reflect.SliceHeader)(unsafe.Pointer(&a))
 fmt.Printf("under: %p, source: %x, len: %d, cap: %d\n", &a, source.Data, source.Len, source.Cap)
}

// before: 0xc00000c018, source: c00001c1b0, len: 0, cap: 5
// under: 0xc00000c030, source: c00001c1b0, len: 1, cap: 5
// after: 0xc00000c018, source: c00001c1b0, len: 0, cap: 5
// []

```
这下就清晰了，因为值传递，go 把 slice 的结构体复制了一份传给函数 AddElm，结果函数向这个复制出来的结构体添加了一个数据，导致 len 加 1，这就导致 0c030 的这个 slice 和原先的 slice 彻底不是一回事了。

我最早开始觉得是函数作用域的问题，在函数内部修改的东西不会影响函数外部，但现在看来是会影响的，虽然函数内部的 slice 在内存地址上与函数外部的地址不同，但slice 是引用类型，修改函数内部的 slice 等同于修改函数外的。不过在这个例子中 append 让函数内外的两个 slice 彻底不一样了，就只能各自修改各自生效了。

这是我对于这个问题的浅显理解，感觉如果要彻底深入理解一步一步走下来还是需要去看 compile 过程和 runtime 过程，继续学习吧...
