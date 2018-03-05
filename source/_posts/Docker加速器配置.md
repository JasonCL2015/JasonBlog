title: Docker加速器配置
keywords: 'Docker,加速'
date: 2017-12-7 21:03:04
categories: Docker 
tags: Docker
description:
---
<blockquote class="blockquote-center">用了Docker的都知道在使用docker pull来拉取镜像时的速度都非常慢，而且还会经常报错（如：超时、镜像不存在等错误）。
主要原因大家都知道：Registry在国内，网络影响太严重。还好国内的一些大企业对此做了些解决办法：加速器，比较常用的有阿里云和DaoCloud。要使用加速器都需要先注册一个对应的账号，我这里已经在这两个网站上都注册过了。

</blockquote>



<!--more-->

## Centos7中配置Docker加速器
 * DaoCloud加速器
  ```
  curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://XXX.m.daocloud.io
  ```
  
> 注意：XXX是自己的地址，注册账号后可以得到这个地址，可以参考官方文档：[《配置 Docker 加速器》][1]，上面这条命令适用于Ubuntu14.04、Debian、CentOS6 、CentOS7，并且可以帮助您配置registry-mirror加速地址并重启Docker Daemon。

 * 阿里云加速器
  ```
  sudo cp -n /lib/systemd/system/docker.service /etc/systemd/system/docker.service
sudo sed -i "s|ExecStart=/usr/bin/docker daemon|ExecStart=/usr/bin/docker daemon --registry-mirror=https://XXX.mirror.aliyuncs.com|g" /etc/systemd/system/docker.service
sudo systemctl daemon-reload
sudo service docker restart
  ```
 > 注意：XXX也是注册阿里云账号后的一个地址，可以参考官方文档：[《配置Docker加速器》][2]
 
 **需要特别注意的是：阿里云在Linux系统中的加速器只支持Docker 1.9版本以上，Centos7以上，Ubuntu 12以上，个人感觉阿里云在Linux系统中的支持不如DaoCloud的完美。**
## Windows中配置Docker加速器

 * DaoCloud加速器
*为已存在的docker-machine配置加速器（假设machine名称为tmp-machine）*
  ```docker
docker-machine ssh tmp-machine
cd /var/lib/boot2docker
vi profile
  ```
*在EXTRA_ARGS中配置加速地址，即修改为：*
  ```docker
EXTRA_ARGS='
--label provider=virtualbox
--registry-mirror http://XXX.m.daocloud.io'
  ```
 > 注意：XXX需要更换成自己的加速地址（--registry-mirror后面是空格不是等号），保存并退出，然后输入命令：docker-machine restart tmp-machine重启虚拟主机。
 
*docker-machine create时配置加速器*
  ```docker
docker-machine create -d virtualbox --engine-registry-mirror=http://XXX.m.daocloud.io tmp-machine
  ```
 > 注意：XXX需要更换成自己的加速地址，这样创建了一个名为tmp-machine的docker虚拟机，并设置了相应的加速器。

 * 阿里云加速器

Windows中使用阿里云的加速器配置方法与配置DaoCloud加速器的方法是一样的，需要注意的是阿里云的加速器地址是以https://开头，格式如：https://XXX.mirror.aliyuncs.com，将上面所说的加速器地址更新成阿里云的就可以了。
个人用了加速器后的体会是：在Linux系统中DaoCloud的加速器比阿里云的要好用，在Windows系统中阿里云的加速器比DaoCloud的速度快。



  [1]: https://link.jianshu.com/?t=https://www.daocloud.io/mirror#accelerator-doc
  [2]: https://link.jianshu.com/?t=https://cr.console.aliyun.com/?spm=5176.2020520152.209.d103.Wa88wv#/accelerator
  [3]: https://launchpad.net/ubuntu/+ppas
  [4]: https://www.winehq.org/download
  [5]: http://down.51cto.com/data/1901084
  [6]: http://down.51cto.com/data/1901086