---
title: 极客时间资源下载工具
tags:
  - go
  - amis
abbrlink: 30180
date: 2024-12-14 10:54:45
---

### 极客时间资源下载工具

#### 安装

根据自己的系统下载

https://github.com/zkep/mygeektime/releases

MacOS为例：

```shell
wget 'https://github.com/zkep/mygeektime/releases/download/v0.0.1/mygeektime_Darwin_arm64.tar.gz'
```

```shell
tar -zxvf mygeektime_Darwin_arm64.tar.gz
```

```shell
./mygeektime server
```

遇见第一个错误，需要使用 chromedriver 模拟登录
```text
Please install chromedriver:
Chromedriver will be used by default to simulate login and obtain cookies
https://googlechromelabs.github.io/chrome-for-testing/#stable

Also you can save Geektime's cookie to 'cookie.txt' in current folder
stat cookie.txt: no such file or directory OR exec: "./chromedriver": stat ./chromedriver: no such file or directory
```
查看 Chrome 版本号, 在谷歌浏览器的地址栏输入：
```shell
chrome://version
```
![](/images/mygeektime/chrome-version.png)

查询本地chrome版本号为：131.0.6778.140
```shell
wget 'https://storage.googleapis.com/chrome-for-testing-public/131.0.6778.140/mac-arm64/chromedriver-mac-arm64.zip'
```

```shell
unzip chromedriver-mac-arm64.zip
```

```shell
mv chromedriver-mac-arm64/chromedriver .
```
继续执行，启动web服务, 弹出模拟登录页面
```shell
./mygeektime server
```
![](/images/mygeektime/login.png)


我的课程列表
![](/images/mygeektime/product.png)


下载课程
![](/images/mygeektime/product-download.png)

下载任务
![](/images/mygeektime/task.png)


视频详情，可导出 mp4 和markdown文本信息
![](/images/mygeektime/task-video.png)


音频详情，可导出 mp3 和markdown文本信息
![](/images/mygeektime/task-audio.png)
