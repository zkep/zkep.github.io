---
title: 移动光猫获取超管密码
abbrlink: 56953
date: 2024-02-07 09:39:54
tags: [network]
---


### 获取MAC
命令行方式
```shell
arp -a 
# ? (192.168.1.1) at 7c:fc:fd:02:17:a0 [ether] on eth0
```
通常情况下路由器IP 是192.168.1.1 
找到192.168.1.1 at 7c:fc:fd:02:17:a0 以：分割的就是 MAC 地址
也有的是-分割： 7C-FC-FD-02-17-A0, 都是一样的

也可以直接查看光猫设备背面，MAC条形码

如果有的每段分割显示的字符不够2，则需要在前面补0
例如： 7c:fc:fd:2:17:a0 补零后为 7c:fc:fd:02:17:a0

现在获取的MAC地址，需要处理成大写并且去掉分隔符号
7c:fc:fd:02:17:a0 -> 7CFCFD0217A0

### 开启telnet
telnet 是命令行明文登录设备的方式

在浏览器访问,参数key就是上面获取的MAC地址
```shell
http://192.168.1.1/cgi-bin/telnetenable.cgi?telnetenable=1&key=7CFCFD0217A0

# 访问浏览器后正常可以看到 “telnet开启” 表示成功
# “操作错误”，则表示MAC地址获取的不对
```
### 获取超管密码

命令行使用telnet登录
```shell
telnet 192.168.1.1
```

根据如下输出,交互式输入账号密码 

默认账号： admin

默认密码： Fh@0217A0

密码是Fh@加上获取的MAC地址的后6位，比如说我这里就是 Fh@0217A0

```shell
Trying 192.168.1.1...
Connected to 192.168.1.1.
Escape character is '^]'.
Login:
```

获取超级管理密码， 命令行依次输入如下命令
```shell
load_cli factory
show admin_name
show admin_pwd
```
如下是完整的交互式输入输出：
```shell
root@zkep:/home/zkep# telnet 192.168.1.1
Trying 192.168.1.1...
Connected to 192.168.1.1.
Escape character is '^]'.
Login:admin
Password:
$load_cli factory
Config\factorydir# show admin_name
Success! admin_name=CMCCAdmin
Config\factorydir# show admin_pwd
Success! admin_pwd=aDm8H%MdAD!5Vz2Hh
Config\factorydir#
```
超管账户名：CMCCAdmin

超管密码： aDm8H%MdAD!5Vz2Hh

访问http://192.168.1.1 将上面获取到的 账户名和密码输入，就进入超管的路由器界面了。


### [kuafu](https://github.com/uaxe/kuafu)

[kuafu（夸父）](https://github.com/uaxe/kuafu) 是基于go实现的小工具，可以很方便的可以获取超管密码










