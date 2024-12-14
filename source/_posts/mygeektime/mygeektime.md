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

遇见第一个错误, 提示安装ffmpeg
```text
Please install ffmpeg:
Ffmpeg will be used for video merging

https://ffmpeg.org/download.html

```

安装ffmpeg
```shell
brew install ffmpeg  
```

继续执行
```shell
./mygeektime server
```

遇见第二个错误，需要使用 chromedriver 模拟登录
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

自定义配置文件启动服务
```shell
# 生成配置模版，后可以自定义模版内容
mygeektime cli config --config=config_templete.yml

# 使用自定义配置模版
mygeektime server --config=config_templete.yml
```

默认配置内容
```yaml
server:
  app_name: My Geek Time
  run_mode: debug
  http_addr: 0.0.0.0
  http_port: 8090
jwt:
  secret: mygeektime-secret
  expires: 7200
database:
# driver: mysql
# source: root:123456@tcp(127.0.0.1:3306)/mygeektime?charset=utf8&parseTime=True&loc=Local&timeout=1000ms
# driver: postgres
# source: host=127.0.0.1 user=postgres password=123456 dbname=mygeektime port=5432 sslmode=disable TimeZone=Asia/Shanghai
  driver:  sqlite   # mysql|postgres|sqlite
  source:  mygeektime.db
  max_idle_conns: 10
  max_open_conns: 10
storage: # mp4 或 mp3 存储目录
  directory: repo # 自定义下载文件夹，默认执行目录下的repo目录
  driver: local
  bucket: object
  host: http://127.0.0.1:8090 # 端口与server中的 http_port 保持一致
browser:  # 
  driver_path: chromedriver # 如果没有cookie文件，默认使用chromedriver模拟登录获取cookie
  cookie_path: cookie.txt # geektime的cookie文件存放位置
  open_browser: true # 服务启动后自动打开浏览器
geektime:
  auto_sync: true # 建议开启，默认将geektime的接口数据缓存
```
