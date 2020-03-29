---
title: hexo常用命令记录
tags: hexo
abbrlink: 49eba02e
date: 2020-03-29 23:29:05
---



# hexo常用指令以及tips
> 一段时间没有更新博客就全忘了，差点连这篇也写不出来，赶紧记一下，估计也只有我看得懂hhh

## 写作
1. 创建文章
   `hexo new [layout] <title>`
   其中`layout`主要用到的有`draft`以及`post`，默认为`post`，如果有自定义布局另说
2. 发布以及更新文章
   - `hexo publish [layout] <title>` 主要用于将草稿(`draft`)发布
   - 在通过git进行了commit操作之后，依次运行`hexo clean`，`hexo generate`以及`hexo deploy`即可(大概)