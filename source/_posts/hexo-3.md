---
title: hexo静态博客搭建（三）hexo自动部署
category: tutorial
tags: hexo
toc: true
date: 2019-05-17 18:25:44
thumbnail:
---


经过前两步，一个能看的blog已经搭起来了，在本地调试完成后，还是需要一个地方部署这个blog。因为有一个服务器来科学上网，所以就吧这个blog部署到了同一个vps上（这两个应用都不是太吃资源）。所以接下来介绍如何将blog部署到一个远程的服务器上。

## 自动部署到VPS

### 需要用到的hexo命令

hexo是一个部署静态blog的工具，因此所有的文件最后都会被生成浏览器能认识的html、css、js文件。

```
hexo clean
hexo deploy
```

因此需要在部署前使用`hexo clean`命令，清理已经生成的`/public`下的静态文件和db.json，再调用`hexo deploy`命令使用hexo提供的部署命令将本地的静态文件部署到远程服务器。

### hexo deploy use rsync

hexo deploy提供了多种[方式](https://hexo.io/zh-cn/docs/deployment)来部署到远程，这里使用了rsync来部署。

#### 需要在服务器端做什么

{% blockquote hexo对于rsync的介绍 %}
需要注意的是，要求您提供的实际上是一个能`通过SSH登陆`远程主机的Linux用户。Hexo会自动处理关于rsync使用的一切操作。因此，您需要在远程主机上为您的Hexo站点建立一个用户，并允许其通过SSH登陆。不过，这里的port，的确是指rsync监听的端口，请确保防火墙打开了该端口。
{% endblockquote %}

也就是说rsync需要我们提供一个可以通过ssh登录的用户，这样hexo可以使用该用户通过ssh将需要部署的文件移动到远程服务器上

#### 本地设置

在搞定了用户之后我们就需要在blog的配置文件`_config.yml`中配置`deploy`选项来启用rsync帮助我们来进行远程部署，如下图所示。
{% asset_img rsync.png test %}

### 使用git hooks来自动调用hexo deploy

在完成了deploy选项的设置后，可以通过`hexo deploy`命令一键将静态文件远程部署到服务器上，但是每次都需要手动调用下这个命令，同时因为blog放到了github上，所以为了偷个小懒就利用git hooks，在本地commit后，调用hexo deploy命令将本地的文件部署到远程。

首先进入到hooks文件中
```
cd .git/hooks
```

添加以下内容到`post-commit`文件中，这样每次在本地commit后，都会出发这个hook，调用这个文件中的命令

```
#!/bin/bash
cd /path/to/your/blog
echo "----- 开始 -----"
hexo clean
hexo generate
hexo deploy
echo "----- 结束 -----"
```

`warning:首先需要进入到你的blog的根目录下，这样hexo的命令才会被正确的调用`