---
title: hexo常用指令以及tips
tags: hexo
abbrlink: 49eba02e
date: 2020-03-29 23:29:05
---



> 一段时间没有更新博客就全忘了，差点连这篇也写不出来，赶紧记一下，估计也只有我看得懂hhh

## 写作
1. 创建文章
   `hexo new [layout] <title>`
   其中`layout`主要用到的有`draft`以及`post`，默认为`post`，如果有自定义布局另说
2. 发布以及更新文章
   - `hexo publish [layout] <title>` 主要用于将草稿(`draft`)发布
   - 依次运行`hexo clean`(可选)，`hexo generate`以及`hexo deploy`即可(大概)
3. 设置多个标签
   在文章的`tags`一栏中,如果想要设置多个标签,使用如下格式
   ```sh
   tags: [标签1, 标签2]
   # 如
   tags: [SQL, Linux]
   ```
4. 插入图片
   无法使用markdown原生插入图片方式(大概)
   在`md`文件中使用`{% asset_img img_name %}`格式来插入图片,其中`img_name`为`md`文件对应的文件夹中的图片名称
   如文件结构如下
   ```
   - Root
     |- Note.md
     |- Note/img_name.png
   ```
   那么在Node.md中引用图片`img_name`即为
   `{% asset_img img_name.png %}`
   通过使用`hexo-asset-image`插件可以直接在md中使用正常的`![]()`写法，非常方便，并且支持`hexo-abbrlink`
## 预览
1. 本地运行
   `hexo server`
   访问`http://localhost:4000`即可

```python
S = aA | bQ
Q = aQ | bD | b
  = a*(bD | b)
D = bB | aA
  = b(bD | aQ) | aA
  = bbD | baQ | aA
  = (bb)*(baQ | aA)
  = (bb)*(ba(a* (bD | b)) | aA)
  = (bb)*(ba(a*bD | a*b) | aA)
  = (bb)*(baa*bD | baa*b | aA)
  = (bb)*baa*bD | (bb)*baa*b | bb*aA
  = ((bb)*baa*b)*((bb)*baa*b | bb*aA)
A = aA | bB | b
  = a*(bB | b)
  = a*(b(bD | aQ) | b)
  = a*(bbD | baQ | b)


```