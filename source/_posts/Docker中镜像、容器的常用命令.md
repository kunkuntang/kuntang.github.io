title: Docker中镜像、容器的常用命令
author: Tommi_Max
tags:
  - 备忘翻阅
categories:
  - Docker
  - ''
date: 2019-11-21 15:15:00
---
## 简述

对于Docker来说整体还是比较容易入门的，理解起来跟虚拟机差不多，只要知道了镜像、容器的概念后，其他的就要考验Linux的知识了。

要理解Docker以及Docker里关于镜像、容器的概念，这里打个比方：
Docker就像一台支持软件多开的机子，Docker的镜像就像一个软件，Docker的容器就是这个软件多开时运行的窗口。每一个镜像都可以独立运行在不同的容器，他们互不干扰。下面简单列举以下有关镜像、容器的命令：

## Docker镜像

### 从网络上下截镜像

```
docker pull [mirror] NAME[:TAG]
```

例如：

```
docker pull ubuntu
```

如果不显式地指定TAG，则默认会选择latest标签，即下载最新版本的镜像。

想要下载指定版本的镜像，则在后面指明版本号：

```
docker pull ubuntu:14.04
```

也可以指明其它注册服务商的仓库地址下载对应的镜像：

```
C:\Users\kunta>docker pull hub.c.163.com/public/ubuntu:16.04-tools
16.04-tools: Pulling from public/ubuntu
[DEPRECATION NOTICE] registry v2 schema1 support will be removed in an upcoming release. Please contact admins of the hub.c.163.com registry NOW to avoid future disruption.
8a2df099fc1a: Pull complete
09aa8e119200: Pull complete
21a4b8922479: Pull complete
a3ed95caeb02: Pull complete
bd2a9dfe68fa: Pull complete
f132d6d54cc2: Pull complete
099b34b8b564: Pull complete
23afed40f2e5: Pull complete
6d60d1682f8c: Pull complete
97121ecff7e5: Pull complete
ef4015576a84: Pull complete
94d536c3a549: Pull complete
Digest: sha256:0fc4a48462a92b0a5a53b35540b8ca33a68606c6cd23c21df54c5f54bccbf33a
Status: Downloaded newer image for hub.c.163.com/public/ubuntu:16.04-tools
hub.c.163.com/public/ubuntu:16.04-tools
```

### 查看本地的存在的镜像

查看本地镜像可以执行`docker images`命令：

```
C:\Users\kunta>docker images
REPOSITORY                           TAG                 IMAGE ID            CREATED             SIZE
hub.c.163.com/kuntang/lingermarket   latest              c7a70a3810cf        23 months ago       418MB
hub.c.163.com/public/centos          6.7-tools           b2ab0ed558bb        2 years ago         602MB
```

其中各字段含义如下：

- REPOSITORY：镜像来自哪一个仓库
- TAG：镜像的标签信息
- IMAGE ID：镜像的ID号(唯一)
- CREATED：创建时间
- SIZE：镜像大小

![](https://user-gold-cdn.xitu.io/2019/10/10/16db635dd1090a6e?w=993&h=519&f=png&s=25613)

### 给本地镜像添加新标签

```
docker tag [mirror ID] [NEW TAG]
```

为方便使用，我们应该用`docker tag`命令为镜像加上别名；

```
C:\Users\kunta>docker tag 119 ubuntu

C:\Users\kunta>docker images
REPOSITORY                           TAG                 IMAGE ID            CREATED             SIZE
hub.c.163.com/kuntang/lingermarket   latest              c7a70a3810cf        23 months ago       418MB
ubuntu                               latest              1196ea15dad6        2 years ago         336MB
hub.c.163.com/public/ubuntu          16.04-tools         1196ea15dad6        2 years ago         336MB
hub.c.163.com/public/centos          6.7-tools           b2ab0ed558bb        2 years ago         602MB
```

或：

```
C:\Users\kunta>docker tag hub.c.163.com/public/ubuntu:16.04-tools ubuntu2:16.04

C:\Users\kunta>docker images
REPOSITORY                           TAG                 IMAGE ID            CREATED             SIZE
hub.c.163.com/kuntang/lingermarket   latest              c7a70a3810cf        23 months ago       418MB
ubuntu2                              16.04               1196ea15dad6        2 years ago         336MB
ubuntu                               latest              1196ea15dad6        2 years ago         336MB
hub.c.163.com/public/ubuntu          16.04-tools         1196ea15dad6        2 years ago         336MB
hub.c.163.com/public/centos          6.7-tools           b2ab0ed558bb        2 years ago         602MB
```

实际上这个命令不会创建新的镜像，仔细比较一下新创建的镜像，它们的ID都是一样的，所以它们指向的是同一个镜像，这里的tag命令相当于给它们新建了一个快捷方式。

### 查看镜像详细信息

```
docker inspect [IMAGE ID]
```

这个命令将会返回一串JSON字符串，如果想要查看某一项的内容，可以使用`-f`来指定。如：

```
docker inspect -f {{".[字段名]"}} [IMAGE ID]
```

> 注意：**字段名**前的`.`不能少。

如想要查看镜像的ID信息:

```
C:\Users\kunta>docker inspect -f {{".Id"}} 119
sha256:1196ea15dad679677c220abedb654d7e8f4402c88fc1985ef1e224b50206d729
```

### 搜寻镜像

```
docker search TERM 
```

支持参数包括 `--automated=false` 仅显示自动创建的镜像 `--no-trunc=false` 输出信息不截断显示 `-s, --start=0` 仅显示评价为指定星级以上的镜像

### 删除镜像

```
docker rmi [IMAGES ID or TAG]
```
** 当一个镜像拥有多个标签的时候，执行删除命令只会删除镜像对应的标签，当镜像只剩下最后一个标签时执行删除操作，则会彻底删除镜像 **

强制删除镜像

```
docker rmi -f [mirrors]
```

### 创建镜像

```
docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
```

### 上传镜像

```
docker push NAME[:TAG]
```

## Docker容器

### 创建容器

创建成功后返回容器ID

```
docker create [mirrors]
```

### 运行容器

```
docker start [mirrors ID]
```

### 创建并运行容器

```
docker run -i -t [mirrors ID] /bin/bash
```

`-t`选项让Docker分配一个伪终端(pseudo-tty)并绑定到容器的标准输入上，`-i`则让容器的标准输入保持打开。

在交互模式下，用户可以通过所创建的终端来输入命令，例如：

```
C:\Users\kunta>docker run -i -t b2a /bin/bash
[root@3eb81b9ab5ba /]# pwd
/
[root@3eb81b9ab5ba /]# ls
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  sbin  selinux  srv  sys  tmp  usr  var
```

用户可以使用`Ctrl+D`或者输入`exit`来退出交互模式。

### 守护态运行

```
docker run -d [mirrors ID]
```

### 进入容器
在使用-d参数时，容器启动后会进入后台，用户无法看到容器中的信息。某些时候如果需要进入容器操作，有多种方法，包括使用docker attach命令、docker exec命令，以及nsenter工具等。

#### attach命令

```
docker attach [mirrors ID]
```

使用attach命令有时候并不方便，当多个窗口同时attach到同一个容器时，所有窗口都会同步显示。当某个窗口因命令阻塞时，其它容器也无法执行操作了。

#### exec命令

Docker自1.3版本起，提供了一个更加方便的工具exec，可以直接在窗口内运行命令。例如进入到刚创建的容器中，并启动一个bash：

```
docker exec -it [mirrors ID] /bin/bash
```

### 终止容器

```
docker stop [mirrors ID]
```

### 重启已经停止的容器

重新启动一个容器
```
docker restart [mirrors ID]
```

### 查看当前容器列表

```
列出正在运行的容器
docker ps
列出所有的容器
docker ps -a
```

### 导出容器

```
docker export [mirrors ID] >[name].tar
```

### 导入容器

```
docker import - [name]
```

---

## 更多阅读

Docker系列文章：

1. [Docker中镜像、容器的常用命令](https://juejin.im/post/5d8820cae51d453b1e478b91)
2. [Docker仓库常用命令](https://juejin.im/post/5d9f53d26fb9a04e3142120c)
3. [Docker中容器的网络配置常用命令](https://juejin.im/post/5d9f4f86e51d4577f3534ead)