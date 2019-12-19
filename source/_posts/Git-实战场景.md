title: Git 实战场景
author: Tommi_Max
tags:
  - 备忘翻阅
categories:
  - Git
date: 2019-12-19 19:54:00
---
### 情景一：开发中途添加远端仓库

##### 通常情况

这个场景通常是：开发者在接到新项目时直接在本地开发，到了开发中途才发现需要把工程纳入到版本管理，这个时候他在代码托管平台上新建了一个空的远程仓库，然后执行一下命令：

```bash
// 添加远程仓库地址并命名为origin
git remote add origin [URL]

// 提交代码到远程仓库并将origin设置为默认提交仓库
git push -u origin master
```

> 设置了默认提交仓库后，以后再往origin这个远程仓库提交代码就可以不用指定直接执行`git push`就可以了。

##### 特殊情况

但由于一般的代码托管平台在新建仓库时可以选择**初始化仓库**，也就是会新建一个**README.md**文件并生成一次提交。如下图：

![upload successful](/images/image-20191128113937513.png)

如果不小心在新建的时候勾选了这个一个选项，那么你将可以有一下方法处理：

一、 删掉重建

- 删掉刚刚新建的远程仓库。
- 重新再建一个空的仓库。
- 按照**通常情况**的流程走一遍就可以了。

这个是最安全又快捷的方法，但如果你出于某些原因无法再新建一个仓库，比如说你没有删除仓库的权限，而你又不想麻烦你的上级来处理这种低级的问题，怕上级会因此怀疑你的能力，那么你可以尝试下面的操作。

二、本地合并

- 根据本地的master分支新建一个备份分支master_backup。
- 切换到备份分支，然后删除本地的master分支。
- 本地切换到master分支。
- 切换回master_bakcup分支并变基到master。
- master分支合并master_backup分支（Fast-forward)。
- 删除master_bakcup分支。

一开始仓库的图为如下：

![upload successful](/images/image-20191128140108874.png)

然后使用`git branch master_backup `命令把master上的代码备份到`master_backup`分支上，再使用`git checkout master_backup`切换到`master_backup`分支，使用` git branch -D master`删除本地的`master`分支。操作完如下图所示：

![upload successful](/images/image-20191128142021000.png)

接着使用`git checkout mater`切换到master分支，这步操作会直接从远端的master上拉取代码，并在本地上新建一个master分支。确认本地存在了master分支后，就可以使用`git checkout master_backup`切换回master_backup分支，最后使用`git rebase master`将maser_backup上的代码变基到master上。变基完如下图：

![upload successful](/images/image-20191128142447611.png)

可以看到，本地的master分支和master_backup分支已经连成一条直线了。最后再切换回master分支，然后使用`git merge master_backup`合并（Fost-forward）master_backup分支就完成了代码的迁移了。此时master_backup分支上的代码就可以删除掉了。最后分支图如下：

![upload successful](/images/image-20191128142917368.png)



## 场景二：忘记把忽略文件添加到.gitignore中

当你忘记添加 .gitignore 文件，不小心把一个很大的不需要追踪的文件添加到暂存区时（还没有提交），比如没有排除`node_modules`文件夹并执行了`git add -A`操作。如图：

![upload successful](/images/image-20191128182507434.png)

这个时候你想把`node_modules`文件夹下的文件从追踪列表中移除，可以使用一下命令：

```bash
git rm --cached node_modules -r
```

![upload successful](/images/image-20191128182813621.png)

这样`node_modules`里的文件就已经从追踪列表中移除了，这个时候就可以在.gitignore文件中添加node_modules目录了。

## 情景三：使用别名

