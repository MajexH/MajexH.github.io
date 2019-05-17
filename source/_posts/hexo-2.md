---
title: hexo静态博客搭建（二）hexo命令&主题
category: tutorial
tags:
  - hexo
toc: true
date: 2019-03-06 16:36:04
thumbnail:
---


## [hexo命令](https://hexo.io/zh-cn/docs/commands)

### hexo 安装

nodejs环境安装完成后，在全局下安装hexo-cli，命令如下

```
npm install -g hexo-cli
```

hexo-cli安装完毕后才可以执行hexo命令

### hexo init

```
hexo init [folder]
```
folder是当前博客目录，通过hexo init命令可以在指定的folder目录下自动克隆一个hexo项目模板（一个nodejs项目），并自动安装相应的依赖，其目录文件结构如下.

<pre>
├── _config.yml          - hexo的相关配置信息
├── db.json
├── node_modules         - nodejs依赖
├── package.json         - nodejs项目信息
├── scaffolds            - 发布模板
├── source               - 存放用户资源，如文章等
└── themes               - 主题，会根据主题生成 、对应的页面
</pre>

### hexo server

通过`hexo init`命令建立博客模板文件后，其自带了`landscape`主题，可以通过hexo server直接启动

### hexo new

```
hexo new [layout] <title>
```
新建一个名为`title`的文章。如果没有设置 layout 的话，默认使用 _config.yml 中的 default_layout 参数代替

### hexo publish

```
hexo publish [layout] <filename>
```
hexo将文章组织为draft和post两种状态，因此在使用了`hexo new draft someFile`后需要通过`hexo publish`命令将处于draft下的文章正式发表，这样才能在blog中正常查看。

### hexo generate & hexo deploy

hexo generate会将markdown文件等一系列的文件编译转换成标准的静态文件。
hexo deploy命令会将本地生成的静态文件一键部署到远程服务器。

## hexo 主题

hexo提供了丰富的主题，这个blog使用了[icarus](https://github.com/ppoffice/hexo-theme-icarus)

### 安装
1. 如果blog通过git来进行版本控制，可以直接通过git安装
```
git submodule add https://github.com/ppoffice/hexo-theme-icarus.git themes/icarus
```
2. 可以去其github页面上下载release包，解压到`themes/icarus`下
  
### 微调样式
大部分对于样式的微调都能在[issues](https://github.com/ppoffice/hexo-theme-icarus/issues)页面找到
1. [阅读页面三栏转二栏](https://github.com/ppoffice/hexo-theme-icarus/issues/379)
2. 添加about页面<br/>
icarus将about页面挂载在了网页的/about页面
```
cd blog根目录/source
mkdir about
touch index.md
```
3. 中文设置
- icarus自带中文设置
```
vim blog根目录/_config.yml
```
将language项调整成zh-CN，hexo会自动加载themes/icarus下的i18n文件
- 导航栏中文设置<br/>
修改`/themes/icarus`下的_config.yml将

### 添加[comment](https://blog.zhangruipeng.me/hexo-theme-icarus/categories/Plugins/Comment/)插件
icarus提供了对于gitalk、Valine等评论插件的支持，这个blog使用了valine对评论插件

#### valine

##### 申请创建应用

申请[leancloud](http://www.leancloud.cn)

{% asset_img application.png test %}

应用申请成功后将下列key填入icarus下的_config.yml中

{% asset_img key.png test %}

##### 添加valine人数统计

在添加valine的comment组件的时候，发现valine支持文章的浏览数量统计，考虑添加valine阅读数量统计。<br/>
找到`layout/comment/valine.ejs`，添以下到`else` block中
```html
<span id="<Your/Path/Name>" class="leancloud-visitors" data-flag-title="Your Article Title">
  <em class="post-meta-item-text">阅读量 </em>
  <i class="leancloud-visitors-count">1000000</i>
</span>
```

```js
(function () {
  var span = document.getElementsByClassName('leancloud-visitors')[0];
  span['id'] = window.location.pathname;
  // 貌似 title 有bug
  // span.dataset.flagTitle = document.title;
})();
```

`span的id`会作为valine统计的主键，所以直接填入每个location对象的pathname即可，即可做出阅读量统计