---
title: Latex笔记
abbrlink: b3937b6d
tags:
---

## 第一章 熟悉Latex
1. 中文无法显示，需要使用`UTF8`选项
   ```latex
   \document[UTF8]{ctextart}
   ```
   其中`ctextart`表示中文文章，`UTF8`指明编码格式
2. `xelatex`适合于中文的编译
3. 通过`maketitle`来显示
   - `\today`
   - `\author`
   - `\title`
   等信息
4. 空行来形成段，在段首不需要手动缩进，汉字后的空格会被忽略，英文不会,但是英文的多个空格依然会被整理为一个空格

