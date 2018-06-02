---
title: hexo博客修改时间
date: 2018-06-02 17:20:02
tags: hexo
categories: hexo
---

hexo博客中默认只有发表时间，并没有最后修改的时间，而且显示时间是年月日，并没有具体到时分秒。这并不能满足我的需求，因此我觉得为博客加上最后的修改时间，并将显示的时间具体到时分秒。本文主要记录如何增加显示博客的最后修改时间，以及将默认显示时间由年月日改成年月日时分秒。

*注意：本文基于主题**[NexT](https://github.com/theme-next/hexo-theme-next)**主题 (version: 5.1.4)，对于其他主题可能并不适合。*

<!-- more -->

# 增加最后修改时间

在`${hexoHome}/themes/hexo-theme-next/_config.yml`文件中找到`post_meta`参数，将其底下的`updated_at`的值改为`true`，即可显示文章的最后修改时间。

```yaml
# Post meta display settings
post_meta:
  item_text: true
  created_at: true
  updated_at: true
  categories: true
```

# 修改文章显示时间格式

文章显示时间格式主要是由`${hexoHome}/themes/hexo-theme-next/layout/_macro/post.swig`文件中控制的。其中`config.date_format`主要指`${hexoHome}/_config.xml`中的`date_format`的值。因此有两种方法可以修改文章显示的时间为年月日时分秒。

![控制文章时间显示格式](/images/hexoPostTimeFormatOld.png)

方法一：修改`${hexoHome}/_config.xml`中的`date_format`的值`YYYY-MM-DD HH:mm:ss`。此方法改起来比较简单，不过考虑到变量名和变量值语义不一致，所以我采用了方法二进行修改。

方法二：在`${hexoHome}/_config.xml`文件中增加`datetime_format: YYYY-MM-DD HH:mm:ss`配置项，然后在`${hexoHome}/themes/hexo-theme-next/layout/_macro/post.swig`文件中将控制日期显示格式的`config.date_format`改成`config.datetime_format`，如下图所示

![文章时间格式显示为年月日时分秒](/images/hexoPostTimeFormatNew.png)