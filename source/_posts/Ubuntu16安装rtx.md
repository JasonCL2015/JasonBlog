title: Ubuntu16安装rtx
keywords: 'Ubuntu,rtx'
date: 2016-09-18 21:03:04
categories: 
	- Linux 
tags: 
	- Linux
description:
---
<blockquote class="blockquote-center">Ubuntu16.04系统中安装腾讯通RTX走过的那些坑
</blockquote>



<!--more-->

## 一、腾讯RTX下载

> 腾讯通RTX是（Real Time eXpert）是腾讯公司推出的企业级实时通信平台，致力于帮助企业提高运作效率、降低沟通成本、拓展商业机会，是一种高度可管理、低成本、易部署的IT平台。RTX集成了丰富的沟通方式，包括文本会话、语音/视频交流、手机短信、文件传输、IP电话、网络会议、以及应用程序共享、电子白板等远程协作方式。

链接至[腾讯通官网下载地址][1]下载rtx客户端，最新版本是2015版。

## 二、安装wine

> Wine (Wine Is Not an Emulator)[即Wine不是一个模拟器]是一个在Linux和UNIX之上的,Windows 3.x和 Windows APIs的实现。注意，Wine不是Windows模拟器，而是运用API转换技术实做出Linux对应到Windows相对应的函数来调用DLL以运行Windows程序。

因为rtx没有Linux版本，所以我们需要在Ubuntu中安装wine来解决打开exe文件的问题。
利用[PPA源][2]进行快速安装wine

    sudo add-apt-repository ppa:wine/wine-builds
    sudo apt-get update

推荐安装winehq官方提供的[最新版本wine][3]

    sudo apt-get install wine-devel
安装wine下的window扩展包安装工具winetricks,在终端输入

    sudo wget http://winetricks.org/winetricks
    chmod +x winetricks
    sudo mv winetricks /usr/local/bin
安装rtx用到的相关扩展，在终端输入

    winetricks msxml3 gdiplus riched20 riched30 ie6 vcrun6 vcrun2005sp1

注意：如果相关扩展无法下载，可以去找其它下载源，手动解压到~/.cache/winetricks下。

##安装RTX
上述环境都准备好后，在终端输入

    wine rtxclient2015formal.exe

##中文方框乱码问题解决
终端输入
    winetricks corefonts
下载字体[字体1][4][字体2][5]
解压上面的两个文件后，拷贝文泉驿字体，并导入注册表文件

    cp wqy-microhei/wqy-microhei.ttc ~/.wine/drive_c/windows/Fonts/.
    regedit font.reg

全部完成！
> 参考阅读
- [字体问题解决](http://rubyisteleanor.blog.51cto.com/8723438/1578982)
- [安装教程](http://www.linuxdiyf.com/linux/21867.html)

  [1]: http://rtx.tencent.com/rtx/download/index.shtml
  [2]: https://launchpad.net/ubuntu/+ppas
  [3]: https://www.winehq.org/download
  [4]: http://down.51cto.com/data/1901084
  [5]: http://down.51cto.com/data/1901086