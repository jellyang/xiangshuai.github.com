---
layout: post
title: 在Cygwin中安装Jekyll
categories:
- 教程
tags:
- cygwin
- jekyll
---

前段时间发现[Cygwin](https://www.cygwin.com/)，可以在windows平台模拟linux下的大部分环境，而且通过其管理git也非常方便，但是在Cygwin下配置jekyll是遇到许多问题，也在网上找了许多方法，尝试了好久最后终于配置成功，在这里记录一下，以后再次配置时能够轻松一点。

安装jekyll需要ruby环境，这里通过rvm进行ruby环境的安装。另外，还需要一些依赖包：*patch，zlib-devel，openssl，openssl-devel，libyaml-devel，libyaml0_2，sqlite3，make，libtool，gcc-core，autoconf，automake，bison，m4，mingw64-i686-gcc，mingw64-x86_64-gcc，cygwin32-readline*等，如果不想逐个安装，最好先把Cygwin的包管理软件配置一下。

**1. 配置包管理环境：**

	cp /cygdrive/f/package/setup-x86_64.exe /usr/local/bin/setup-x86_64.exe

*注：setup-x86[_64].exe名字不能改，因为自动进行软件下载时，会使用setup-x86[_64].exe命令进行下载，所以要确保名字无误，否则可能安装失败。*

**2. 安装rvm**

	curl -L https://get.rvm.io | bash -s stable –ruby

安装完rvm，根据控制台输出日志启动rvm，日志信息如下：

![](http://xiangshuai.github.io/resources/20140831201602.jpg)

**3. 启动rvm后，安装ruby（如果ruby版本低于2.1时）**

	rvm list known
	rvm install 2.1
	rvm use 2.1 --default

**4. 安装jekyll**

	gem update --system
    gem list
	gem install jekyll
*安装jekyll可能会出错，错误信息：*

![](http://xiangshuai.github.io/resources/20140831201702.jpg)

错误主要是由ffi-1.9.3.gem安装的问题，可在[这里](https://github.com/ffi/ffi/issues/284)找到解决方法，具体如下图：

![](http://xiangshuai.github.io/resources/20140831201802.jpg)

即：在Cygwin中安装*libffi6，libffi-devel，cygwin32-libffi，pkg-config*，并将PKG_CONFIG_PATH指向为/lib/pkgconfig。
	
	export PKG_CONFIG_PATH="/lib/pkgconfig"

再次安装jekyll便可顺利安装*ffi-1.9.3*了：

![](http://xiangshuai.github.io/resources/20140831201902.jpg)

这样，就可以在Cygwin中成功安装jekyll了 ^_^ ！