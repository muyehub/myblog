---
layout:     post
title:      "用 go 实现一个简易聊天室"
date:       2022-01-22 16:27:33 +0800
author:     "max"
---
### 概述 
first，我们需要创建一个房间，可以使用 net 包直接启动一个常驻的 tcp 服务
second，我们需要有用户的信息，用户通过访问 tcp 服务来进入房间
end，我们需要在用户进入/离开房间时对其他用户进行广播，还需要将用户在房间内产生的消息发送到房间里让其他用户看见

### 实现 
我们通过设定三个 channel 来分别标记用户进入房间、用户离开房间和用户广播消息的存储。
首先，我们创建一个 tcp 服务器，新建一个 server.go 的文件，然后通过 net 包来初始化 tcp 服务器，端口就选择 2022。监听器创建完毕后我们可以看到它里面有三个方法，分别是 Accept、Close 和 Addr，Close 就是用来关闭这个端口监听，Addr 就是返回监听器所对应的 ip，这里默认的话也就是本地的 ip，Accept 方法就是返回一个连接，一旦有人请求这个文件，就意味着产生一个连接，如果没有人请求那么 Accept 就会等待，这里我们用一个死循环来让它可以重复的接受多个连接请求。
```go
	listener, err := net.Listen("tcp", ":2022")  
	if err != nil {  
		 panic(err)  
	}
	for {  
		 conn, err := listener.Accept()  
		 if err != nil {  
			 log.Println(err)  
			 continue  
		 }
		  
		 go handleConn(conn)  
	}
```

然后就是 handleConn 这个方法，将每次的连接传入进去，然后初始化连接的用户信息，再通过一个协程将用户消息通道里的信息拿出来消费，接下来做的就是向用户进入/离开的消息 channel 中写入该用户已经上线或用户离开的消息，然后再将用户记录到用户列表中。最后通过扫描该连接来读取用户的输入内容。
```go
func handleConn(conn net.Conn) {  
	 defer conn.Close()  
	  
	 // 1. 新用户进来，构建该用户的实例  
	 user := &User{  
		 ID:             GenUserID(),  
		 Addr:           conn.RemoteAddr().String(),  
		 EnterAt:        time.Now(),  
		 MessageChannel: make(chan string, 8),  
	 }  
	  
	 // 2. 由于当前是在一个新的 goroutine 中进行读操作的， 所以需要开一个 goroutine 用户写操作。读写 goroutine 之间可以通过 channel 进行通信  
	 go sendMessage(conn, user.MessageChannel)  
	  
	 // 3. 给当前用户发送欢迎信息，向所有用户告知新用户到来  
	 user.MessageChannel <- "Welcome, " + user.String()  
	 messageChannel <- "user:`" + strconv.Itoa(user.ID) + "` has enter"  
	  
	 // 4. 记录到全局用户列表中，避免用锁  
	 enteringChannel <- user  
	  
	   // 5. 循环读取用户输入  
	 input := bufio.NewScanner(conn)  
	 for input.Scan() {  
		 messageChannel <- strconv.Itoa(user.ID) + ":" + input.Text()  
	 }  
	 if err := input.Err(); err != nil {  
		 log.Println("读取错误：", err)  
	 }  
	 // 6. 用户离开  
	 leavingChannel <- user  
	 messageChannel <- "user:`" + strconv.Itoa(user.ID) + "` has left"  
}
```

上面这段代码有严格的执行顺序，我们可以看到最后连用户离开消息也一并发送但是却没有看到判断连接断开的代码，那是因为上面监听用户输入的代码阻塞了流程，当用户断开时自然就会往下走了。如果要手动判断 net.Conn 是否断开，然后再继续操作的话，可以使用一个 byte 类型的切片，将当前时间通过 net 包下的 SetReadDeadline 函数传入，然后使用 net 包下的 read 方法来读取这个 byte 切片，当发生错误信息并且错误信息是 io.EOF 类型时，说明连接已经断开，具体代码可以参考这里：[Best way to reliably detect that a TCP connection is closed](https://web.archive.org/web/20140801101158/http://osdir.com/ml/go-language-discuss/2012-08/msg01586.html)

接下来我们来定义 client.go 的客户端文件，同样的也是监听 2022 端口，不同的是 server 是创建监听来监听连接，而 client 是直接连接到当前地址的对应端口，这时 server 端就会收到一个连接的请求从而创立 tcp 连接。接下来创建了一个协程，主要是用 io.Copy 操作将用户的输入拷贝到连接里一份，同时创建一个类型为结构体的 无缓冲channel，在操作完毕后将 channel 弹空，这样保证程序的顺序执行。
```go
	func main() {  
		 conn, err := net.Dial("tcp", ":2022")  
		 if err != nil {  
			 panic(err)  
		 }  
		 done := make(chan struct{})  
		 go func() {  
			 io.Copy(os.Stdout, conn)  
			 log.Println("done")  
			 done <- struct{}{}  
		 }()  
		 mustCopy(conn, os.Stdin)  
		 conn.Close()  
		 <-done  
	}  
	  
	func mustCopy(dst io.Writer, src io.Reader) {  
		if _, err := io.Copy(dst, src); err != nil {  
			 log.Fatal(err)  
		}
	}
```

主要的部分就这些了，当然还有一些细致末节，比如还应该有一个 broadcaster 函数，来读取那三个消息 channel 进行处理。完整的代码在[这里](https://github.com/muyehub/simple_chatroom)，以上内容是阅读《go 编程之旅》第四章时的感悟，以上。

