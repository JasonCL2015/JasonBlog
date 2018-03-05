title: 离线安装docker最新版
keywords: 'Docker,安装'
date: 2017-11-25 21:03:04
categories: Docker 
tags: Docker
description:
---
<blockquote class="blockquote-center">
离线安装Docker最新版
</blockquote>



<!--more-->

 
 如果安装了以前版本，还要删除以下这个包。
  ```
container-selinux.noarch 2:2.10-2.el7
  ```

离线安装三个rpm
  ```
-rw-r--r--. 1 root root 19524568 May 15 10:04 docker-ce-17.03.1.ce-1.el7.centos.x86_64.rpm
-rw-r--r--. 1 root root 29108 May 15 10:04 docker-ce-selinux-17.03.1.ce-1.el7.centos.noarch.rpm
-rw-r--r--. 1 root root 50076 May 16 08:32 libtool-ltdl-2.4.2-22.el7_3.x86_64.rpm
  ```
下载地址：
  ```
https://download.docker.com/linux/
https://centos.pkgs.org/7/centos-updates-x86_64/libtool-ltdl-2.4.2-22.el7_3.x86_64.rpm.html
http://mirror.centos.org/centos/7/updates/x86_64/Packages/libtool-ltdl-2.4.2-22.el7_3.x86_64.rpm
  ```
 ![此处输入图片的描述][1]
![此处输入图片的描述][2]


  [1]: http://ojb6w1er1.bkt.clouddn.com/docker%E7%A6%BB%E7%BA%BF%E5%AE%89%E8%A3%85.png
  [2]: http://ojb6w1er1.bkt.clouddn.com/docker%E7%A6%BB%E7%BA%BF%E5%AE%89%E8%A3%852.png