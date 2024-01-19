---
title: hexo搭建github博客
abbrlink: hexo
date: 2018-03-02 16:51:15
tags: [hexo]
categories: [hexo]
sticky: 1
---

### 使用github pages服务搭建博客

    全是静态文件，访问速度快；
    免费方便，不用花一分钱就可以搭建一个自由的个人博客，不需要服务器不需要后台；
    可以随意绑定自己的域名，不仔细看的话根本看不出来你的网站是基于github的；
    数据绝对安全，基于github的版本管理，想恢复到哪个历史版本都行；
    博客内容可以轻松打包、转移、发布到其它平台；
    等等；

<!--more-->

#### 准备工作

在开始一切之前，你必须已经：
有一个[github](https://github.com/)账号，没有的话去注册一个；
安装了node.js、npm，并了解相关基础知识；

#### 搭建github博客

##### 创建仓库

新建一个名为你的用户名.github.io的仓库，比如说，如果你的github用户名是test，那么你就新建test.github.io的仓库（必须是你的用户名，其它名称无效），将来你的网站访问地址就是 <http://test.github.io> 了，是不是很方便？
由此可见，每一个github账户最多只能创建一个这样可以直接使用域名访问的仓库。
几个注意的地方：
注册的邮箱一定要验证，否则不会成功；
仓库名字必须是：username.github.io，其中username是你的用户名；
仓库创建成功不会立即生效，需要过一段时间，大概10-30分钟，或者更久，我的等了半个小时才生效；
创建成功后，默认会在你这个仓库里生成一些示例页面，以后你的网站所有代码都是放在这个仓库里啦。

##### 绑定域名

当然，你不绑定域名肯定也是可以的，就用默认的 xxx.github.io 来访问，如果你想更个性一点，想拥有一个属于自己的域名，那也是OK的。

首先你要注册一个域名，域名注册以前总是推荐去godaddy，现在觉得其实国内的阿里云也挺不错的，价格也不贵，毕竟是大公司，放心！

绑定域名分2种情况：带www和不带www的。

域名配置最常见有2种方式，CNAME和A记录，CNAME填写域名，A记录填写IP，由于不带www方式只能采用A记录，所以必须先ping一下你的用户名.github.io的IP，然后到你的域名DNS设置页，将A记录指向你ping出来的IP，将CNAME指向你的用户名.github.io，这样可以保证无论是否添加www都可以访问

##### 配置SSH key

为什么要配置这个呢？因为你提交代码肯定要拥有你的github权限才可以，但是直接使用用户名和密码太不安全了，所以我们使用ssh key来解决本地和服务器的连接问题。

用git bash执行如下命令：

```bash
cd ~/. ssh #检查本机已存在的ssh密钥
```

如果提示：No such file or directory 说明你是第一次使用git。

```bash
ssh-keygen -t rsa -C "邮件地址"
```

然后连续3次回车，最终会生成一个文件在用户目录下，打开用户目录，找到.ssh\id_rsa.pub文件，记事本打开并复制里面的内容，打开你的github主页，进入个人设置 -> SSH and GPG keys -> New SSH key：

将刚复制的内容粘贴到key那里，title随便填，保存。

##### 测试是否成功

```bash
ssh -T git@github.com # 注意邮箱地址不用改
```

如果提示Are you sure you want to continue connecting (yes/no)?，输入yes，然后会看到：

```bash
Hi xxx! You’ve successfully authenticated, but GitHub does not provide shell access.
```

看到这个信息说明SSH已配置成功！
此时你还需要配置：

```bash
git config --global user.name "zkep"// 你的github用户名，非昵称
git config --global user.email  "xxx@qq.com"// 填写你的github注册邮箱
```

#### 使用hexo写博客

##### hexo简介

[Hexo](https://hexo.io/zh-cn/)是一个简单、快速、强大的基于 Github Pages 的博客发布工具，支持Markdown格式，有众多优秀插件和主题。

##### 原理

由于github pages存放的都是静态文件，博客存放的不只是文章内容，还有文章列表、分类、标签、翻页等动态内容，假如每次写完一篇文章都要手动更新博文目录和相关链接信息，相信谁都会疯掉，所以hexo所做的就是将这些md文件都放在本地，每次写完文章后调用写好的命令来批量完成相关页面的生成，然后再将有改动的页面提交到github。

##### 注意事项

安装之前先来说几个注意事项：

很多命令既可以用Windows的cmd来完成，也可以使用git bash来完成，但是部分命令会有一些问题，为避免不必要的问题，建议全部使用git bash来执行；
hexo不同版本差别比较大，网上很多文章的配置信息都是基于2.x的，所以注意不要被误导；
hexo有2种_config.yml文件，一个是根目录下的全局的_config.yml，一个是各个theme下的；

##### 安装

```bash
npm install -g hexo  # 全局安装
cd /home/test.github.io   # 在指定目录下新建项目文件夹
hexo init
hexo g # 生成
hexo s # 启动服务
```

hexo s是开启本地预览服务，打开浏览器访问 <http://localhost:4000> 即可看到内容，很多人会碰到浏览器一直在转圈但是就是加载不出来的问题，一般情况下是因为端口占用的缘故

##### 修改主题

既然默认主题很丑，那我们别的不做，首先来替换一个好看点的主题。这是 [官方主题](https://hexo.io/themes/)。

个人比较喜欢的2个主题：[hexo-theme-jekyll](https://github.com/pinggod/hexo-theme-jekyll) 和 [hexo-theme-yilia](https://github.com/litten/hexo-theme-yilia)。

首先下载这个主题,放到项目的themes目录

修改_config.yml中的theme: landscape改为theme: yilia，然后重新执行hexo g来重新生成。

如果出现一些莫名其妙的问题，可以先执行hexo clean来清理一下public的内容，然后再来重新生成和发布。

##### 上传之前

在上传代码到github之前，一定要记得先把你以前所有代码下载下来（虽然github有版本管理，但备份一下总是好的），因为从hexo提交代码时会把你以前的所有代码都删掉。

##### 上传到github

如果你一切都配置好了，发布上传很容易，一句hexo d就搞定，当然关键还是你要把所有东西配置好。

首先，ssh key肯定要配置好。

其次，配置_config.yml中有关deploy的部分：

正确写法：

```bash
deploy:
  type: git
  repository: git@github.com:xxx/xxx.github.io.git
  branch: master
```

此时直接执行hexo d的话一般会报如下错误：

```bash
Deployer not found: github 或者 Deployer not found: git
```

原因是还需要安装一个插件：

```bash
npm install hexo-deployer-git --save
```

##### 常用hexo命令

常见命令

```bash
hexo new "postName" #新建文章
hexo new page "pageName" #新建页面
hexo generate #生成静态页面至public目录
hexo server #开启预览访问端口（默认端口4000，'ctrl + c'关闭server）
hexo deploy #部署到GitHub
hexo help  # 查看帮助
hexo version  #查看Hexo的版本
```

缩写：

```bash
hexo n == hexo new
hexo g == hexo generate
hexo s == hexo server
hexo d == hexo deploy
```

组合命令：

```bash
hexo s -g #生成并本地预览
hexo d -g #生成并上传
```

##### _config.yml

这里面都是一些全局配置，每个参数的意思都比较简单明了，所以就不作详细介绍了。

需要特别注意的地方是，冒号后面必须有一个空格，否则可能会出问题。

##### 写博客

定位到我们的hexo根目录，执行命令：

```bash
hexo new 'my-first-blog'
```

hexo会帮我们在_posts下生成相关md文件：

我们只需要打开这个文件就可以开始写博客了，默认生成如下内容：
一般完整格式如下：

```json
---
title: postName #文章页面上的显示名称，一般是中文
date: 2013-12-02 15:30:16 #文章生成时间，一般不改，当然也可以任意修改
categories: 默认分类 #分类
tags: [tag1,tag2,tag3] #文章标签，可空，多标签请用格式，注意:后面有个空格
description: 附加一段文章摘要，字数最好在140字以内，会出现在meta的description里面
---

以下是正文
```

那么hexo new page 'postName'命令和hexo new 'postName'有什么区别呢？
最终部署时生成：hexo\public\my-second-blog\index.html，但是它不会作为文章出现在博文目录。

##### 如何让博文列表不显示全部内容

默认情况下，生成的博文目录会显示全部的文章内容，如何设置文章摘要的长度呢？

答案是在合适的位置加上`<!--more-->`即可，例如：

```json
# 前言

使用github pages服务搭建博客的好处有：

1. 全是静态文件，访问速度快；
2. 免费方便，不用花一分钱就可以搭建一个自由的个人博客，不需要服务器不需要后台；
3. 可以随意绑定自己的域名，不仔细看的话根本看不出来你的网站是基于github的；

<!--more-->

4. 数据绝对安全，基于github的版本管理，想恢复到哪个历史版本都行；
5. 博客内容可以轻松打包、转移、发布到其它平台；
6. 等等；

```

### Hexo安装[mermaid diagrams](https://mermaid-js.github.io/mermaid/#/)

#### 安装插件

```shell
npm install hexo-filter-mermaid-diagrams
```

#### 修改配置文件

在hexo的_config.yml文件（根目录的并非主题的）中，添加以下内容：

```yaml
# mermaid chart
mermaid: ## mermaid url https://github.com/knsv/mermaid
  enable: true  # default true
  version: "7.1.2" # default v7.1.2
  options:  # find more api options from https://github.com/knsv/mermaid/blob/master/src/mermaidAPI.js
    #startOnload: true  // default true

```

#### js文件修改

1. 修改位置 （fluid主题为例）

> themes/fluid/layout/_partials/footer.ejs

2. 根据footer的格式不同，添加的内容不同。
格式有after_footer.pug , after-footer.ejs ,footer.swig等。
以下是在next的footer.swig添加的内容。其他格式参考[github: hexo-filter-mermaid-diagrams](https://github.com/webappdevelp/hexo-filter-mermaid-diagrams)

```html
<footer class="text-center mt-5 py-3">
  
</footer>
<% if (theme.mermaid.enable) { %>
  <script src='https://unpkg.com/mermaid@<%= theme.mermaid.version %>/dist/mermaid.min.js'></script>
  <script>
    if (window.mermaid) {
      mermaid.initialize({theme: 'forest'});
    }
  </script>
<% } %>
```

### hexo博客链接持久化

1. 安装hexo-abbrlink插件

```shell
 npm install hexo-abbrlink --save
```

2. 配置

在根目录下的配置_config.yml里的配置:permalink

```yaml
permalink: :abbrlink.html
```

生成之后的效果是这样的了<http://url/abbrlink.html>

3. 说明

> 如果文章头中存在abbrlink，则不会做任何处理。
> 如查文章头中不存在abbrlink，则会et title根据配置的alg算法来成生abbrlink字符串。
> 当然，你也可以自己手动为特殊的文章写链接地址。只要在文章中配置好abbrlink就可以了

```yaml
  title: go-zero
  abbrlink: go-zero
  date: 2021-03-28 13:42:22
  tags: [go,go-zero]
```

### 将本地博客hexo静态页面部署github 

> 基本做法是git上新建两个仓库 ，一个私有仓库存放markdown这些源代码（暂时定为blog吧），一个公开访问仓库的username.github.io存放生成的页面

1. 在Github的 blog 仓库上新建一个xxx分支，并切换到该分支，并在该仓库 ->Settings->Branches->Default branch中将默认分支设为xxx，save保存；然后将该仓库克隆到本地，进入该blog文件目录。

2. 将本地博客的部署文件拷贝进blog文件目录
  如题，先将本地博客的部署文件（Hexo目录下的全部文件）全部拷贝进blog文件目录中去。
  接下来，进入blog文件目录下，将该目录下的全部文件提交到xxx分支，提交之前需注意：

* 将themes目录以内中的主题的.git目录删除（如果有），因为一个git仓库中不能包含另一个git仓库，提交主题文件夹会失败。
* 可能有人会问，删除了themes目录中的.git不就不能git pull更新主题了吗，很简单，需要更新主题时在另一个地方git clone下来该主题的最新版本，然后将内容拷到当前主题目录即可
* 提交hexo分支，执行git add .、git commit -m 'back up hexo files'（引号内容可改）、git push即可将博客的hexo部署环境提交到GitHub个人仓库的xxx分支。


> 配置_config.yml的deploy
```bash
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: https://github.com/zkep/zkep.github.io.git
  branch: master
  message: 
```

> 这样blog私有仓库git的正常的commit，push都会提交到私有仓库。hexo d 的命令则会把生成的.deploy_git目录里的静态页面提交到username.github.io仓库

### node版本太高导致的hexo d 命令生成了空白页面解决方法
>  hexo-deployer-git 插件对node版本比较挑剔
```bash
  # 查看本地node版本
  node -v
  # 如果版本大于v12.16.2，建议降低版本到v12.16.2试试
  # 开始下n来管理node的版本。
  sudo npm i -g n
  # 替换node版本到v12.16.2
  sudo n v12.16.2
  # 再次在hexo代码目录执行
  hexo clean
  hexo g
  # 再次查看生成的public文件夹中index.html，就正常了
```
