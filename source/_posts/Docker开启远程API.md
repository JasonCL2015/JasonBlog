title: Docker开启远程API
keywords: 'Docker,网络通信'
date: 2017-10-19 21:03:04
categories: Docker 
tags: Docker
description:
---
<blockquote class="blockquote-center">
有时需要远程调用DockerAPI进行容器操作，比如Jenkins，本文讲述如何开启Docker远程API。
</blockquote>



<!--more-->

 * 修改配置文件 CentOS6:
  ```docker
/etc/sysconfig/docker
  ```
添加一行: 
  ```docker
DOCKER_OPTS='-H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock'
  ```
CentOS7:
  ```
/usr/lib/systemd/system/docker.service 
  ```
修改一行:
  ```
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock
  ```
 * 重启Docker 
   * CentOS6:
  ```
service docker restart
  ```
    * CentOS7:
  ```
systemctl daemon-reload
systemctl restart docker.service
  ```
 * 测试
在本机 
  ```
curl http://127.0.0.1:2375/info
  ```
在其他机器上：
  ```
curl http://hostanme:2375/info
  ```
 * 此时本地client可以继续通过Unix sock与docker daemon通行
例如：docker info 命令继续有效。

