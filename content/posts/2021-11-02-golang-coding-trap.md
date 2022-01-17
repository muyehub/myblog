---
layout:     post
title:      "golang 编程陷阱---摘自《go 专家编程》"
date:       2021-11-02 23:47:33 +0800
author:     "max"
---

### 关于切片扩容

在使用 append 向 slice 中追加元素时，如果 slice 容量不足以存储新元素，则会把当前切片扩容并产生一个新的切片。如下：

```go
package main

import "fmt"

func main() {
   s := make([]int, 0)
   w := append(s, 1)
   t := append(s, 2)
   v := append(s, 3)
   fmt.Printf("%p, %p , %p , %p, %p", s, w, t, v)
}
// 0x116ce80, 0xc00001c198 , 0xc00001c1a0 , 0xc00001c1a8
```

append 会每次都生成一个新的切片，并且每次都会做扩容操作，元切片 s 并没有改变任何。

所以，在操作中要将 append 的返回值接收，如果不接收编译器会报错，如果用 _ 忽略返回值，则需要考虑扩容的情况，避免滥用。

### 空切片

向 slice 中 append 空值(nil)时也会增加 slice 的长度，这在有些情况下可能会导致严重的错误，并且不易察觉，书中举了一个 Kubernetes 项目中的例子。

有一个错误收集器，用来将验证函数的一系列错误收集到一个 slice 中，检查 slice 的长度如果大于 0 时，说明有错误产生，则程序退出，如下：

```go
package main

import (
   "errors"
 "os")

func main() {
   var errs []error
   errs = append(errs, ValidateName("张三"))

   if len(errs) > 0 {
      println(errs)
      os.Exit(1)
   }
}

func ValidateName(name string) error {
   if name != "" {
      return nil
 }

   return errors.New("empty name")
}
```

在使用 append 函数时，谨记 append 可能会追加 nil 值，应该尽量避免追加无意义的元素。

### append 的本质

如下代码的执行说明了 append 的执行逻辑：

```go
package main

import "fmt"

func main() {
   x := make([]int, 0, 10)
   x = append(x, 1, 2, 3)
   y := append(x, 4)
   z := append(x, 5)
   fmt.Println(x, y, z)
}
// [1 2 3] [1 2 3 5] [1 2 3 5]
```

在这段代码中，x 除了最开始的 append，在后面两次 append 中并没有变化，len 依然是 3，cap 依然是 10，append 函数的入参为切片，而切片只是一个 struct 数据结构，参数传递时发生了值拷贝，所以 append 无法操作原切片。

### 循环变量引用

```go
package main

import "fmt"

func main() {
   var out []*int
   for i := 0; i < 3; i++ {
      out = append(out, &i)
   }
   fmt.Println("Values:", *out[0], *out[1], *out[2])
}
// Values: 3 3 3
```

在 go 中，循环变量 i 只分配一次地址，在三次循环中，i 中存储的值分别是 0，1，2，但是 i 的地址没有变化。

因为 append 的是 i 的地址，而经过 for 循环后 i 的地址中最终存的数据是 3，所以最终的结果是 3 个 3。

如果需要以指针的形式存放循环变量，则可以显式的拷贝一次：

```go
for i := 0; i < 3; i++ {
	iCopy := i
  out = append(out, &iCopy)
}
```

另外一种解决方案是修改切片的类型，避免存储指针。
