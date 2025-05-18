---
title: GRE Over IPSec 短记
date: 2024-07-07T01:02:24+08:00
lastmod: 2024-07-07T01:02:24+08:00
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

>要求FW1 与 RT2 之间用 Internet 互联地址建立 GRE Over IPSec VPN,
实现 Loopback4 之间的加密访问。RT2 的 ACL 名称为 ACL-VPN,
transform-set 名称为 SET-1,crypto map 名称为 MAP-1。FW1 的
isakmp proposal 名称为 P-1,isakmp peer 名称为 PEER-1,ipsec
proposal 名称为 P-2,tunnel ipsec 名称为 IPSEC-1,tunnel gre 名
称为 GRE-1。

网络拓扑如下

| Internet交换机 | 200.200.200.1；200.200.200.5         |
| ----------- | ----------------------------------- |
| RT2         | 连接Internet交换机   IP 地址为200.200.200.2 |
| FW2         | 连接Internet交换机   自身IP为200.200.200.6  |

测试RT2是否能通过Internet Ping通 FW1
`Ping 200.200.200.6 -i 200.200.200.2`

RT2配置
```
interface Tunnel4
ip address 10.4.255.50 255.255.255.252
tunnel source 200.200.200.6
tunnel destination 200.200.200.2
tunnel speed-up

---

crypto isakmp key 0 Key-1122 address 200.200.200.2 255.255.255.255
crypto isakmp policy 1
authentication pre-share
encryption 3des
group 2
hash md5
!
crypto ipsec transform-set SET-1 esp-3des esp-md5-hmac
!
crypto map MAP-1 1 ipsec-isakmp
match address ACL-VPN
set peer 200.200.200.2
set transform-set SET-1

---

ip access-list extended ACL-VPN
permit gre 200.200.200.6 255.255.255.255 200.200.200.2 255.255.255.255

```

不要忘记了设置静态路由

```
ip route default 200.200.200.5
ip route 10.4.7.4 255.255.255.255 Tunnel4

###还有 G0/2口要设置crypto map
crypto map MAP-1
```


FW2配置

```
tunnel gre "GRE-1"
source 200.200.200.2
destination 200.200.200.6
interface ethernet0/3
next-tunnel ipsec IPSEC-1

---

interface tunnel4
zone "VPNHub"
ip address 10.4.255.49 255.255.255.252
manage ping
tunnel gre "GRE-1"

```

Web端部分:
策略设置 Any - Any

设置IPSEC VPN

![Pt1.png](https://s2.loli.net/2024/07/08/kvQBbA42cIwtryO.png)

配置对端  共享秘钥设置与RT2一致
![pt2.png](https://s2.loli.net/2024/07/08/E3ghSr6YKuJAvOV.png)

配置P1

![p1.png](https://s2.loli.net/2024/07/08/XOtwTY17DZ5k2Mq.png)

配置P2

![p2.png](https://s2.loli.net/2024/07/08/xUQE4ruT2m5BWOo.png)

>全部配置完后在高级设置中选择 自动连接 点击开启
>查看IPSECVPN监控,状态为Active
>测试路由和防火墙Loopback4是否正常互相访问

![bg.png](https://s2.loli.net/2024/07/08/oJkL4WugHIVTfQA.png)