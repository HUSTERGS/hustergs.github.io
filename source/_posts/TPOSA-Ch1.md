---
title: TPOSA Ch1
abbrlink: ef4f8410
date: 2020-12-16 23:16:33
tags:
---


算是一本有点老的书了？因为之前大一的时候打印了出来一只没看，就想着要不找个时间看完算了

### High Performance Networking in Chrome

#### 1.2 The Many Facets of Performance
1. Chrome是作为一个平台来设计的。在Chrome之前的浏览器都是单一进程，每一个页面共享同一个内存空间，如果某一个页面出了问题，那么其他的页面也可能会受到影响。而Chromes是多进程来工作的。每一个tab页面都是一个单独的进程，拥有自己的内存空间以及一个*沙箱*(*sandbox*)
2. 浏览器的工作主要就是三个步骤
   - 获得资源
   - 页面渲染
   - 执行脚本
   其中页面渲染和脚本的执行是单线程并且交错执行的(*single-threaded, interleaved model of execution*)--无法对DOM进行并发的修改，这其中一部分是因为js本身是单线程执行的语言。所以提速的方向主要是在如何将两者一起结合起来快速的运行。

3. Chrome使用Blink作为layout引擎。而对于js，Chrome实现了V8 runtime
4. 浏览器对不同网络资源能够根据其延迟等信息来进行优先级的判定是非常重要的一点
<!-- more -->
#### 1.3 What is a Modern Web Application
1. 通过统计不同网站对于不同网络资源的请求情况（js, html, css, media）等。可以知道大部分的网络连接都是短而小的，但是底层的TCP连接是针对*large, streaming downloads*

#### 1.4 The Life of a Resource Request on the Wire
- 本地Cache
- DNS
- TCP
- SSL
- request for server
- server processing
- response from server

上述的这几个延时是针对单一资源请求的，占到了总时间的80%

#### 1.5 What is "Fast Enough"？
#### 1.6 Chrome‘s Network Stack from 10000 Feet
##### Multi-Process Architecture
一般来说Chrome是*process-per-site model*，每一个不同的site在不同的进程当中，而相同网站的实例被集中在一起。每一个tab页面有自己单独的Blink以及V8引擎，同时在自己的沙箱中运行。不同的进程对系统资源包括网络都只有有限的访问权限，想要获取相关权限需要向主进程(*kernel process*)发起申请（通过IPC）

##### Inter-Process Communication and Multi-Process Resource Loading
##### Cross-Platform Resource Fetching
##### Architechure and Performance on Mobile Platforms
在移动端与桌面端基本相同，在Android上由于内存的原因可能不能为每一个tab都分配一个Blink。而在iOS中由于系统层面的限制无法使用多进程，而是使用了多线程。同时考虑到流量的原因，诸如*prerendering*等特性可能只会在Wi-Fi状态下被启用。

##### Speculative Optimization with Chrome's Predictor
`Predictor`通过分析网络请求的规律来预测之后可能的模式


常规的用户模式
- 鼠标放在链接上可能会点击
- 每天访问的网站
- ...

#### 1.7 The Lifetime of Your Browser Session
