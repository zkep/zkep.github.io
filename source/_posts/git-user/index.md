---
title: git多用户配置
abbrlink: 55384
date: 2024-02-05 17:55:04
tags:
---

### 场景
github的用户名与gitlab用户名不一样，全局设置git user.name 导致提交到所有仓库的用户名是一个
现在需要区分开，各自维护自己的git用户名


### 清除全局配置

查看全局配置信息
```shell
git config --global --lists
```
全局配置中有 user.name 和 user.email 信息，请执行以下命令将其清除掉：

```shell
git config --global --unset user.name
git config --global --unset user.email
```
### 生成钥对
生成对应的用户密钥
```shell
ssh-keygen -t rsa -C "xxxx@gmail.com"
```
按下 ENTER 键后，会有如下提示：
```shell
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
```
输入公钥的名字，默认情况是叫 id_rsa，为了和后面的 gitlab 配置区分，这里输入 id_rsa_github, 输入完毕后，一路回车，钥对就生成完毕了


继续开始生成 gitlab 上的仓库钥对，步骤和上面一样：
```shell
ssh-keygen -t rsa -C "xxxx@test.com"
```
生成的公钥名就叫做：id_rsa_gitlab


### 将公钥添加到 SSH Keys 中
将id_rsa_github.pub 和 id_rsa_gitlab.pub 内容分别添加到 github 和 gitlab 的 SSH Keys 中

### 添加私钥
执行如下命令将私钥添加到本地
```shell
ssh-add ~/.ssh/id_rsa_github // 将 GitHub 私钥添加到本地
ssh-add ~/.ssh/id_rsa_gitlab // 将 GitLab 私钥添加到本地
```
### 验证私钥
执行 ssh-add -l 验证下，如果都能显示出来和下面一样，就 OK 了
```shell
2048 SHA256:mXVNxWHZsZpKOnHlPslF2jXAWR+jc7M6P5hYbrCo xxx@gmail.com (RSA)
2048 SHA256:Blhp3+Hx5mp9HDivFjDuwc/PaQ8ux45TRa6nTsfIe0PEz4 xxx@test.com (RSA)
```

### 管理密钥
在本地创建一个密钥配置文件，通过该文件，实现根据仓库的 remote 链接地址自动选择合适的私钥
编辑 ~/.ssh 目录下的 config 文件，如果没有，请创建
```shell
vim ~/.ssh/config
```
配置如下：
```shell
Host github
HostName github.com
User xxx
IdentityFile ~/.ssh/id_rsa_github

Host gitlab
HostName gitlab.xxx.com
User xxx
IdentityFile ~/.ssh/id_rsa_gitlab

```
该文件分为多个用户配置，每个用户配置包含以下几个配置项：

Host：仓库网站的别名，随意取
HostName：仓库网站的域名（IP 地址应该也可以）
User：仓库网站上的用户名
IdentityFile：私钥的绝对路径

ssh -T 命令检测 Host连通性
```shell
ssh -T git@github
```
输出如下
```shell
Hi xxx! You've successfully authenticated, but GitHub does not provide shell access.
```


### 仓库配置

git 的配置分为三级别，System —> Global —>Local。System 
即系统级别，Global 为配置的全局，
Local 为仓库级别，优先级是 Local > Global > System
为每个仓库单独配置用户名信息，在仓库里执行
```shell
git config --local user.name "xxxx"
git config --local user.email "xxx@gmail.com"
```
看本仓库的所有配置信息
```shell
git config --local --list
```