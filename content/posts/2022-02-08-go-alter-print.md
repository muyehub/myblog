---
title: "控制 goroutine 执行顺序的一例"
date: 2022-02-08T22:14:29+08:00
draft: true
---

### 依次打印
题目：依次输出 dog、pig、sheep，并执行 100 次，每个输出都需要一个单独的 goroutine

这里我们通过有缓冲的容量为 1 的 channel 来标记状态，通过 for-select 结构来控制当前要输出哪一个单词，因为 select 在 channel 阻塞的时候不会执行对应的 case，我们可以通过这个特性来操作。

具体思路如下：
* 设置三个 channel，初始化时使 dog 的 channel 中有一条数据，这样 select 就会先输出 dog
* 然后在 dog 的 case 中结束的时候给 pig 的 channel 中新增一条数据，这样下次就会进入 pig 的 case 进行输出操作
* 过程中我们可以使用 atomic 包来控制次数，不会产生并发问题

具体代码如下：
```Go
package main

import
(
	"fmt"
	"sync"
	"sync/atomic"
)

var wg sync.WaitGroup
var counter uint64 = 0
var dogChan = make(chan struct{}, 1)
var pigChan = make(chan struct{}, 1)
var sheepChan = make(chan struct{}, 1)

func main() {
	wg.Add(1)
	go animalPrint()
	dogChan <- struct{}{}
	wg.Wait()
}

func animalPrint() {
	for {
		if counter == uint64(300) {
			wg.Done()
			return
		}

		select {
		case <-dogChan:
			fmt.Println("dog")
			atomic.AddUint64(&counter, 1)
			pigChan <- struct{}{}
		case <-pigChan:
			fmt.Println("pig")
			atomic.AddUint64(&counter, 1)
			sheepChan <- struct{}{}
		case <-sheepChan:
			fmt.Println("sheep")
			atomic.AddUint64(&counter, 1)
			dogChan <- struct{}{}
		}
	}
}
```

是不是觉得重复代码有点多？
我们还是依照上面的思路，把代码换种方式实现：
```Go
package main

import
(
	"fmt"
	"sync"
	"sync/atomic"
)

var wg sync.WaitGroup
var counter uint64 = 0
var dogChan = make(chan struct{}, 1)
var pigChan = make(chan struct{}, 1)
var sheepChan = make(chan struct{}, 1)

func main() {
	wg.Add(3)
	dogChan <- struct{}{}
	go animalPrint2("dog", dogChan, pigChan)
	go animalPrint2("pig", pigChan, sheepChan)
	go animalPrint2("sheep", sheepChan, dogChan)
	wg.Wait()
}

func animalPrint2(name string, curChan, nextChan chan struct{}) {
	cnt := 0
	for {
		if cnt == 100 {
			wg.Done()
			break
		}
		<-curChan
		cnt++
		println(name)
		nextChan <- struct{}{}
	}
}
```

### 交替打印
我们把问题扩展一下，如果打印的顺序是交替打印呢？还可以用这样的思路实现么？

问题：交替打印数字和字母，实现如下效果：
12AB34CD56EF78GH910IJ1112KL1314MN1516OP1718QR1920ST2122UV2324WX2526YZ

思路一样，代码如下：

```Go
package main

import
(
	"fmt"
	"sync"
)

var wg sync.WaitGroup
var numChan = make(chan struct{}, 1)
var byteChan = make(chan struct{}, 1)

func main() {
	wg.Add(2)
	numChan <- struct{}{}
	go numPrint()
	go bytePrint()
	wg.Wait()
}

func numPrint() {
	i := 1
	for {
		if i == 27 {
			wg.Done()
			break
		}
		<-numChan
		println(i, i+1)
		i += 2
		byteChan <- struct{}{}
	}
}

func bytePrint() {
	i := 'A'
	for {
		if i > 'Z' {
			wg.Done()
			break
		}
		<-byteChan
		fmt.Printf("%c%c", i, i+1)
		i += 2
		numChan <- struct{}{}
	}
}
```

如果你有其他更好的思路，希望你告诉我，这个确实有意思
