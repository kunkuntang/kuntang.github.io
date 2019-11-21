title: Docker中容器的网络配置常用命令
author: Tommi_Max
tags:
  - 备忘翻阅
  - ''
categories:
  - Docker
date: 2019-11-21 15:22:00
---
## 网络基础配置

虽然Docker可以根据镜像“多开”容器，并而每个容器互不影响，但并不代表容器与容器之间是完全决裂的。Docker在运行镜像的时候提供了映射容器端口到宿主主机、容器端口到另一个容器的网络互联功能，使得容器与宿主主机、容器与容器之间可以相互通信。

###　从外部访问容器应用

在启动容器的时候，如果不指定对应的参数，在容器外是无法通过网络来访问容器内的网络应用和服务的。当容器中运行一些需要被外部访问的网络应用时，可以通过-P或者-p参数来指定端口映射。当使用-P标记时，Docker会随机映射一个49000~49900的端口至容器内部开放的网络端口：

```
docker run -d -p [mirror ID or TAG]
```

使用-p （小写）则可以指定要映射的端口，并且在一个指定端口上只可以绑定一个容器。支持的格式有ip:hostPort:containerPort | ip::containerPort | hostPort:containerPort。

#### 映射所有接口地址

使用hostPort:containerPort将本地的5000端口映射到容器的5000端口：

```
docker run -d -p 5000:5000 training/webapp python app.py
```

此时默认会绑定本地所有接口上的所有地址。多次使用-p标记可以绑定多个端口：

```
docker run -d -p 5000:5000 -p 3000:80 training/webapp python app.py
```

#### 映射到指定地址的指定端口

可以使用ip:hostPort:containerPort格式指定映射使用一个特定地址，比如localhost地址127.0.0.1:

```
docker run -d -p 127.0.0.1:5000:5000 training/webapp python app.py
```

> 也可以是内部其它容器的IP地址。

#### 映射到指定地址的任意端口

使用ip::containerPort绑定localhost的任意端口到容器5000端口，本地主机会自动分配一个端口：

```
docker run -d -p 127.0.0.1::5000 training/webapp python app.py
```

还可以使用udp标记来指定udp端口：

```
docker run -d -p 127.0.0.1:5000:5000/udp training/webapp python app.py
```

#### 查看映射端口配置

使用docker port查看当前映射的端口配置，也可以查看到绑定的地址：

```
docker port nostalgic_morse 5000
```

> 容器有自己的内部网络和IP地址（使用docker inspect+容器ID可以获取所有的变量值）。

### 容器互联实现容器间的互通信

容器的连接系统是除了端口映射外另一种可以与容器中应用进行交互的方式。它会在源和接收容器之间创建一个隧道，接收容器可以看到源容器指定的信息。

#### 自定义容器命名

连接系统依据容器的名称来执行。因此首先需要自定义一个好记的容器命名。
虽然当创建容器的时候，系统默认会分配一个名字，但自定义命名容器有两个好处：

- 自定义的命名比较好记
- 当要连接其它容器时，可以作为一个有用的参数点，比如连接web容器到db容器。

使用--name标记可以为容器自定义命名：

```
docker run -d -p --name web training/webapp python app.py
```

使用docker ps可以查看命名，或者使用docker inspect来查看容器的名字：

```
docker inspect -f "{{name}}" [mirror ID]
```

> 容器的名称是唯一的，如果已经命名了一个叫web的容器，必须先用docker rm命令来删除这个容器，才能再以web这个名称创建新容器。

容器互联

使用--link参数可以让容器之间安全地进行交互。

--link参数格式是--link name:alias，其中name是要链接容器的名称，alias是这个连接的别名。

比如我们先创建一个新的数据库容器：

```
docker run -d --name db training/postgres
```

然后再创建一个web容器，并将它连接到db容器：

```
docker run -d -p --name web --link db:db training/webapp python app.py
```

这个时候db容器就与web容器可以互相通信了。可以使用docker ps来查看容器的连接。

> 使用--link参数可以让Docker在两个容器之间通过一个安全的隧道互相通信，而不用通过开放端口的方式来实现，避免了把端口暴露到外部网络上。

#### 查看公开容器的接连信息

- 环境变量：使用env命令来查看容器的环境变量

```
docker run --name web --link db:db training/webapp env
```

- /etc/hosts文件：使用link参数时，Docker会添加host信息到父容器的/etc/hosts的文件。下面是父容器web的hosts文件

```
docker run -t -i --link db:db training/webapp /bin/bash
root@aed84ee21bd3:/opt/webapp# cat /etc/hosts
127.17.0.7 aed84ee21bde
...
172.17.0.5 db
```

第一个是web容器的host信息，默认用自己的id为主机名。第二个是db容器的ip和主机名。

---

## 更多阅读

Docker系列文章：

1. [Docker中镜像、容器的常用命令](https://juejin.im/post/5d8820cae51d453b1e478b91)
2. [Docker仓库常用命令](https://juejin.im/post/5d9f53d26fb9a04e3142120c)
3. [Docker中容器的网络配置常用命令](https://juejin.im/post/5d9f4f86e51d4577f3534ead)