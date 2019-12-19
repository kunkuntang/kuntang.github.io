title: Git常用命令
author: Tommi_Max
tags:
  - 备忘翻阅
categories:
  - Git
date: 2019-11-21 14:29:00
---
<!-- toc -->

# Git常用命令

## 本地仓库

本地仓库位于自己的电脑里，我们首先在本地编写好代码，然后提交到本地的仓库，但是这些代码仅仅只是在本地，还没推送到服务器上。一开始对于本地仓库的操作也仅仅只是影响本地，只有推送了才应用到服务器仓库。因此在推送到远端之前，可以很大胆地进行各种操作而不用担心会对远程仓库造成什么影响。

### 初始化仓库(Init Repository)

为一个项目初始化为git仓库。

```bash
git init
```

### 克隆仓库(Clone Repository)

把远端的一个仓库下载到本地。

```bash
git clone [url]
```

### 暂存新文件(Stage New File) & 暂存文件修改(Sage Modified File)

把新建的文件或者文件的新修改放到暂存区。

```bash
git add [file]
```

### 取消刚刚新暂存的文件(Unstage New File)

对于未追踪的文件，可以使用一下命令来取消文件暂存：

```bash
git rm --cached <file>
```

### 取消刚刚暂存的文件修改(Unstage Modified File)

对于已经在追踪（之前通过`git add`添加过）的文件，要想取消刚刚暂存的文件修改，可以使用以下命令：

```bash
git reset HEAD <file>
```

### 查看仓库状态（Check Repository Status）

```bash
git status
```

查看状态简览

`git status -s` 或者 `git status --short`

### 查看未暂存的修改(Check Unstaged Modified File)

```bash
git diff
```

此命令比较的是工作目录中当前文件和暂存区域快照之间的差异，也就是修改之后还没有暂存起来的变化内容。

### 查看已暂存的修改(Check Staged Modified File)

```bash
git diff --cached
```

或者

```bash
git diff --staged // Git 1.6.1 及更高版本
```

### 提交更新(Commit File)

在提交之前先要执行`git add `操作把文件放到暂存区。

```bash
git commit
```

这种方式会启动文本编辑器以便输入本次提交的说明。也可以在`git commit`后面带上`-m`参数使得提交信息与命令放在同一行

```bash
git commit -m ""
```

如果想跳过暂存区直接提交更新：

```bash
git commit -a -m ""
```

`-a` 参数会自动执行`git add `步骤，把**所有已经跟踪过的文件暂存起来 **。

### 移除文件

要把文件不纳入仓库中的版本管理中，首先要从已追踪文件列表中移除，然后再提交。

```
git rm <file>
```

执行后会发现本地的文件被删除了，并且把修改放到了暂存区，因此只要把暂存区里的更改提交就完成移除文件的操作。

当要移除被本地修改过的文件，应该使用：

```bash
// 从暂存区里不追踪文件，但是本地保留文件
git rm --cache <file>
```

或者：

```bash
// 从暂存区里不追踪文件并在本地删除文件
git rm -f <file>
```

### 移动文件（删除文件）

要在 Git 中对文件改名，可以这么做: 

```bash
git mv [old fileName] [new fileName]
```

执行后如图所示：

51AE82F6-D51D-43CC-9C1D-919CCEA100E7.png
![upload successful](/images/git_move_file.png)
可以看到Git变更出现了一个R的操作，代表着rename。

`git mv [old fileName] [new fileName]`可以分解成以下三个命令：

```bash
mv [old fileName] [new fileName]
git rm [old fileName]
git add [new fileName]
```

上面三行命令同样可以完成文件的重命名操作。

**注意:在window系统中进行重命名要主要大小写问题，一般来说在window下git对文件大小写不敏感，需要在git全局设置`git config core.ignorecase false`才能检测得到。**

> **Git 并不显式跟踪文件移动操作，因为Git是根据文件内容来推断文件是否有更改的，所以如果在 Git 中重命名了某个文件，仓库中存储的元数据 并不会体现出这是一次改名操作。** 

### 查看提交历史 

使用`git log`命令查看项目仓库的提交历史，默认不用任何参数的话，git log会按提交时间列出所有的更新，最近的更新排在最上面 。

```bash
git log
```

`git log`常用的参数有：

- -p：显示每次提交的内容差异。

- -<n>：仅显示最近n次提交 。

- --stat：查每次提交的简略的统计信息 。

-  --pretty：指定使用不同于默认格式的方式展示提交历史 。

  - --pretty=oneline： 将每个提交放在一行显示 。

  - --pretty=short ：显示提交的简要信息。

  - --pretty=full：显示完全的信息。

  - --pretty=fuller：显示全部的信息。

  - --pretty=format ：可以定制要显示的记录格式（这样的输出对后期提取分析格外有用 ）。

    对于`git log --pretty=format`常用的格式占位符以及其代表的含义如下表：
    

| 选项  | 说明                                   |
| ---- | --------------------------------------|
| %H   | 提交对象(commit)的完整哈希字串            |
| %h   | 提交对象的简短哈希字串                    |
| %T   | 树对象（tree）的完整哈希字串              |
| %t   | 树对象的简短哈希字串                      |
| %P   | 父对象（parent）的完整哈希字串            |
| %p   | 父对象的简短哈希字串                      |
| %an  | 作者（author）的名字                      |
| %ae  | 作者的电子邮箱地址                        |
| %ad  | 作者修订日期(可以用 --date= 选项定制格式) |
| %ar  | 作者修订日期，按多久以前的方式显示        |
| %cn  | 提交者(committer)的名字                   |
| %ce  | 提交者的电子邮件地址                      |
| %cd  | 提交日期                                  |
| %cr  | 提交日期，按多久以前的方式显示            |
| %s   | 提交说明                                  |


    > *作者* 和 *提交者* 之间究竟有何差别？其实作者指的是实际作出修改的人，提交者指的是最后将此工作成果提交到仓库的人。所以，当你为某个项目发布补丁，然后某个核心成员将你的补丁并入项目时，你就是作者，而那个核心成员就是提交者。 

- --graph： 添加一些ASCII字符串来形象地展示你的分支、合并历史。

### 撤销操作

如果在刚刚提交完后马上发现还有东西需要修改，或者漏掉文件需要提交，或者提交信息写错了，可以使用以下命令覆盖上一次的提交：

```bash
git commit --amend
// git commit -m 'do some thing'
// git add README.md
// git commit --amend
```

这个命令会将暂存区中的文件提交，并启动一个文本编辑器，在里面可以看到之前提交的信息，你可以在文本编辑器里修改。编辑后保存会覆盖原来的提交信息。

#### 取消暂存的文件

当你误将一个未完成的文件添加到缓存区是，可以使用下面命令来撤销暂存：

```bash
git reset HEAD <file> ...
// git reset HEAD READMD.md
```

使用该命令后，目标文件将会退回**修改未暂存**的状态。

#### 撤销文件的修改（还原文件）

如果你在编写代码的过程中，突然需要丢弃刚刚所做的更改（比如说产品说这个功能不用做了），那么可以使用下面命令来还原文件：

```
git checkout -- <file>
// git checkout -- README.md
```

> **上面的命令只能对还没放到暂存区里的文件进行还原，如果文件在暂存区，则需要先执行 取消暂存文件 的命令。**

**撤销的修改不能恢复**，如需要恢复建议使用**分支**。

## 远程仓库

远程仓库位于服务器上，是保存我们代码和与其他协作组同步代码的地方。

### 查看远程仓库

```bash
git remote
```

它会列出你指定的每一个远程服务器 的简写。如果你已经克隆了自己的仓库，那么至少应该能看到 origin - 这是 Git 给你克隆的仓库服务器的默认名字。

添加参数`-v`可以显示简写及对于的URL。

> **Git不想传统的CSV，它是分布式的，这也说明了一个仓库可以对应多个远程仓库进行同步。这在某些项目环境下会有应用。**

如果想查看某个远程仓库的详细信息，可以使用下面命令：

```bash
git remote show [remote-name]
// git remote show master
```

### 修改Git 仓库地址

```bash
git remote set-url origin [NEW_URL]
```

### 重命名远程仓库

如果想要重命名引用的名字可以运行git remote rename去修改一个远程仓库的简写名。例如，想要将pb重命名为paul，可以用git remote rename这样做:  

```bash
git remote rename [old-branch-name] [new-branch-name]
// git remote rename pb paul
```

### 移除远程仓库

如果因为一些原因想要移除一个远程仓库 - 你已经从服务器上搬走了或不再想使用某一个特定的镜像了，又或者 某一个贡献者不再贡献了-可以使用git remote rm: 

```bash
git remote rm [branch-name]
```



### 添加远程仓库地址

```bash
git remote add origin git@github.com:<username>/<projectName>.git
// username是你在github里注册的用户名，projectName则是你远程仓库的项目名称
git push -u origin master
```

### 从远程仓库中抓取与拉取

```bash
git fetch [remote-name]
// git fetch origin
```

这个命令会访问远程仓库，从中拉取你还没有的数据（拉取本地还没有的所有分支代码），你也可以指定一个分支名来拉取指定分支代码。

```bash
git fetch [remote-name] [branch-name]
// git fetch origin master
```

但是不管怎么样，这个操作不会把拉取下来的代码自动合并到当前的工作区，而要你手动将其合并到你想要的分支。

如果你使用 clone 命令克隆了一个仓库，命令会自动将其添加为远程仓库并默认以 “origin” 为简写 。这是可以使用`git pull`命令来抓取远程分支的代码并自动合并到当前分支 。

### 推送到远程仓库 

想把本地的提交推送到远程仓库，可以使用下面命令：

```bash
git push [remote-name] [branch-name]
// git push origin master
```

## 标签

Git 可以给历史中的某一个提交打上标签，以示重要。比较有代表性的是人 们会使用这个功能来标记发布结点(v1.0 等等) 。

### 列出标签

使用下面的命令列出所有标签：

```bash
git tag
```

这个命令以字母顺序列出标签，但是它们出现的顺序并不重要。

可以使用`-l`参数来查找特定的标签 ，例如像看1.8.5系列的标签，可以执行：

```
git tag -l 'v1.8.5*'
```

### 创建标签

Git有两种类型的标签：

- 轻量标签（lightweight）
- 附注标签（annotated）

轻量标签就像一个不会改变的分支（它只是一个特定提交的引用）。

附注标签则是存储在Git数据库的一个完整对象，他们是可以被校验的，其中包括打标签者的名字、电子邮件地址、日期时间。并且可以使用GNU Privacy Guard(GPG)签名与验证。

通常建议创建附注标签，这样可以拥有以上所有信息。但如果只是一个临时的标签，或者因为某些原因不想保留那些信息，则可以使用轻量标签。

#### 创建轻量标签

使用git tag即可为commit创建一个标签：

```bash
git tag [tag-name]
```

#### 创建附注标签

在git tag后面添加-a 指定标签名称，-m 指定附加的信息：

```bash
git tag -a [tag-name] -m [message]
// git tag -a v1.4 -m 'annotated tag message'
```

#### 对提交打标签

对于以往的一次提交打标签，可以在最末尾指定提交的commit id：

```bash
// 对以往的提交添加轻量标签
git tag [tag-name] [commit-id]
// 对以往的提交添加附注标签
git tag -a [tag-name] -m [tag-message] [commit-id]
// git tag v1.2 9fcea32
// git tag -a v1.2 -m 'message' 9fcea32
```

#### 推送标签

默认情况下，git push命令并不会传送标签到远程仓库服务器上。在创建完标签后你必须显式地推送标签到共享服务器上。

```bash
git push origin [tagname]
// git push origin v1.3
```

如果想要一次性推送很多标签，也可以使用带有--tags选项的git push命令。这将会把所有不在远程仓库服务器上的标签全部传送到那里。 

```bash
git push origin --tags
```

#### 检出标签 

在 Git 中你并不能真的检出一个标签，因为它们并不能像分支一样来回移动。如果你想要工作目录与仓库中特定的标签版本完全一样，可以使用`git checkout -b [branchname] [tagname]`在特定的标签上创建一个新分支: 

```bash
git checkout -b [branchname] [tagname]
// git checkout -b devCopy v1.3
```

>  **如果在这之后又进行了一次提交，devCopy 分支会因为改动向前移动了，那么 devCopy 分支就会和 v1.3 标签稍微有些不同，这时就应该当心了。 **

###  Git 别名 

Git 并不会在你输入部分命令时自动推断出你想要的命令。如果不想每次都输入完整的 Git 命令，可以通过 git config 文件来轻松地为每一个命令设置一个别名。

```bash
git config --global alias.[name] [command]
// git config --global alias.co checkout
// git config --global alias.br branch
// git config --global alias.ci commit
// git config --global alias.st status
```

这意味着，当要输入`git commit`时，只需要输入 `git ci`即可。

添加一个取消暂存文件的快捷别名：

```bash
git config --global alias.unstage 'reset HEAD --' 
// git unstage fileA 等于 git reset HEAD -- fileA
```

添加一个 last 别名 快速查看最后一次提交：

```bash
git config --global alias.last 'log -1 HEAD'
// git last 等于 git log -1 HEAD
```