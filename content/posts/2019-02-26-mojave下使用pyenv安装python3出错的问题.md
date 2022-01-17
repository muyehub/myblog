---
layout:     post
title:      "mojave下使用pyenv安装python3出错的问题"
date:       2019-02-26 15:33:39 +0800
author:     "max"
---

## 使用pyenv安装python3时出现如下报错
![](https://ws1.sinaimg.cn/large/60f1733aly1g0jo757rfnj21go0q8wnf.jpg)
按照提示来看,好像是在提示我没有安装zlib库,但是我已经装了,没办法,google一下吧.
(如果你因为没有安装zlib库而产生这个问题, 那就好办了, 直接brew install zlib)
## google搜索到的解决方案大都是要安装xcode命令行工具
```shell
xcode-select --install
```
这个我也安装了, 不安装的话啥也干不了啊
## 最后解决方案
查看一下xcode-select -v的版本
![](https://ws1.sinaimg.cn/large/60f1733aly1g0jo9e0g3lj20cg0240tb.jpg)

这个版本的xcode-select 在默认情况下不包含Mojave SDK的头文件的,需要手动安装,mojave采用了新的SDK,关于新SDK的解释,官方的文档在这里
https://developer.apple.com/macos/whats-new/

接下来,我手动安装了新的SDK头文件,解决完毕
```shell
sudo installer -pkg /Library/Developer/CommandLineTools/Packages/macOS_SDK_headers_for_macOS_10.14.pkg -target /
```
![](https://ws1.sinaimg.cn/large/60f1733aly1g0jomwysz8j21fm040n0e.jpg)

