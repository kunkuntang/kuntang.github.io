title: Git常用命令
author: Tommi_Max
tags:
  - 备忘翻阅
categories:
  - Git
date: 2019-11-21 14:29:00
---

### 初始化仓库(Init Repository)

```bash
git init
```

### 克隆仓库(Clone Repository)

```bash
git clone [url]
```

### 暂存新文件(Stage New File) & 暂存文件修改(stage modified file)

```bash
git add [file]
```

### 取消暂存文件(Unstage File)

```bash
git rm --cached <file>
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

### 修改Git 仓库地址

```bash
git remote set-url origin [NEW_URL]
```

### 为已有项目添加远程仓库地址

```bash
git remote add origin git@github.com:<username>/<projectName>.git
// username是你在github里注册的用户名，projectName则是你远程仓库的项目名称
git push -u origin master
```

