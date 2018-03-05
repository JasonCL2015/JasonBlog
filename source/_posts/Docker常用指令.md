title: Docker常用指令
keywords: 'Docker'
date: 2017-12-12 10:02:44
categories: Docker
tags: Docker
description:
---
<blockquote class="blockquote-center">
Docker常用指令
</blockquote>



<!--more-->

### 容器清理

  ```java
    docker ps -a | grep "Exited" | awk '{print $1 }'|xargs docker stop
    
    docker ps -a | grep "Exited" | awk '{print $1 }'|xargs docker rm
    
    docker images|grep none|awk '{print $3 }'|xargs docker rmi
  ```

### 查看虚悬镜像 
  ```java
    docker images -f dangling=true
  ```
### 删除虚悬镜像 
  ```java
    docker rmi $(docker images -q -f dangling=true)
  ```
### 查看在mongo:3.2之后建立的镜像  
  ```java
    docker images -f since=mongo:3.2
  ```
### 查看在mongo:3.2之前建立的镜像  
  ```java
    docker images -f before=mongo:3.2
  ```
### 在容器基础上构建新镜像 
  ```java
    docker commit [选项] <容器ID或容器名> [<仓库名>[:<标签>]]

    docker commit \
    --author "Tao Wang <twang2218@gmail.com>" \
    --message "修改了默认网页" \
    webserver \
    nginx:v2
    sha256:07e33465974800ce65751acc279adc6ed2dc5ed4e0838f8b86f0c87aa1795214
  ```
### 运行容器
  ```java
    docker run --name web2 -d -p 81:80 nginx:v2
  ```
### 删除镜像
  ```java
    docker rmi [选项] <镜像1> [<镜像2> ...]
  ```
### 删除容器
  ```java
    docker rm
  ```
### 清除所有终止状态的容器
  ```java
    docker rm $(docker ps -a -q)
  ```
### 创建数据卷容器
  ```java
    sudo docker run -d -v /dbdata --name dbdata training/postgres
    echo Data-only container for postgres
  ```
### 挂载数据卷容器中的数据卷
  ```java
    $ sudo docker run -d --volumes-from dbdata --name db1 training/p
    ostgres
    $ sudo docker run -d --volumes-from dbdata --name db2 training/p
    ostgres
  ```
    
### build: 构建镜像
  ```java
    docker build -t oauth2-server:0.6.0 .
  ```
### run: 运行镜像
  ```java
	//守护进程形式 
    docker run -d -p 8180:8180 --name oa oauth2-server:0.6.0
	//shell交互形式(本服务不支持，可运行其他镜像) 
    docker run -i -t --rm java:8 /bin/bash
  ```
### 查看镜像
  ```java
    docker images
  ```
### 查看容器
  ```java
    docker ps -a
  ```
### 查看日志
  ```java
    docker logs -f --tail=100 ContainerId
    	* -f, --follow=false 跟踪日志输出。 
    	* -t, --timestamps=false 显示时间戳。 
    	* --tail="all" 输出日志尾部特定行(默认是所有)。 
    	* --since="" 时间段
  ```

### 查看磁盘挂载
  ```java
    docker inspect -f {{.Volumes}} ContainerId
  ```
### 导出日志目录
  ```java
    docker cp ContainerId:/var/log/oauth2-server /local/path
  ```
### 谨慎操作
* 删除一个镜像
  ```java
    docker rmi -f ImageId
  ```
* 删除所有镜像 
  ```java
    docker rmi -f $(docker images -q) 
  ```
* 删除一个容器
  ```java
    docker rm -v -f ContainerId
  ```
* 删除所有容器 
  ```java
    docker rm -v -f $(docker ps -a -q)
  ```
* 删除旧的容器 
  ```java
    docker ps -a | grep 'weeks ago' | awk '{print $1}' | xargs docker rm -v -f
  ```
* 删除停止状态的容器 
  ```java
    docker rm -v -f $(docker ps -a -q -f status=exited)
  ```
* 设置容器的磁盘空间（需要重启docker）
  ```java
　docker -d --storage-opt dm.basesize=50G
  ```
* 在容器中执行命令
  ```java
　docker exec -i -t ContainerName myshell 
	* -d, --detach=false Detached mode: run command in the background
	* -i, --interactive=false Keep STDIN open even if not attached
	* -t, --tty=false Allocate a pseudo-TTY
	* -u, --user= Username or UID (format: <name|uid>[:<group|gid>])
 如: docker exec -i -t ContainerName /bin/bash
  ```
* 提交容器当前状态到镜像
  ```java
　docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]

	* -a, --author="" Author (e.g., "John Hannibal Smith <hannibal@a-team.com>")
	* -c, --change=[] Apply specified Dockerfile instructions while committing the image
	* -m, --message="" Commit message
	* -p, --pause=true Pause container during commit
  ```
* 保存/备份镜像到本地
  ```java
	* docker save -o /local/path/fedora-latest.tar fedora:latest
	* docker save fedora:latest > /local/path/fedora-latest.tar
  ```
* 加载本地镜像文件到docker
  ```java
	* docker load --input fedora.tar
	* docker load < fedora.tar
  ```
* 容器与主机文件/目录传输
  ```java
	* docker cp [options] CONTAINER:PATH LOCALPATH
	* docker cp [options] LOCALPATH CONTAINER:PATH
  ```
* 连接到运行的容器中（谨慎使用，不推荐）
   ```java
    * docker attach [OPTIONS] CONTAINERID
	* --no-stdin=false Do not attach STDIN
	* --sig-proxy=true Proxy all received signals to the process
  ```
* 退出attach
* 默认情况下，如果使用ctrl-c退出container,那么container也会stop
* 按ctrl-p ctrl-q可以退出到宿主机，而保持container仍然在运行
* 查看容器或镜像的详细信息
  ```java
　docker inspect [OPTIONS] CONTAINER|IMAGE [CONTAINER|IMAGE...]

	* -f, --format="" Format the output using the given go template
	* --type=container|image Return JSON for specified type, permissible values are "image" or "container"
  ```
 * docker run 可选参数
     * -c/--cpu-shares 512 每个CPU的配额（1024允许占用全部CPU）
	 * --cpuset=0,1 只允许运行在前两个CPU上（--cpu 2）
	 * --cpu 4 使用四核（加载run命令的末尾）
	 * -m 128m 内存分配 详细参考Docker容器资源管理
	 * --dns 覆盖容器默认dns配置
	 * --mac-address 覆盖容器默认mac地址配置
	 * --add-host 添加hosts（如--add-host db-static:86.75.30.9）
	 * --security-opt 指定安全策略（主机配置）
	 * --privileged Docker将拥有访问主机所有设备的权限
	 * --device 指定容器可访问的设备（如--device=/dev/snd:/dev/snd:rwx rwx为权限）
	 * --rm 容器停止后删除容器(不与-d一起使用) 
	 * -e 设置环境变量（如-e "deep=purple"）
	 * -h 来设定hostname
	 * --link name:alias 连接其他容器
	 * -v $HOSTDIR:$DOCKERDIR 挂载主机目录到容器目录
	 * --read-only 设置容器只读
 * docker run --net 参数
	 * none。关闭容器内的网络连接 将网络模式设置为none时，这个容器将不允许访问任何外部router。 这个容器内部只会有一个loopback接口，而且不存在任何可以访问外部网络的router。
	 * bridge。通过veth接口来连接容器，默认配置。 Docker默认会将容器设置为bridge模式。此时在主机上面将会存在一个docker0的网络接口， 同时会针对容器创建一对veth接口。其中一个veth接口是在主机充当网卡桥接作用，另外一个veth接口存在于容器的命名空间中， 并且指向容器的loopback。Docker会自动给这个容器分配一个IP，并且将容器内的数据通过桥接转发到外部。
	 * host。允许容器使用host的网络堆栈信息。 注意：这种方式将允许容器访问host中类似D-BUS之类的系统服务，所以认为是不安全的。 当网络模式设置为host时，这个容器将完全共享host的网络堆栈。host所有的网络接口将完全对容器开放。 容器的主机名也会存在于主机的hostname中。这时，容器所有对外暴露的端口和对其它容器的连接，将完全失效。
	 * container。使用另外一个容器的网络堆栈信息。 当网络模式设置为Container时，这个容器将完全复用另外一个容器的网络堆栈。 同时使用时这个容器的名称必须要符合下面的格式：--net container:<name|id>.

 * 查看镜像历史
  ```java
    docker history [OPTIONS] IMAGE
	-H, --human=true Print sizes and dates in human readable format
	--no-trunc=false Don't truncate output
	-q, --quiet=false Only show numeric IDs
  ```
 * 导出镜像
  ```java
    docker export [OPTIONS] CONTAINER
	docker export exampleimage > exampleimage.tar
	docker export --output="exampleimage.tar" exampleimage
  ```
 * 导入镜像
  ```java
    docker import URL|- [REPOSITORY[:TAG]]
	docker import http://example.com/exampleimage.tgz
	cat exampleimage.tgz | docker import - exampleimagelocal:new
  ```
 * 其他
  ```java
	docker tag IMAGE[:TAG] [REGISTRYHOST/][USERNAME/]NAME[:TAG]
	docker stop container --stops it.
	docker start container --will start it again.
	docker restart container --restarts a container.
	docker kill container --sends a SIGKILL to a container.
	docker wait container --blocks until container stops.
	docker port container --shows public facing port of container.
	docker top container --shows running processes in container.
	docker stats container --shows containers' resource usage statistics.
	docker diff container --shows changed files in the container's FS.
	docker events --gets events from container.
	docker login --to login to a registry.
	docker search --searches registry for image.
	docker pull pulls an image from registry to local machine.
	docker push --pushes an image to the registry from local machine.
  ```
    

### 本地生成镜像，打包发布

#### 本地项目根目录下创建Dockerfile
  ```java
    FROM java:openjdk-8-jre
    WORKDIR /usr/src/
    COPY target/academic-0.0.1.jar /usr/src/
    CMD ["java", "-Duser.timezone=GMT+08", "-jar", "academic-0.0.1.jar"]
    RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
    EXPOSE 8072
  ```
#### 在Dockerfile所在目录执行创建镜像指令
  ```java
    docker build -t academic .  (-t academic是为镜像指定名称为academic)
  ```
#### 打包镜像为tar文件
  ```java
    docker save academic > academic.tar
  ```
#### 上传academic.tar至服务器
#### ssh到服务器，执行导出镜像命令
  ```java
    docker load < academic.tar
  ```
#### 编写docker-compose.yml文件
  ```java
    执行dc up -d 即可
  ```
