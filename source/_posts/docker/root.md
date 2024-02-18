---
title: docker容器内使用root用户安装其他软件
abbrlink: 49176
date: 2024-02-18 13:04:25
tags: [docker]
---

### 使用root进入容器
--user root 指定root用户进入容器
```shell
docker exec -it --user root [容器名称] bash 
```

### 更新
```shell
# yum update
apt-get update 
```

### 安装vim
```shell
# yum install vim
apt-get install vim
```
