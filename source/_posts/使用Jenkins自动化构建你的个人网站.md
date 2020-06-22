title: 使用Jenkins自动化构建你的个人网站
author: Tommi_Max
tags:
  - 个人总结
categories:
  - 项目构建
date: 2020-06-22 18:49:00
---
## 前言

首先想要做这个自动化构建的初衷是，最近网站备案信息作了更改，我需要重新备案，但是现在备案可比几年前的时候严格多了，导致了现在网站的内容有好多不符合规范，因此我把个人网站修改了之后，再提交审核，结果又出现了其他问题被拒绝回来再次需要修改，这样来来回回搞了好几回，因为我的个人网站都是静态网站，每次修改后都要手动把代码发布到服务器上，这样的重复性工作显得很繁琐，所以研究了一下怎么用工具自动化构建自己的网站。

## 使用Docker在电脑上运行 Jenkins 平台

首先拉取Jenkins镜像，我使用的网易的蜂巢镜像源，镜像地址是：`hubhub.c.163.com/library/jenkins:latest`。[镜像仓库地址（需要登录才能查看）](https://c.163yun.com/hub#/library/repository/info?repoId=3093)  

运行命令`docker pull hub.c.163.com/library/jenkins:latest` 把镜像拉取到本地，然后为了日后使用方便，我们给这个镜像打上一个标签：

```
docker tag hubhub.c.163.com/library/jenkins:latest cubesuger/jenkins
```

> 我这里是使用cubeSuger/jenknis这个名称

根据这个镜像的使用说明，我们需要映射 8080 和 50000 端口，那我们使用下面的命令来运行这个镜像：

```
docker run -p 8080:8080 -p 50000:50000 --name jenkins -v D:/docker/docker-file/jenkins:/var/jenkins_home cubesuger/jenkins
```

> -p 是映射端口，--name 是为运行镜像所创建的容器添加一个名称（名称为 jenkins）, -v 是把 Jenkins 的配置和插件保存到我们的磁盘上

执行后等待一会儿，便会看到下面内容便是成功启动了 Jenknis。

```
INFO:

*************************************************************
*************************************************************
*************************************************************

Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:

91a3b5b09eea439689e5faead48e9891

This may also be found at: /var/jenkins_home/secrets/initialAdminPassword

*************************************************************
*************************************************************
*************************************************************

--> setting agent port for jnlp
--> setting agent port for jnlp... done
```

> 注意这里包含了一个密码串`91a3b5b09eea439689e5faead48e9891`，这个密码串会在接下来的登录中用到

在浏览器打开 `localhost:8080` 就会看到jenkins的初始化界面：

![jenkins_image1](/images/jenkins_image1.png)

我们填上Jenkins的默认密码串，密码串可以在刚刚启动时输出的内容中找到。找不到的也可以到`/var/jenkins_home/secrets/initialAdminPassword`这个目录下用文件编辑器打开查看。（`/var/jenkins_home`这个目录我们映射到了`D:/docker/docker-file/jenkins`这个目录中，所以可以在电脑上直接到这个目录上打开查看，而不用在容器中用cat命令查看）

输入后便会跳转到用户管理界面，这个时候最好修改一下原始密码，防止以后忘记要进入容器中寻找。

一般来说我们从蜂巢上下载下来的Jenkins版本会相对老一些，这时候我们应该对jenkins做升级，防止因为版本而出现问题。

## 更新Jenkins

我们可以从清华镜像中下载最新版本的Jenkins，[（下载地址：://mirrors.tuna.tsinghua.edu.cn/jenkins/war/latest/）](https://mirrors.tuna.tsinghua.edu.cn/jenkins/war/latest/)，打开后选择下载jenkins.war即可，然后我们把下载好的文件移动到刚刚我们给容器共享的目录中（这里我设置的目录是：D:/docker/docker-file/jenkins），然后我们执行命令以root身份进入容器：

```
docker exec -it -u root [containerID] /bin/bash
``` 

> 注：containerID 就是创建容器是的ID。

> 默认情况下会以 jenkins 这个用户进入容器，但是这个用户只能操作jenkins_home下的目录，所以要更新jenkins版本，就要用root用户进入容器进行操作。


然后我们查看当前jenkins的war包在哪里，点击设置，查看系统信息

等重启完毕后就能看到新版本的Jenkins在运行了：

![jenkins_image2](/images/jenkins_image2.png)


## 更新国内源

默认情况下Jenkins会使用国外的地址进行更新和插件下载，这些地址访问时可能会非常慢，因此如果出现这种现象的时候，我们应该把更新地址改为国内的镜像地址。更新为国内源需要两步：

1. 修改`jenkins_home/update`目录下`default`文件里的地址，因为我们在启动容器的时候做了映射，所以这个文件可以在`D:\docker\docker-file\jenkins\updates`这个目录中找到，打开`default`文件，把里面的`http://www.google.com/`全部改成`http://www.baidu.com/`，把`http://updates.jenkins-ci.org/download/`全部改成`https://mirrors.tuna.tsinghua.edu.cn/jenkins/`，然后保存修改。
2. 打开jenkins控制台，进入插件管理，点击Advanced（高级）,在最下面的update site 中填写`https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json`这个地址，点击submit（提交）然后等待刷新然后重启就可以了。

修改完成后可以发现插件的更新速度有了明显的提升。

## 安装必要的插件

接下来安装必要的插件来完成我们的自动化构建。

### 汉化插件

对于英文不好的朋友，可以先去安装一个汉化插件，名字叫 Localization Chinese(Simplified)。安装完后重启就可以看到汉化后的界面了。


![](/images/jenkins_image3.png)

### 安装git插件

在插件中搜索git并安装，一般来说在安装git的过程中会把凭证插件也安装上，所以就不用手动去安装凭证插件了，如果发现没有安装凭证插件，则需要手动安装一下。

### 安装并配置凭证插件

要想自动化构建网站，首先要可以获取到构建网站的代码，这就涉及到安全验证的问题，比如GitHub的私有仓库就有两种方式来获取代码，ssh和用户密码。还有我们有时候在构建好代码后，需要登录到服务器上把构建好的代码放到指定目录上去，也需要用到凭证。

安装Credentials Binding插件，安装完成后重启jenkins。

重启完成后点击管理凭证，然后点击全局，添加一个凭证，这里可以选择 用户名和密码 、ssh等方式。选择自己想要的验证方式并且配置就可以了。用户名和密码比较简单，这里就列举比较复杂的ssh方式：

首先进入到容器中生成公钥和私钥[（这里参考GitHub上的教程）：](https://help.github.com/en/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)

输入命令然后按回车即可：

`ssh-keygen -t rsa -b 4096 -C "your_email@example.com"`

然后执行命令把秘钥添加到ssh-agent中：

`eval "$(ssh-agent -s)"`

然后把新生成的秘钥添加到GitHub上[（参考）](https://help.github.com/en/github/authenticating-to-github/adding-a-new-ssh-key-to-your-github-account)

最后在jenkins的凭证中配置：

![jenkins_image4](/images/jenkins_image4.png)

私钥中选中在容器中创建的 `~/.ssh/id_rsa`，然后保存即可。

### 安装Node.js

搜索node.js插件并安装，安装完成后到系统管理，全局工具配置中配置Node.js。


![jenkins_image5](/images/jenkins_image5.png)

这里我选择安装了两个版本的Node.js，都是使用从nodejs.org官网上的安装方法，如果你的项目有使用到一些全局的包也可以在下面的`Global npm packages to install`选项中配置。

### 安装Publish Over SSH

当我们把项目在容器中构建好后，就需要把构建好的代码发送到生产服务器上，这个时候就需要用到这个插件：`Publish Over SSH`

搜索`Publish Over SSH`插件并安装，安装好在系统管理——系统配置中配置`Publish Over SSH`。

![jeninks_image6](/images/jeninks_image6.png)

这样就可以在构建流程中使用了。

### 创建工作流程

首先创建一个自由风格的项目，给项目起个名字，然后就进入到了构建流程配置中：

![jenkins_image7](/images/jenkins_image7.png)

1. 可以给项目添加一个描述
2. 然后在源码管理中选择git，添加git仓库地址，凭证选择之前创建好的凭证，分支选择你需要构建的分支即可。
3. 如果在构建之前需要登录服务器把之前的旧目录删除，则可以在构建环境中勾选`Send files or execute commands over SSH before the build starts`，然后选择之前配置好的服务器，在`Exec command`中编写命令：

![jenkins_image8](/images/jenkins_image8.png)

4. 勾选紧接着的`Provide Node & npm bin/ folder to PATH`，然后选择当前项目需要用到的Node.js版本。

![jenkins_image9](/images/jenkins_image9.png)

5. 然后在构建中输入需要执行的构建命令。

![jenkins_image10](/images/jenkins_image10.png)

6. 最后把构建好的代码发送到生产服务器上。

![jenkins_image11](/images/jenkins_image11.png)


点击保存后就可以进行构建了。

第一次构建的时候需要安装node.js和一些全局的npm包，所以等待的时间会久一些，之后的构建速度就快很多了。

## 结尾

至此，我们就已经完成了使用Jenkins来自动化构建自己的网站应用了。