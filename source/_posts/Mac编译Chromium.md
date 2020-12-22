---
title: Mac编译Chromium
abbrlink: 23dc19dd
date: 2020-12-18 20:40:27
tags:
---


由于最近看的一本书(TPOSA)中的第一章讲的就是Chromium，所以突然就想下载一下源码，然后又想要不就试试编译一下？

首先官方有一个[指导文档](https://chromium.googlesource.com/chromium/src/+/master/docs/mac_build_instructions.md)

可能不会列出所有的可能性，但是还是稍微记录一下我本人的具体步骤以及踩过的坑吧

时间：2020/12/17
Mac: macbook pro late 2013 15'
Mac系统版本: `10.15.7 (19H15)`
Chromium版本(hash): `a7361976c2c0be7139f4de8eb8844702a198f0cf`

参考的博客
- [macOS 10.15 下载及编译 Chromium](https://juejin.cn/post/6844904110848933902)
- [mac上面进行chromium编译](https://www.jianshu.com/p/4ba1d2275767)

<!-- more -->
1. clone源码
    ```sh
    # 直接从官方clone，即使开了梯子也可能会很慢？
    git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
    # 从GitHub官方镜像clone
    git clone https://github.com/chromium/chromium.git
    ```
    而且虽然GitHub有两种（ssh/https）clone方式，似乎还是https走代理的速度更快？

2. 安装`depot_tools`
   
   ```sh
   # 放到某一个合适的位置
   git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
   # 更改path
   export PATH="$PATH:/path/to/depot_tools"
   ```
   来自上述[官方文档](https://chromium.googlesource.com/chromium/src/+/master/docs/mac_build_instructions.md)
   
   尝试直接运行`gn gen out/Default`，出错
   `Error: client not configured; see 'gclient config'`
   需要先运行`gclient sync`命令
   直接运行出错：
   `gn.py: Could not find gn executable at xxx`
   需要配置文件`.gclient`
   在Chromium文件夹创建`.gclient`文件
   
3. 创建`.gclient`文件以运行`gn`命令
   在clone下来的Chromium文件夹创建`.gclient`文件并添加以下内容
   ```
    solutions = [
    {
        "managed": False,
        "name": "src",
        "url": "https://chromium.googlesource.com/chromium/src.git",
        "custom_deps": {},
        "deps_file": ".DEPS.git",
        "safesync_url": "",
    },
    ]
   ```
   使用`gclient sync`来获取第三方依赖？
   ```sh
   # 在Chromium文件夹
   gclient sync
   ```
   上述命令可能会执行很久，需要clone下大约20G的内容。根据[这片文章](https://zhuanlan.zhihu.com/p/70879583)。可以将上述配置中的"url"改为github的链接`https://github.com/chromium/chromium.git`，可能会更快？

4. 继续尝试执行`gn gen out/Default`
   出错
   `ERROR at //.gn:6:1: Unable to load "xxxx/chromium/third_party/angle/dotfile_settings.gni".`
   手动从angle仓库clone到`third_party`文件夹
   继续报错
   ```
    ERROR at //build_overrides/build.gni:5:1: Unable to load "/Users/samuel/git-repos/chromium/build/config/gclient_args.gni".
    import("//build/config/gclient_args.gni")
    ^---------------------------------------
    See //build/toolchain/toolchain.gni:9:1: whence it was imported.
    import("//build_overrides/build.gni")
    ^-----------------------------------
    See //build/config/coverage/coverage.gni:5:1: whence it was imported.
    import("//build/toolchain/toolchain.gni")
    ^---------------------------------------
    See //build/config/profiling/profiling.gni:6:1: whence it was imported.
    import("//build/config/coverage/coverage.gni")
    ^--------------------------------------------
    See //build/config/sanitizers/sanitizers.gni:8:1: whence it was imported.
    import("//build/config/profiling/profiling.gni")
    ^----------------------------------------------
    See //build/config/compiler/compiler.gni:9:1: whence it was imported.
    import("//build/config/sanitizers/sanitizers.gni")
    ^------------------------------------------------
    See //BUILD.gn:12:1: whence it was imported.
    import("//build/config/compiler/compiler.gni")
    ^--------------------------------------------
   ```
   根据[stackoverflow的回答](https://stackoverflow.com/questions/58039019/chromium-build-failed-on-ubuntu-error-at-build-overrides-build-gni51-unab)尝试运行以下命令
   `gclient sync --force`
   发现中途输出`Skipping Mac toolchain installation for mac`。感觉是不是还是需要安装以下xcode?发现[官方文档](https://chromium.googlesource.com/chromium/src/+/master/docs/mac_build_instructions.md#system-requirements)明确将developer SDK作为必需准备之一，说明确实需要安装xcode。（本来以为可以不装xcode）
   装完之后还是有上述的`Skipping Mac toolchain installation for mac`的问题，主要看了这篇[博客](https://juejin.cn/post/6844904110848933902)中提到的手动添加`mac_sdk_path`的方法，但是并没有什么用。后来看这一篇[博文](https://blog.csdn.net/dangwei_90/article/details/100926129)的时候才发现是不是自己执行`gn gen out/Default`的位置有问题，应该在`src`目录下执行而不是在根目录下执行。
   换到了`src`目录出现了新的问题`xcode-select: error: tool 'xcodebuild' requires Xcode, but active developer directory '/Library/Developer/CommandLineTools' is a command line tools instance`
   在[stackoverflow](https://stackoverflow.com/questions/17980759/xcode-select-active-developer-directory-error)中找到了答案，大概是因为我之前安装的`CommandLineTools`导致`xcode-select`没有指向新安装的xcode，通过
   `sudo xcode-select -s /Applications/Xcode.app/Contents/Developer`命令
   使得`gn gen out/Default`得以正常执行


5. 使用`autoninja`来进行编译
   报了很多奇怪的错误，似乎有什么python包缺失以及timeout的问题。接着在尝试通过`brew install ccache`来安装ccache的时候发现我的xcode似乎并没有选择接受license，同时第一次启动xcode又装了一些component，使用以下命令接受license
   ```sh
   sudo xcodebuild -license accept
   ```
   还是不行，尝试具体解决出现的问题。
   - `httplib2`缺失
     由于使用的是`conda`通过`conda install -c conda-forge httplib2`来安装
     发现并没有成功，还是报相同的错误
     查看错误中对应的`.py`文件`depot_tools/ninjalog_uploader.py`文件的开头，使用了`#!/usr/bin/env python`似乎是会使用`$PATH`中找到的第一个python，但是因为装了conda的原因，虽然直接`which python`发现此时的python确实就是conda下的python(`/usr/local/anaconda3/bin/python`)，但是实际运行`autoninja`的时候似乎还是会使用`/usr/bin/python`
     使用`source deactivate`可以取消conda的环境配置，然后使用`pip3 install httplib2`再次安装，成功

   进入src文件夹
   ```sh
   cd src
   autoninja -C out/Default chrome
   ```
   开始编译，需要注意的是编译可能会消耗大量的时间以及空间，以本机为例，编译花费了近8个小时，以及120G左右的空间
   
   编译结束之后可以在`src/out/Default/Chromium.app/Contents/MacOS`下找到Chromium的可执行文件