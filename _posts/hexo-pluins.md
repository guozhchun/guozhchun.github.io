---
title: 几个好用的hexo插件
date: 2018-05-31 22:09:44
tags: [hexo, 插件]
categories: hexo
---

hexo官方提供了很多**[插件](https://hexo.io/plugins/)**方便编写管理文章，以下几个插件是我觉得相对比较好用的插件。

*注意：本文均以**[NexT](https://github.com/theme-next/hexo-theme-next)**主题 (version: 5.1.4)为基础进行的配置，不同的主题配置不同*

<!-- more -->

# Gitment

这个插件主要是用于增加文章的评论的。主要是借助github的issues，文章评论的内容会当成某个仓库issues放在github上。NexT 5.1.4 版本已经集成了这个插件，只需要注册一个OAuth Application，改下主题的相关配置项即可。主要步骤如下

## 注册OAuth Application

**[点击此处](https://github.com/settings/applications/new)**申请一个新的OAuth Application。如下图所示，需要注意的是Authorization callback URL需要填写自己的hexo网站域名。注册成功后会得到一个`Client ID `和`Client Secret `，这两个值是后面配置`${hexoHome}/themes/hexo-theme-next/_config.yml`中需要用到的。


![register OAuth Application](/images/registerOAuthApplication.png)

## 修改主题的_config.yml配置

主要是将gitment底下的`enable`参数改为`true`，`github_user`写自己的github用户名，`github_repo`是用于存放评论内容的issues对应的仓库，必须存在，可以新建一个新的空仓库，也可以放在已有的仓库底下。`client_id`和`client_secret`是上一部中申请OAuth Application生成的`Client ID `和`Client Secret `。

![gitment config](/images/gitmentConfig.png)

## 初始化评论

配置完插件后，需要对开放评论的界面手动初始化评论才能让其他用户进行评论

![初始化文章评论](/images/initComments.png)

## 关闭页面评论

对于一些不想开放评论功能的界面，可以在对应的页面文件中的开头加入`comments: false`语句，这样评论区域就不会显示在该页面上了。例如关闭tags页面的评论

![close tag page comments](/images/closeTagComments.png)

# 不蒜子

这个插件主要是用来统计文章和网站的访问量的，具体使用可以参考**[官网](http://ibruce.info/2015/04/04/busuanzi)**。NexT 5.1.4 版本已经集成了这个插件，只需要将`${hexoHome}/themes/hexo-theme-next/_config.yml`配置文件中`busuanzi_count`底下的`enable`参数改成`true`即可。其他参数可以参考配置文件中的说明进行修改。

![busuanzi config](/images/busuanzi.png)

# hexo-filter-flowchart

使用这个插件可以很方便的用markdown画出流程图，由于我使用的markdown编辑器是**[Typora](https://www.typora.io/)**，内部集成了flowcharts.js文件，可以使用其画出流程图，对于需要将这些通过Typora画出的流程图上传到网站上并能正常显示，就需要使用**[hexo-filter-flowchart](https://github.com/bubkoo/hexo-filter-flowchart)**这个插件了。主要配置如下

## 安装flowchart插件

在`${hexoHome}`目录下执行以下命令完成安装

```
npm install --save hexo-filter-flowchart
```

## 修改 _config.yml配置项

在` ${hexoHome}/_config.xml` 文件中，增加以下的配置项

```
flowchart:
  # raphael:   # optional, the source url of raphael.js
  # flowchart: # optional, the source url of flowchart.js
  options: # options used for `drawSVG`
```

# local_search

**[local_search](https://github.com/theme-next/hexo-generator-searchdb)**这个插件主要是为博客增加搜索功能。主要安装配置如下

## 安装hexo-generator-searchdb

在`${hexoHome}`目录下执行以下命令安装hexo-generator-searchdb插件

```
npm install hexo-generator-searchdb --save
```

## 修改主题的 _config.yml配置

主要是将`local_search`底下的`enable`值改为`true`

![local search](/images/localSearch.png)

# hexo-featured-image

这个插件主要是使用markdown语法写的图片能够上传到网站上并正常显示的一个插件。只需要安装该插件，然后在`${hexoHome}/source`目录底下新建一个`images`目录用于存放图片即可，在引用图片时图片的路径为`/images/yourImageName`，因为使用`hexo g`时会将`${hexoHome}/source/images`底下的图片拷贝到`${hexoHome}/public/images目录下`。例如：引用图片`${hexoHome}/source/images/localSearch.png`需要写`![](/images/localSearch.png)`。这里有点不太好的地方就是引用的图片在markdown上无法显示，因为本地路径不对（但是发布之后路径是对的，因此在线时可以正常显示）。具体配置可以参考**[官网](https://github.com/poacher2k/hexo-featured-image)**，这里以其最简单的配置进行说明（不修改 _config.yml文件）

## 安装插件

在`${hexoHome}`目录下执行以下命令完成安装

```
npm install --save hexo-featured-image
```

 ## 新建images目录

在`${hexoHome}/source`目录下新建`images`目录

![create image directory](/images/imagesDir.png)

## 引用图片

在需要引用图片的地方使用markdown的格式进行引用，需要注意的是引用图片的路径必须为`/images/yourImageName`。例如引用`localSearch.png`文件应该写`![local search](/images/localSearch.png)`。

![use image](/images/useImage.png)

# 参考来源

1. [https://imsun.net/posts/gitment-introduction](https://imsun.net/posts/gitment-introduction/)
2. [http://ibruce.info/2015/04/04/busuanzi](http://ibruce.info/2015/04/04/busuanzi)
3. [https://github.com/bubkoo/hexo-filter-flowchart](https://github.com/bubkoo/hexo-filter-flowchart)
4. [https://github.com/theme-next/hexo-generator-searchdb](https://github.com/theme-next/hexo-generator-searchdb)
5. [https://github.com/poacher2k/hexo-featured-image](https://github.com/poacher2k/hexo-featured-image)