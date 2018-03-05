title: CentOS7安装docker-compose
keywords: 'DockerCompose,安装'
date: 2017-11-21 21:03:04
categories: Docker 
tags: Docker
description:
---
<blockquote class="blockquote-center">
compose是用来在docker中定义和运行复杂应用的小工具,比如在一个文件中定义多个容器,只用一行命令就可以让一切就绪并运行.
</blockquote>



<!--more-->
###安装pip:
![此处输入图片的描述][1]
这里显示，找不到相应的包？？
说没有python-pip软件包可以安装。
这是因为像centos这类衍生出来的发行版，他们的源有时候内容更新的比较滞后，或者说有时候一些扩展的源根本就没有。
所以在使用yum来search python-pip的时候，会说没有找到该软件包。因此为了能够安装这些包，需要先安装扩展源EPEL。
EPEL(http://fedoraproject.org/wiki/EPEL) 是由 Fedora 社区打造，为 RHEL 及衍生发行版如 CentOS、Scientific Linux 等提供高质量软件包的项目。
首先安装epel扩展源：
  ```
sudo yum -y install epel-release
  ```
  ![此处输入图片的描述][2]
然后安装python-pip
  ```
sudo yum install python2-pip.noarch
  ```
  ![此处输入图片的描述][3]
  ![此处输入图片的描述][4]
###对安装好的pip进行一次升级
  ```
sudo pip install --upgrade pip
  ```
  ![此处输入图片的描述][5]
安装docker-compose
  ```
pip install docker-compose
  ```
  ![此处输入图片的描述][6]
运行docker-compose
![此处输入图片的描述][7]
 
出现报错
pkg_resources.DistributionNotFound: backports.ssl-match-hostname>=3.5
使用pip 更新backports.ssl-match-hostname的版本
  ```
pip install --upgrade backports.ssl_match_hostname
  ```
更新backports.ssl_match_hostname 到3.5版本后问题解决
![此处输入图片的描述][8]
![此处输入图片的描述][9]


  [1]: http://ojb6w1er1.bkt.clouddn.com/%E5%AE%89%E8%A3%85dockercompose.png
  [2]: http://ojb6w1er1.bkt.clouddn.com/%E5%AE%89%E8%A3%85dockercompose2.png
  [3]: http://ojb6w1er1.bkt.clouddn.com/%E5%AE%89%E8%A3%85dockercompose3.png
  [4]: http://ojb6w1er1.bkt.clouddn.com/%E5%AE%89%E8%A3%85dockercompose4.png
  [5]: http://ojb6w1er1.bkt.clouddn.com/%E5%AE%89%E8%A3%85dockercompose5.png
  [6]: http://ojb6w1er1.bkt.clouddn.com/%E5%AE%89%E8%A3%85dockercompose6.png
  [7]: http://ojb6w1er1.bkt.clouddn.com/%E5%AE%89%E8%A3%85dockercompose7.png
  [8]: http://ojb6w1er1.bkt.clouddn.com/%E5%AE%89%E8%A3%85dockercompose8.png
  [9]: http://ojb6w1er1.bkt.clouddn.com/%E5%AE%89%E8%A3%85dockercompose9.png