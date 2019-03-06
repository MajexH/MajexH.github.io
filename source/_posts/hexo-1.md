---
title: hexo静态博客搭建（一）hexo简介&环境
category: tutorial
tags:
  - hexo
toc: true
date: 2019-01-26 15:12:05
---


一直以来都有搭建博客的想法，粗略了解了一下WordPress和hexo，在尝试过WordPress的安装部署后果断选择了hexo。准备从以下几个方面介绍下hexo的搭建过程

- hexo简介 & 环境
- hexo命令 & 主题
- hexo自动部署

## hexo简介

我们首先来看看[hexo官网](https://hexo.io/zh-cn/index.html)对hexo的介绍

> - 超快速度
> - 支持 Markdown
> - 一键部署
> - 丰富的插件

简而言之，hexo就是

- 一个基于nodejs的博客引擎
- 支持ejs等模板引擎
- 通过渲染Markdown文件生成博客所需各类静态文件
- 有较为完整的生态支持，包含各种插件，满足大部分需求

所以对于熟悉nodejs的用户能够通过hexo快速搭建一个博客

## hexo环境

### nodejs环境

#### nvm + nrm

- 可以使用[nvm](https://github.com/creationix/nvm)进行nodejs版本切换和管理
- 国内环境下推荐使用淘宝的npm源[cnpm](https://npm.taobao.org/)或者使用
[nrm](https://github.com/Pana/nrm)切换到淘宝源
- 全局安装hexo-cli
```
npm install hexo-cli -g   
```

#### 使用包管理器

osx环境下使用homebrew安装

```
brew install nodejs
```

### git环境

windows可以通过访问[git官网](https://git-scm.com/)下载安装包即可

各类linux系等建议通过包管理器下载