---
title: 横向移动 通过SSH连接使目标机及作为Socks5代理服务器
date: 2019-10-15T20:05:30+08:00
lastmod: 2019-10-15T20:05:30+08:00
author:
  - 未淼
tags: 
description: ""
weight: 
slug: ""
draft: false
comments: false
showToc: true
TocOpen: true
hidemeta: false
disableShare: true
showbreadcrumbs: true
---



## 直接说方法


```
ssh -D <本地端口号> -f -C -q -N <用户名>@<SSH服务器地址>
```


- `-D <本地端口号>`：指定一个端口号，会在本地创建一个Socks5端口用于转发，所有通过这个端口号的连接都将被转发到SSH建立连接后的服务器，并由服务器发送到目标地址
    
- `-C`：启用压缩，通过使用压缩，减少数据传输量，提高传输速度
    
- `-q`：使用安静模式，禁止SSH客户端在连接过程中输出任何警告或错误消息，只输出关键信息
    
- `-N`：告诉SSH客户端不要执行任何远程命令，在此情况下，只会建立SSH连接并启用端口转发

> 很多情况下，我们想要借助一台处于内网又能出网的业务服务器来访问到内网部分，又不能直接在服务器上安装服务，可以用服务器自带ssh服务建立一个连接，打通你和内网的互通，很方便，记录一下
