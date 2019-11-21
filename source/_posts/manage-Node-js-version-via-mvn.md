layout: '[post]'
title: mac下卸载Node.js并使用nvm来管理Node版本
tags:
  - 转载
  - 备忘翻阅
categories:
  - ''
date: 2018-11-12 01:07:00
---

> 转载自：https://www.zhihu.com/question/27389115/answer/36434788
> 转载自：https://blog.csdn.net/xieamy/article/details/70270039

<!-- more -->

### 前言

因为不小心的缘故，把自己电脑上的node版本由LTS升到了Current，从而遇到了之前有一个基于vue-cli上搭建的项目跑不起来，在读取配置文件的时候直接报fs模块的错误，因此有了想卸载Node.js再从新安装的想法。

### 当前环境

当前的环境是 Mac os Sierra 10.12.6, 由于之前是通过直接在官网上下载安装包的方式来安装Node，并没有使用类似n或者nvm这类的Node版本管理工具，所以卸载的方法有点不一样。

### 删除代码

1. 删除/usr/local/lib中的所有node和node_modules
2. 删除/usr/local/lib中的所有node和node_modules的文件夹
3. 如果是从brew安装的, 运行brew uninstall node
4. 检查~/中所有的local, lib或者include文件夹, 删除里面所有node和node_modules
5. 在/usr/local/bin中, 删除所有node的可执行文件
6. 最后运行以下代码:

```
sudo rm /usr/local/bin/npm
sudo rm /usr/local/share/man/man1/node.1
sudo rm /usr/local/lib/dtrace/node.d
sudo rm -rf ~/.npm
sudo rm -rf ~/.node-gyp
sudo rm /opt/local/bin/node
sudo rm /opt/local/include/node
sudo rm -rf /opt/local/lib/node_modules

```

### 安装nvm
 
打开nvm的github地址：[https://github.com/creationix/nvm](https://github.com/creationix/nvm)。在下面的简介中找到install，按照指示执行代码：

`curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.1/install.sh | bash`

当安装好之后会提示：

```
=> Appending nvm source string to /Users/kuntang/.bash_profile
=> Appending bash_completion source string to /Users/kuntang/.bash_profile
=> Close and reopen your terminal to start using nvm or run the following to use it now:

export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
Tomzi-Maxs-MacBook-Pro:bin kuntang$ nvm
-bash: nvm: command not found
```

这个时候直接敲 nvm还不能执行，需要执行一下它提示的代码：

`export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm`

这样就可以使用nvm还管理node版本了。
