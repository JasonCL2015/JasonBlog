title: Docker容器访问外部网络
keywords: 'Docker,网络通信'
date: 2017-10-19 21:03:04
categories: Docker 
tags: Docker
description:
---
<blockquote class="blockquote-center">
最近工作中部署一个项目，在项目内部需要访问外网。给某云上传文件，但是一直报unknown host，无法解析域名，然后找了好久原因，下面废话不多说，来一起看看详细的解决方法。
</blockquote>



<!--more-->
##解决方法
Linux系统默认没有打开IP转发功能，要确认IP转发功能的状态，可以查看/proc文件系统，使用下面命令：
  ```
cat /proc/sys/net/ipv4/ip_forward
0
  ```
如果上述文件中的值为0,说明禁止进行IP转发；如果是1,则说明IP转发功能已经打开,要想打开IP转发功能，可以直接修改上述文件：
  ```
echo 1 > /proc/sys/net/ipv4/ip_forward
  ```
把文件的内容由0修改为1。禁用IP转发则把1改为0。
上面的命令并没有保存对IP转发配置的更改，下次系统启动时仍会使用原来的值，要想永久修改IP转发，需要修改/etc/sysctl.conf文件，修改下面一行的值：
  ```
net.ipv4.ip_forward = 1
  ```
修改后可以重启系统来使修改生效，也可以执行下面的命令来使修改生效：
  ```
sysctl -p /etc/sysctl.conf
  ```
进行了上面的配置后，IP转发功能就永久开启了
