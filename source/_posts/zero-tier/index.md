---
title: ZeroTier组建公网可访问局域网络
abbrlink: 54294
date: 2024-01-23 22:10:30
tags:
---


### ZeroTier简介
[ZeroTier](https://www.zerotier.com/) 是随时随地安全地连接任何设备，可以构建几乎任何类型的现代、安全的多点虚拟化网络。从强大的点对点网络到多云网状基础设施，我们通过本地网络的简单性实现全球连接。
    
具体介绍，参考官方文档 [ZeroTier文档](https://docs.zerotier.com/)

### 应用场景
   家里有基于树莓派基于openwrt实现的旁路由，树莓派还连接了usb的硬盘盒来实现资源存储,没有公网IP的情况下随时随地能够被访问到， 以往的做法是整一台vps或ecs提供内网穿透服务，比如使用大名鼎鼎的 [frp](https://github.com/fatedier/frp)。 但是有了ZeroTier后，只需要注册一个账号，就可以访问到的虚拟网络中的局域网络

### 开始使用

**Step 1**:  注册[ZeroTier](https://www.zerotier.com/)账号
![](/images/zero-tier/login.png)

**Step 2**:  [创建网络](https://docs.zerotier.com/start)
![](/images/zero-tier/create_network.png)

![](/images/zero-tier/1.png)

![](/images/zero-tier/3.png)

**Step 3**: [设备安装客户端](https://www.zerotier.com/download)

####  MacOS
```shell
brew install zerotier-one
```
加入新的网络，复制注册时候的 NETWORK ID
![](/images/zero-tier/mac-join.png)

#### iOS
AppStore 搜索 Zerotier，下载。点击右上角加号，输入网络ID，OK。

#### Linux
根据官网给的说明，直接运行脚本即可。
```shell
curl -s https://install.zerotier.com | sudo bash
```
如果你安装了 GPG 密钥验证，用下面这条命令
```shell
curl -s 'https://raw.githubusercontent.com/zerotier/ZeroTierOne/master/doc/contact%40zerotier.com.gpg' | gpg --import && \
if z=$(curl -s 'https://install.zerotier.com/' | gpg); then echo "$z" | sudo bash; fi
```
脚本会自动将 Zerotier 的源添加到 apt/yum 里并安装。

安装好后，运行命令
```shell
sudo zerotier-cli join 你的网络ID
```

#### OpenWRT
软路由上，有一个 Zerotier 工具，启用，输入网络 ID，保存并启用即可。
![](/images/zero-tier/openwrt-join.png)

#### Docker
首先启用虚拟网卡

使用docker compose：
```yaml
version: '3.3'
services:
zerotier:
devices:
- /dev/net/tun
privileged: true
image: zerotier/zerotier
network_mode: host
cap_add:
- NET_ADMIN
- SYS_ADMIN
command: ['你的网络ID']
volumes:
- ./data:/var/lib/zerotier-one
```

直接启动即可。

#### 授权
新设备加入后，需要在网页上授权

![](/images/zero-tier/2.png)

#### 命令行的使用
```shell
zerotier-cli peers
```
![](/images/zero-tier/peers.png)
Path 指的是连接时连接到的 IP 地址
link 代表了连接状况。DIRECT 代表可以直接连接，RELAY 代表需要转发。
转发的速度当然会慢一些，因此尽量保证你的 NET 类型为 A/B。不过只要你的路由器支持 uPnP，一般都是直连。

```shell
zerotier-cli listnetworks
```
![](/images/zero-tier/listnetworks.png)

信息依次是：网络 ID、名称、本机的虚拟 IP 地址，和网络状态

网络状态 OK 表示正常，REQUEST_CONFIGURATION 表示没有uPNP连不上，ACCESS_DENIED 表示还没有授权
