---
layout: post
title: cgi, fastcgi, php-cgi, php-fpm的关系
date:  2019-02-21 15:33:39 +0800
author: "max"
---

#### cgi
通用网关接口, Common Gateway Interface, CGI是Web服务器运行时外部程序的规范, 按CGI编写的程序可以扩展服务器功能. 所以, 广义上的cgi是一种接口标准, 不是字面意义上的接口. 狭义上, cgi就是cgi程序, 运行在服务器上, 提供同客户端HTML页面的接口. 绝大多数的cgi程序被用来解释处理来自表单的输入信息, 并在服务器产生相应的处理, 或将相应的信息反馈给浏览器, cgi程序使网页具有交互功能. 
cgi程序处理步骤:
1) 通过Internet把用户请求送到web服务器.
2) web服务器接受用户请求并交给CGI程序处理.
3) CGI程序把处理结果传送给web服务器.
4) web服务器把结果送回到用户.

#### fastcgi
快速通用网关接口, Fast Common Gateway Interface, CGI有很多缺点, 每接收一个请求就要fork一个进程处理, 只能接收一个请求作出一个响应, 请求结束后该进程就会结束. 而fastcgi会事先启动起来, 作为一个cgi的管理服务器存在, 使用进程/线程池来处理一连串的请求.
fastcgi程序处理步骤:
1) web服务器启动时载入fastcgi进程管理器
2) fastcgi自身初始化, 启动多个cgi解释器进程并等待来自web server的请求
3) 当请求来到web服务器时, web服务器通过socket请求fastcgi进程管理器, fastcgi进程管理器选择并连接到一个cgi解释器, web服务器将cgi环境变量和标准输入发送到fastcgi子进程
4) fastcgi子进程处理请求完成后将标准输出和错误从同一连接返回给web服务器, 当fastcgi子进程结束后请求便结束.fastcgi子进程接着等待处理来自fastcgi进程管理器的下一个连接.

#### php-cgi
php实现的cgi程序

#### php-fpm
php实现的fastcgi程序