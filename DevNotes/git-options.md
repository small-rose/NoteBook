---
layout: default
title: Git Theory
parent: Git
nav_order: 2
---

# Git Theory
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

Git 操作大全
-----------------------------

## git & github 使用秘籍

- [github-cheat-sheet](https://github.com/tiimgreen/github-cheat-sheet/blob/master/README.zh-cn.md)


## Git 完全指南

## 零、速查表

### 1、底层命令速查

| 底层命令速查表：                                             |
| ------------------------------------------------------------ |
| **git对象：**（blob对象）                                    |
| `git hash-object -w file_url`   生成一个key，val存储到`.git/objects` |
| **tree对象：**                                               |
| `git update --add --cacheinfo 100644  hash file_name` ，表示向暂存区添加一条记录，让git对象对应上文件名，并存储到`.git/objects` |
| `git write-tree` ，生成树对象存储到`.git/objects`            |
| **commit对象：**                                             |
| `echo "first commit" | git commit-tree treehash`，生成一个提交对象到`.git/objects` |
| **查询操作**                                                 |
| `git cat-file -p  hash` ，显示hash值对应git对象的内容        |
| `git cat-file -t  hash` ，显示hash值对应git对象的类型        |
| `git ls-files -s `，查看暂存区                               |

### 2、Git命令速查

初始化

```bash
#初始化仓库 
git init  
```

新增或修改操作：

```bash
#查看文件状态
git  status 

#添加文件到暂存区
git  add ./ 

#提交到本地仓库
git  commit -m "msg" 
```

删除或重命名操作

```bash
#删除文件
git  rm  filename
#重命名文件
git  mv  filename1  filename2

#查看文件状态
git  status

#提交到本地仓库
git  commit -m "msg" 
```

查询操作

```bash
#查看文件状态
git  status 

#查看未暂存的修改
git  diff

#查看位提交暂存
git  diff --cache

#查看提交历史记录
git  log  --oneline
```

### 3、分支命令

```bash
# 查看完整的分支图
git log --oneline --decorate --graph --all

# 在当前提交对象上创建新的分支
git branch name

# 在指定提交对象（hash）上创建新的分支
git branch name commitHash

# 切换到name分支
git checkout name  

# 删除空的分支，删除已经合并的分支
git branch -d name

# 强制删分支
git branch -D name

# 存储
git stash 

# 查看存储
git stash list

# 应用存储并删除栈
git stash pop
```









## 一、认识Git

版本控制工具，用来记录文件内容变化的。

版本控制分类：

- 集中式版本控制系统，如`CVS`，`SVN`等都有一个单一集中管理的服务器，保存所有文件的修订版本，每个人都可以看到所有代码，管理员统一管理所有用户相关权限，项目管理方便，但是存在单点故障风险，如果服务器宕机，就无法提交代码协同工作，如果磁盘损坏，会丢失所有历史更新记录。

- 分布式版本控制系统，如`Git`，`BitKeeper` 等，客户端并不只是提取最新版版本的文件快照，而是会将代码仓库完整的镜像下来，所需空间相对大一点点。相当于每台电脑都是服务器。每次操作都是对代码仓库完整备份。分布式版本控制系统在管理项目时，存放的不是项目与版本之间的差异，它存的是索引，占用磁盘空间少，每个客户端都有历史记录。如果某个服务器宕机，不影响本地开发，不丢失历史记录。

GIT优点：

​		完全分布式

缺点：

​		命令学习掌握



## 二、Git 安装

### 1、安装验证

下载：https://git-scm.com/downloads

安装完成之后验证

```bash
git --version
```

Git 操作配置的命令：

```bash
git  config 
```

一般配置有三大类：

- **--system** ：系统中对所有用户都普遍适用的配置。使用`git config --system` 命令会修改`/etc/gitconfig`文件。
- **--global** ：用户目录下的配置文件只适用于该用户。使用`git config --global` 读写的是`~/.gitconfig` 文件
- 不写参数，表示堆当前项目的git目录进行配置，使用`git config` 可以直接针对当前项目配置，即工作目录下的`.git/config`文件

优先级别以就近原则为准。

`.git/config`  > `~/.gitconfig`  >  `/etc/gitconfig`



### 2、初始化配置

安装完成后初始化配置用户名和邮件地址：

```bash
git config --global user.name "xiaocai"
git config --global user.email "small-rose@qq.com"
```
查看已有配置：
```bash
git  config --list
```



## 三、目录结构

初始化工作区命令：

```bash
git init
```

初始化之后，有一个`.git`隐藏文件，里面的目录结构大概如下：

```
hooks/
ingfo/
objects/
refs/
config
description
HEAD
```

| 文件夹                | 作用                                       |
| --------------------- | ------------------------------------------ |
| hooks/                | 目录包含服务端或客户端的钩子脚本           |
| info/                 | 包含一个全局性排除文件                     |
| logs/                 | 保存日志信息                               |
| <mark>objects/</mark> | 目录存储所有数据内容，重要                 |
| <mark>refs/</mark>    | 目录存储指向数据提交对象的指针，分支，重要 |
| <mark>config</mark>   | 文件包含项目特有的配置选项，重要           |
| description           | 用来显示对仓库的描述信息                   |
| HEAD                  | 文件指示目前被检出的分支，重要             |
| index                 | 文件保存暂存区信息，重要                   |



## 四、Git 原理

三个区域：

- 工作区：沙箱环境 git不会管理 随便更改操作
- 暂存区：记录文件的操作
- 版本库：最终的代码实现提交到这里 .git目录就是版本库

![](/images/git/git-01.jpg)

三个对象

- Git对象
- 树对象
- 提交树

### 4.1 Git 对象

**Git 对象**

Git对象是Git的核心部分，是一个简单的键值对数据库。可以像该数据库插入任意类型的内容，它会返回一个键值，通过键值可以再次检索内容。

（1）Git 存储

向数据库写入内容并返回对应键值：

```bash
echo "xiaocai“ | git hash-object -w --stdin
```

**`-w`** ：指示 hash-object 命令存储数据对象，如果不指定该参数，则命令仅返回对应键值，不存储数据。

**`--stdin`**：standard input 的缩写，指示该命令从标准输入读取内容，如果不知道该参数，则需要在目录尾部给你待存储文件的路径。

如：

```bash
# 返回对应文件的键值
echo "xiaocai" | git hash-object d:/objects 
a88713553b066c421f74fabac048560b3209c3a8

# 存储文件到/data/.git/objects传递
echo "xiaocai" | git hash-object -w  /data/.git/objects 
a88713553b066c421f74fabac048560b3209c3a8
```

（2）查看Git 存储结构

```bash
$ find .git/objects/ -type f
.git/objects/a8/8713553b066c421f74fabac048560b3209c3a8
```

 Git的存储，一个文件对应一条内容，校验前两个字符，用于命名子目录，余下的38个字符用作文件名。

（3）根据键值拉取数据

```bash
# 返回对应文件的内容
$ git cat-file -p a88713553b066c421f74fabac048560b3209c3a8
xiaocai
# 返回对应文件内容的类型
$ git cat-file -t a88713553b066c421f74fabac048560b3209c3a8
blob
```

**`-p`**：该参数可以指示该命令自动判断内容的类型，并显示格式友好的内容。

**`-t`**：该参数可以返回该内容的类型，一般是`blob`类型

（4）对一个文件进行简单的版本控制

创建一个新文件并将其内容存入数据库

```bash
# 创建新文件 xctest.txt
echo "version 1" > xctest.txt 
# 将文件存入数据库
git hash-object -w xctest.txt
83baae61804e65cc73a7201a7252750c76066a30 #返回值
# 查看文件内容
git cat-file -p 83baae61804e65cc73a7201a7252750c76066a30
vsersion 1
```

修改文件内容：

修改1，将原来版本内容替换了

```bash
# 修改原有内容
echo "version 2" > xctest.txt 
# 保存数据
git hash-object -w xctest.txt
1f7a7a472abf3dd9643fd615f6da379c4acb3e3a

# 查看文件内容
git cat-file -p 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a
vsersion 2
```

追加文件内容：

```bash
# 追加内容
echo "version 23" >> xctest.txt 
# 保存到Git数据
git hash-object -w xctest.txt
# 查看文件内容
$ git cat-file -p 0e80f6a3d33fed2e3c7d0b0e0f1eb9edaa5ff4e9
version 2
version 23
# 查看历史版本内容
git cat-file -p 83baae61804e65cc73a7201a7252750c76066a30
vsersion 1
```

每次存储只有一个文件的版本，不是整个项目版本，Git对象没有保存文件名，直接保存到本地版本库。

Git 对象主要作用是用来存储数据内容，针对是单个文件的版本快照。

### 4.2 树对象

**树对象**

树对象（Tree Object）解决文件名保存问题，一般项目会将多个文件组织到一起，Git 就是一种类似Unix文件系统的方式存储内容，所有内容均以树对象和数据对象（Git 对象）的形式存储，其中树对象对应了Unix的目录项，数据对象（Git 对象）对应文件内容，一个树对象包含一条或多条记录，每条记录含有一个指向Git 对象或子树对象的`SHA-1`指针，以及相应的模式、类型、文件名信息。一个树对象也可以包含另一个树对象。

> 简单的理解就是针对整个项目（多个文件的）版本快照

（1）查看树对象

```bash
git cat-file -p master^{tree}(或者是树对象的hash) 
```

master^{tree} 表示master分支上最新的提交锁指向的树对象。

（2）构建树对象

```bash
update-index
write-tree
read-tree
```

使用`update-index` 创建暂存区，并使用`write-tree` 生成树对象：

```bash
# 创建暂存区
git update-index --add --cacheinfo 100644\
	83baae61804e65cc73a7201a7252750c76066a30 xctest.txt
```

- 文件模式为100644 表示这是一个普通文件
- 文件模式为100755 表示这是一个可执行文件
- 文件模式为120000 表示这是一个符号连接
- `--add` 因为此前该文件并没有在暂存区中 首次要加add
- `--cacheinfo` 因为要添加的文件在git数据库中,没有位于当前目录下，参数表示从数据库拿文件。

**注意：**

`update-index` 没有向本地版本库操作，只是在暂存区里生成了一条记录。

（3）查看暂存区命令

```bash
git ls-files -s
```

此时执行查看暂存区，里面只有一个文件

```bash
# 查看暂存区
git ls-files -s
100644 83baae61804e65cc73a7201a7252750c76066a30 0       xctest.txt
```

Git对象（数据库）里：

```bash
$ find .git/objects/ -type f
.git/objects/83/baae61804e65cc73a7201a7252750c76066a30
```

（4）使用`write-tree` 生成树对象：

```bash
# 生成树对象，会保存到数据库
git write-tree	
6e545fd2bd8f012c7c236e7818cabac6b36d9c2a
```

再次查看数据库中发现有两个对象：

```bash
$ find .git/objects/ -type f
.git/objects/6e/545fd2bd8f012c7c236e7818cabac6b36d9c2a
.git/objects/83/baae61804e65cc73a7201a7252750c76066a30
```

即：**`write-tree` 生成树对象时，会将暂存区现有的内容生成一份快照（即项目快照、项目版本），并存储到Git数据库中，并且不会清空暂存区。**

（5）如果后续再次向暂存区操作

```bash
# 创建新文件
echo "new v1" > new.txt

# 存入到数据库
$ git hash-object -w new.txt
eae614245cc5faa121ed130b4eba7f9afbcc7cd9
```

此时数据库应该有3个对象：

```bash
# 此时数据库有三个对象了
find .git/objects/ -type f
.git/objects/6e/545fd2bd8f012c7c236e7818cabac6b36d9c2a  # 树对象，项目第一个版本
.git/objects/83/baae61804e65cc73a7201a7252750c76066a30  # xctest.txt，第一个版本
.git/objects/ea/e614245cc5faa121ed130b4eba7f9afbcc7cd9  # new.txt，第一个版本
```

（6）再次修改`xctest.txt`

```bash
echo "xiaocai v2" >> xctest.txt
```

此时需要重新生成hash，保存到数据库

```bash
$ git hash-object -w xctest.txt
0b6ca5d6025d73d7d62d3c90555add1e8662ab91
```

再次查看数据库：

```bash
find .git/objects/ -type f
.git/objects/0b/6ca5d6025d73d7d62d3c90555add1e8662ab91  # xctest.txt的第2个版本(git对象)
.git/objects/6e/545fd2bd8f012c7c236e7818cabac6b36d9c2a  # 项目第1个版本（树对象）
.git/objects/83/baae61804e65cc73a7201a7252750c76066a30  # xctest.txt第1个版本(git对象)
.git/objects/ea/e614245cc5faa121ed130b4eba7f9afbcc7cd9  # new.txt第1个版本(git对象)
```

看看暂存区现有的：

```bash
$ git ls-files -s
100644 83baae61804e65cc73a7201a7252750c76066a30 0       xctest.txt
```

（7）重新加入暂存区

因为`xctest.txt`文件修改了，需要将第二个版本的hash 重新加入暂存区

```bash
# 加入暂存区
git update-index --add --cacheinfo 100644 0b6ca5d6025d73d7d62d3c90555add1e8662ab91 xctest.txt
```

再次查看暂存区发现 `xctest.txt` 的hash 重新计算了。

**暂存区中，文件名字相同，只改变文件的内容，就会重新生成一个hash**

```bash
# 再次查询暂存区
git ls-files -s
100644 0b6ca5d6025d73d7d62d3c90555add1e8662ab91 0       xctest.txt
```

新建的文件 `new.txt` 也要加入到暂存区：

```bash
git update-index --add --cacheinfo 100644 eae614245cc5faa121ed130b4eba7f9afbcc7cd9 new.txt
```

此时暂存区情况：

```bash
$ git ls-files -s
100644 eae614245cc5faa121ed130b4eba7f9afbcc7cd9 0       new.txt
100644 0b6ca5d6025d73d7d62d3c90555add1e8662ab91 0       xctest.txt
```

**暂存区中，文件名字不相同，就会新增一条新的记录**

此时暂存区内容和工作去内容一致，再次执行

```bash
git write-tree
f1fa781509c015b1a1f4c7757cb7e1848d032f6d
```

生成一个新的树对象，即生成了项目第二个版本快照：

```bash
find .git/objects/ -type f
.git/objects/0b/6ca5d6025d73d7d62d3c90555add1e8662ab91  # xctest.txt的第2个版本(git对象)
.git/objects/6e/545fd2bd8f012c7c236e7818cabac6b36d9c2a  # 项目第1个版本（树对象）
.git/objects/83/baae61804e65cc73a7201a7252750c76066a30  # xctest.txt第1个版本(git对象)
.git/objects/ea/e614245cc5faa121ed130b4eba7f9afbcc7cd9  # new.txt第1个版本(git对象)
.git/objects/f1/fa781509c015b1a1f4c7757cb7e1848d032f6d  # 项目第2个版本（树对象）
```

**树对象**

此时此刻，执行到这里，数据库里已经有了2个项目版本，项目第1个版本指向`xctest.txt`第1个版本，项目第2个版本指向`new.txt`的第1个版本和`xctest.txt`的第2个版本。

将第一个树对象加入到第二个树对象，使其成为新的树对象

```bash
# 将一个树对象读入到暂存区
git read-tree --prefix=bak 6e545fd2bd8f012c7c236e7818cabac6b36d9c2a
```

查看暂存区：

```bash
$ git ls-files -s
100644 83baae61804e65cc73a7201a7252750c76066a30 0       bak/xctest.txt
100644 eae614245cc5faa121ed130b4eba7f9afbcc7cd9 0       new.txt
100644 0b6ca5d6025d73d7d62d3c90555add1e8662ab91 0       xctest.txt
```

再次生成新的树对象：

```bash
git write-tree
8ff5b56ab67cbb0336873021cf833b76eafa4067
```

查看类型：

```bash
git cat-file -p 8ff5b56ab67cbb0336873021cf833b76eafa4067
040000 tree 6e545fd2bd8f012c7c236e7818cabac6b36d9c2a    bak
100644 blob eae614245cc5faa121ed130b4eba7f9afbcc7cd9    new.txt
100644 blob 0b6ca5d6025d73d7d62d3c90555add1e8662ab91    xctest.txt
```

此时的数据库里：

```bash
$ find .git/objects/ -type f
.git/objects/0b/6ca5d6025d73d7d62d3c90555add1e8662ab91  # xctest.txt的第2个版本(git对象)
.git/objects/6e/545fd2bd8f012c7c236e7818cabac6b36d9c2a  # 项目第1个版本（树对象）
.git/objects/83/baae61804e65cc73a7201a7252750c76066a30  # xctest.txt第1个版本(git对象)
.git/objects/ea/e614245cc5faa121ed130b4eba7f9afbcc7cd9  # new.txt第1个版本(git对象)
.git/objects/8f/f5b56ab67cbb0336873021cf833b76eafa4067  # 包含项目第1个版本和第2个版本的树对象
.git/objects/f1/fa781509c015b1a1f4c7757cb7e1848d032f6d  # 项目第2个版本（树对象）
```

整个结构为：

第1个版本：

```txt
tree1 -----> xctest.txt v1	
```

第2个版本：

```txt
           +------> tree1 -----> xctest.txt v1
           |
tree2 -----+------> xctest.txt v2
           |
           +------> new.txt v1
```

那么可以推测，第3个版本： 

```txt
                         +------> tree1 -----> xctest.txt v1
                         |
      +-----> tree2 -----+------> xctest.txt v2
      |                  |
      |                  +------> new.txt v1
tree3 +                    
      |
      +------ xctest.txt v3
```

问题：无法区分版本做了什么，没有注释。

### 4.3 提交对象

可以通过commit-tree 命令创建一个提交对象，为此需要指定一个树对象的`SHA-1`值，以及该提交的父提交对象，第一次将暂存区做快照就没有父对象。

**创建提交对象**

```bash
# 返回提交对象的hash
echo "first commit" | git commit-tree <hash>
```

**查看提交对象**

```bash
git cat-file -p <hash>
```



示例：提交刚才的第一个项目版本

```bash
echo "first commit" | git commit-tree 6e545fd2bd8f012c7c236e7818cabac6b36d9c2a
6d8d40d59a48e33fd334ebabffe3e8cb89308f21
```

查看这个hash的信息：

```bash
#查看类型：
git cat-file -t 6d8d40d59a48e33fd334ebabffe3e8cb89308f21
commit

# 查看信息
$ git cat-file -p 6d8d40d59a48e33fd334ebabffe3e8cb89308f21
tree 6e545fd2bd8f012c7c236e7818cabac6b36d9c2a
author small-rose <small-rose@qq.com> 1605513658 +0800
committer small-rose <small-rose@qq.com> 1605513658 +0800

first commit
```

再提交项目的第二个版本

```bash
# 再次提交
echo "second commit" | git commit-tree f1fa781509c015b1a1f4c7757cb7e1848d032f6d -p 6d8d40d59a48e33fd334ebabffe3e8cb89308f21
# 返回了当前提交对象hash
ca6961bbf978608e602b060d6c4a7a79cb0041c5

```

此时的`-p` 表示指定当前提交对象的父对象hash。

再次查看当前对象：

```bash
git cat-file -p ca6961bbf978608e602b060d6c4a7a79cb0041c5
tree f1fa781509c015b1a1f4c7757cb7e1848d032f6d
parent 6d8d40d59a48e33fd334ebabffe3e8cb89308f21
author small-rose <small-rose@qq.com> 1605514000 +0800
committer small-rose <small-rose@qq.com> 1605514000 +0800

second commit
```

里面还包含了树对象，和父级提交对象，提交信息，注释。

![](/images/git//git-demo-01.png)

Git的版本控制中，真正代表项目版本的是提交对象，而项目版本快照的真正本质是树对象。提交对象是对树对象的一层封装，并且提交对象是链式的，可以将版本进行串起来，Git的每次版本提交都不是增量，而是快照。

如果要进行跳跃版本切换，可以直接切换对应提交对象的hash。

**一次完整的项目提交 包括至少一个提交对象 一个树对象 0或多个git对象**

## 五、Git 基础命令

### 1、git init

**初始化工作空间**

初始化工作目录命令格式：

```bash
git init
```

生成 `.git` 目录，所有git需要的数据和资源都存在在这个目录。

### 2、git add

**跟踪已修改文件到暂存区：**

跟踪一个已修改文件到暂存区的命令格式：

```bash
git add ./
```

git add 命令将修改的文件生成git对象，加入暂存区。

过程：将将修改的文件生成成git对象，放入版本库，再将Git对象加入到暂存区，只是没有生成树对象。在这个过程中，生成Git对象是增量式的。

相当于执行了N次（N个文件）：

```
git hash-object -w 
git update-index
```

### 3、git status

**跟踪文件状态：**

```bash
git status [指定的文件]
```

status ：

- `untracked`：未跟踪，红色

- `tracked` ：已跟踪。

	- `unmodified`：未修改，已提交，一般不列出显示。
- `modified`：已修改，红色
	- `staged` ： 已暂存，绿色


**跟踪新文件：**

`git add` 命令执行之后使用 `git status`查看，出现`changes to be committed` 表示已经暂存。

**暂存已修改文件：**

已经暂存的文件进行再次修改，使用 `git status`查看，此时会出现

`changes to be committed` 表示该文件之前暂存区有一份，表示已暂存；同时也会出现

`changes not staged for commit` 表示改文件又有了新的修改。此时已修改文件的状态为`modified`；修改之后的git对象还没有生成。如果`git add` 重新暂存，在暂存区则会进行覆盖操作，并重新生成git对象的hash。

### 4、git diff

**查看已暂存和未暂存的更新：**

`git status` 仅仅列出修改过的文件。

- 判断当前做的哪些更新还没有做暂存：

```bash
git diff
```

- 判断哪些更新已经暂存准备好了下次提交

```bash
git diff --cached

# 1.6 以上
git diff --staged
```

示例：查看哪些暂存还没提交的数据，这是之前操作的数据。

```bash
git diff --cached
diff --git a/bak/xctest.txt b/bak/xctest.txt
new file mode 100644
index 0000000..83baae6
--- /dev/null
+++ b/bak/xctest.txt
@@ -0,0 +1 @@
+version 1
diff --git a/new.txt b/new.txt
new file mode 100644
index 0000000..eae6142
--- /dev/null
+++ b/new.txt
@@ -0,0 +1 @@
+new v1
diff --git a/xctest.txt b/xctest.txt
new file mode 100644
index 0000000..0b6ca5d
--- /dev/null
+++ b/xctest.txt
@@ -0,0 +1,2 @@
+version 1
+xiaocai v2
```

### 5、git commit

**提交文件：**

```bash
git commit 
```

没有参数会进入一个注释文件可以写大段注释。

```bash
 git commit -m " messgae info"
```

`-m` 一般写短小文字较少的注释。注释建议，带上关键信息，如完成进度，fix bug 

2条命令都是提交项目版本到本地库，生成树对象和提交对象。

相当于执行：

```bash
git write-tree
git commit-tree
```

**跳过使用暂存区域：**

```bash
git commit -a -m "xiaocai test"
```

通过`-a` 参数，git可以自动把所有已经跟踪过的文件暂存起来一并提交，从而跳过`git add`步骤。

注意：使用 -a 的前提是文件状态要已经被跟踪。

### 6、git  rm

**移除文件：**

从Git 中移除文件，就必须要从已跟踪文件清单中注册删除，其实就是从暂存区注册删除，然后提交。可以使用以下命令完成：

```bash
git rm
```

该命令将把文件从暂存区注册删除，并且同时从工作目录删除对应的文件，这样文件就不会出现在未跟踪文件清单中。

需要注意的是，删除之后进行`git add` 和 `git commit`操作，对应的Git 对象永远不会删除，暂存区删除之后，版本库里进行的是新增操作，新增的是一个没有内容的git对象和一个树对象。如果要找回，可以找到对应的提交对象hash，回退即可。

如果我们先手工删除了文件，可以执行`git rm` 即可，相当于执行了`git add ./` 和 `git commit` 。也可以手工执行

```bash
git add ./
git commit` 
```

`git rm` 其实就是删除工作目录中对应的文件，再将修改添加到暂存区。

### 5、git  mv

**文件改名：**

git文件修改文件名称命令

```bash
# 重命名操作
git mv oldfile.suffix1  newfile.suffix2
```

使用，新建一个文件然后提交：

```bash
# 新建xiaocai.txt
echo "xiaocai de wen jian" > xiaocai.txt

#git add
git commit -a -m "new a file test"

# 再执行重命名操作
git mv xiaocai.txt  xc.txt
```

`git rm `  在旧的版本中类似相当于执行了三条命令：

```bash
mv xiaocai.txt  xc.txt
git rm  xiaocai.txt
git add  xiaocai.txt
```

我的git版本比较新，新的版本 status 显示的是 renamed，暂时没注意过程，后续清楚了再补上。**`TODO`**

查看状态，此时是renamed状态，属于修改操作，

```bash
git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        renamed:    xiaocai.txt -> xc.txt
```

**`git mv file1  file2`** 其实就是将工作目录中的文件进行重命名，再将修改添加到暂存区。

### 8、git log

**查看历史记录：**

在提交很多更新之后，想回顾查看提交历史，或者回退历史版本时，使用该命令。

```bash
git log
```

没有参数会按提交时间列出所有更新，最近在上面，倒序排列。

示例：

```bash
$ git log
commit c32a601099e6ca73b910829856bc4b1ba88c014e (HEAD -> master)
Author: small-rose <small-rose@qq.com>
Date:   Mon Nov 16 19:57:12 2020 +0800

    rename xiaocai.txt to xc.txt

commit d57678e00b1dea5ce92817f5ca495c2dc852496c
Author: small-rose <small-rose@qq.com>
Date:   Mon Nov 16 19:54:07 2020 +0800

    new a file test

commit 4e84326e84545905f106975a4ce32eb520b4bc98
Author: small-rose <small-rose@qq.com>
Date:   Mon Nov 16 19:46:44 2020 +0800

    first commit
```

按下箭头翻页， 按 `q` 退出。如果要回退，利用提交对象的hash即可。

不方便看还可以进行格式化显示：

```bash
git log --pretty=oneline
git log --oneline
```

效果：

```bash
$ git log --pretty=oneline
c32a601099e6ca73b910829856bc4b1ba88c014e (HEAD -> master) rename xiaocai.txt to xc.txt
d57678e00b1dea5ce92817f5ca495c2dc852496c new a file test
4e84326e84545905f106975a4ce32eb520b4bc98 first commit


$ git log --oneline
c32a601 (HEAD -> master) rename xiaocai.txt to xc.txt
d57678e new a file test
4e84326 first commit
```



## 六、Git 分支

Git 分支模型高效轻量。Git 亮点技能。

分支就是一个提交对象前面的指针，每次提交完成，指针就在提交对象的前面指向最新提交。

### 1、创建分支

基础命令

```
git branch
```

没有参数时，显示分支列表。

后面带参数时，表示创建分支命令：

```bash
git branch  dev
```

执行之后，查看日志：

```bash
git log --oneline
c32a601 (HEAD -> master, dev) rename xiaocai.txt to xc.txt
d57678e new a file test
4e84326 first commit

```

新建新的分支并切换到该分支上（一步到位式）

```bash
git checkout -b  dev_test
```

执行结果：

```bash
$ git checkout -b  dev_test
Switched to a new branch 'dev_test'
```

再查看分支图：

```bash
$ git log --oneline --decorate --graph --all
* 37c967e (dev) add dev code
* c32a601 (HEAD -> dev_test, master) rename xiaocai.txt to xc.txt
* d57678e new a file test
* 4e84326 (first) first commit

```

可以看到`HEAD` 已经指向了`dev_test`，并且创建新的`dev_test`分支是基于`master`分支创建的，表示创建并切换分支成功。



### 2、切换分支

```bash
git checkout dev
```

切换之后，可以看到路径变化：`~/Desktop/test (dev)`

执行之后，查看日志：

```bash
git log --oneline
c32a601 (HEAD -> dev, master) rename xiaocai.txt to xc.txt
d57678e new a file test
4e84326 first commit
```

在新的`dev` 分支进行操作开发

```bash
echo "dev code " > code.java

git add ./

git commit -m "add dev code "
```

执行之后，查看日志：

```bash
git log --oneline
37c967e (HEAD -> dev) add dev code
c32a601 (master) rename xiaocai.txt to xc.txt
d57678e new a file test
4e84326 first commit
```

此时 HEAD 指针跟着`dev` 分支前进了。

<font color='red'><b>**注意**</b></font>：**分支切换会改变工作目录中的文件，在切换分支时，一定要注意你工作目录里的文件会被改变。如果是切换到一个比较旧的分支，工作目录会恢复到该分支最后一次提交时的样子。如果Git不能完整这个切换工作，它将禁止切换分支。**

最佳实践注意事项：

（1）每次切换分支前，当前分支一定是干净的（所有文件都是已提交状态）。所以在切换分支前使用`git status` 命令验证状态。

（2）问题发生于在切换分支时，如果当前分支上有未暂存的修改（一般是第一次）或者有未提交的暂存（一般是第一次），分支可以切换成功，但是会对其他分支造成污染。

### 3、删除分支

删除之前一定要先切换分支

```bash
git checkout master
```

切换成功后，显示master分支

```bash
git log --oneline
c32a601 (HEAD -> master) rename xiaocai.txt to xc.txt
d57678e new a file test
4e84326 first commit
```

删除分支命令：

```bash
git branch -d name
```

### 4、其他分支相关

查看每个分支最后一次提交：

```
git branch -v
```

-----

新建一个分支并且使分支指向对应的提交对象：【时光机，版本穿梭】

```bash
git branch  name commitHash
```

示例：

```bash
# 当前是master分支
git log --oneline
c32a601 (HEAD -> master) rename xiaocai.txt to xc.txt
d57678e new a file test
4e84326 first commit
```

现在创建一个分支想回第一次提交的时候看看代码怎么写的：

```bash
git branch  first 4e84326

# 新的分支出现了
git log --oneline
c32a601 (HEAD -> master) rename xiaocai.txt to xc.txt
d57678e new a file test
4e84326 (first) first commit
```

此时新的first分支语句出现了，可以切换过去看看：

```bash
# 切换分支
git checkout first
Switched to branch 'first'

# 现在进入了 first分支
git log --oneline
4e84326 (HEAD -> first) first commit
```

-----

查看哪些分支已经合并到当前分支：

```bash
git branch  -merged
```

在这个列表中分支名字前面没有`*`号的分支通常可以使用`git branch -d` 删除掉。

----

查看所有包含未合并工作的分支：

```bash
git branch --no-merged
```

尝试使用`git branch -d` 删除在这个列表中的分支时会失败。

如果真的确定要删除分支，可以使用`git branch -D` 进行强制删除。

----

**分支的本质是一个提交对象，HEAD 是一个指针，它默认指向master，切换分支时，其实就是让HEAD指向不同的分支。每次有新的提交时，HEAD都会带着当前指向的分支一起往前移动。**

### 5、撤销与重置

**撤销命令**

```bash
git commit -amend
```

该命令将暂存区的文件体积，如果上次提交以来你还未做任何修改，在杀你提交后马上执行此命令，那么快照会保持不变，而你锁修改的只是提交信息。

如果提交后发现忘记了暂存某些需要的修改，可以像下面这样操作：

```bash
git commit -m 'some desc'
git add forgeotten_file
git commit -amend
```

最终只会有一个提交，第二次提交将代替第一次提交的结果。



**重置命令**

```
git reset HEAD  文件名
```



### 6、配置别名

**配置别名**

Git 没有自动推断命令功能，有些命令比较长，不想每次输入完整的命令，可以通过`git config`文件来轻松为每个命令设置一个别名。

```
git config --global alias.co  checkout
git config --global alias.br  branch
git config --global alias.cm  commit
git config --global alias.st  status
```

如果需要执行`git commit` 只需要输入 `git cm` 即可。

对于复杂命令，比如查看完整的分支图的命令：

```bash
git log --oneline --decorate --graph --all
```

执行结果：

```bash
git log --oneline --decorate --graph --all
* 37c967e (dev) add dev code
* c32a601 (master) rename xiaocai.txt to xc.txt
* d57678e new a file test
* 4e84326 (HEAD -> first) first commit

```

将该命令配置别名时需要带上双引号：

```
git config --global alias.xiaocai  "log --oneline --decorate --graph --all"
```

### 7、案例实操

工作流程：

（1）正在开发某个系统，现有某个新的需求需要创建一个分支。

（2）在这个分支上进行功能开发。

（3）领导说线上系统出现严重问题，需要紧急修复，需要按照下列流程操作：

（4）切换到线上分支 prod（prod在演示中为master）

（5）为紧急修改的问题创建一个修复分支，并在其中修复及测试。

（6）测试完成后，切换回线上分支prod，然后合并这个修复分支，最后将改动推送到线上分支prod。

（7）继续回到新需求的分支，进行开发工作。

操作：

1、确认当前工作的分支是否干净？

**注意，切换分支一定要先确认分支是否干净**

```
$ git status
On branch dev_test
nothing to commit, working tree clean
```

如果不干净，就需要先提交当前分支，再次确认是否干净。

如果干净，就可以切换分支。

2、创建新的需求的分支，加入新的需求代号为fun008

```bash
# 创建一个新需求的分支
git checkout -b  fun008

# 然后开开心心写功能，这里就模拟建一个文件好了
echo 'fun008 v1' > fun008.txt
```

不巧了，领导说线上系统出现严重问题，在issue709，现在需要紧急修复一下，赶紧搞一下。好了现在不能愉快地继续新需求了，咋办。

3、检查当前分支是否干净

**注意，切换分支一定要先确认分支是否干净**

```bash
$ git status
On branch fun008
Untracked files:
  (use "git add <file>..." to include in what will be committed)

        fun008.txt

nothing added to commit but untracked files present (use "git add" to track)

```

我靠，才建的文件，还没提交，干净提交一波，不然切换的分支会被污染了。

```bash
git add ./
git commit -m 'fun008 的功能'
$ git status
On branch fun008
nothing to commit, working tree clean
```

好了，终于天下安定了，可以去开分支修复bug了。

4、创建修复问题的分支，修复bug

```bash
# 先回到主分支，不要乱改
$ git checkout master

# 然后看看现在的分支结构图
$ git log --oneline --decorate --graph --all
* 9b4d966 (fun008) fun008 的功能
| * 37c967e (dev) add dev code
|/
* c32a601 (HEAD -> master, dev_test) rename xiaocai.txt to xc.txt
* d57678e new a file test
* 4e84326 (first) first commit

```

没有问题之后，再以主分支为基点，创建bug分支

```bash
# 创建一个修复bug的分支 bug709 或者 issue709
git checkout -b  bug709
```

进行修改bug操作，此处就以修改`new.txt`文件模拟修改bug。

```bash
$ dir
new.txt  xc.txt  xctest.txt

# 修改bug
 ~/Desktop/test (bug709)
$ vi new.txt

# 添加修改状态到暂存
 ~/Desktop/test (bug709)
$ git add ./

# 提交修复
 ~/Desktop/test (bug709)
$ git commit -m 'fix bug709'
[bug709 659454d] fix bug709
 1 file changed, 3 insertions(+)

```

此时各个分支状态：

```bash
$ git log --oneline --decorate --graph --all
* 659454d (HEAD -> bug709) fix bug709
| * 9b4d966 (fun008) fun008 的功能
|/
| * 37c967e (dev) add dev code
|/
* c32a601 (master, dev_test) rename xiaocai.txt to xc.txt
* d57678e new a file test
* 4e84326 (first) first commit

```

可以看到 `bug709`分支 和 `fun008` 分支是并行的。现在bug经过测试也修复好了，那么需要将  `bug709`分支 bug修复的代码合并到`master`分支上来。

5、修复bug完成了，需要再次回到prod/ master 分支上来

**注意，切换分支一定要先确认分支是否干净**

```bash
$ git status
On branch bug709
nothing to commit, working tree clean

```

确认分支干净之后，可以切换回主分支：

```bash
# 切换到主分支
$ git checkout master

# 查看HEAD的指向
$ git log --oneline --decorate --graph --all
* 659454d (bug709) fix bug709
| * 9b4d966 (fun008) fun008 的功能
|/
| * 37c967e (dev) add dev code
|/
* c32a601 (HEAD -> master, dev_test) rename xiaocai.txt to xc.txt
* d57678e new a file test
* 4e84326 (first) first commit

```

进行合并操作：

在合并之前可以验证一下，当前的`new.txt` 文件内容

```bash
$ cat new.txt
new v1
```

执行合并：

```bash
$ git merge bug709
Updating c32a601..659454d
Fast-forward
 new.txt | 3 +++
 1 file changed, 3 insertions(+)

```

`Fast-forward` 表示快进式合并，快进式合并不会产生冲突。

再次验证，当前的`new.txt` 文件内容，有在修复bug的操作了：

```bash
$ cat new.txt
new v1

change bug code v2
```

看看分支图：

```bash
$ git log --oneline --decorate --graph --all
* 659454d (HEAD -> master, bug709) fix bug709
| * 9b4d966 (fun008) fun008 的功能
|/
| * 37c967e (dev) add dev code
|/
* c32a601 (dev_test) rename xiaocai.txt to xc.txt
* d57678e new a file test
* 4e84326 (first) first commit

```

可以看到`master`分支直接跳过了`fun008`分支，快进到了`bug709` 的分支，此时bug成功修复了，但是fun008的分支却是过期的，并且该分支是基于之前的`maser`分支创建，分支落后于现在`bug709`分支，即之前让紧急修复bug的问题在fun008分支依然还在。

而且此时`bug709` 已经合并到`master`，现在bug已经修复可以删除掉了：

```bash
$ git branch -d bug709
Deleted branch bug709 (was 659454d).

```

master 分支完美了，现在把master推送到远程仓库即可。

```bash
git push 
```

8、回到`fun008`分支，继续开发需求功能。

**注意，切换分支一定要先确认分支是否干净**

```bash
$ git status
On branch master
nothing to commit, working tree clean

```

确认完毕，可以切换到工作的分支

```bash
git checkout fun008
```

继续愉快的完成工作

```bash
# 然后开开心心写功能，这里就模拟修改文件好了，注意 new.txt 在修复bug的时候master分支也有改动过，这里再改一次，合并的时候肯定会有冲突
echo 'fun008-code v2' >> fun008.txt

echo 'fun008-code v2' >> fun008.txt

echo 'fun008-code v2' >> new.txt
```

经过一阵捣鼓，fun008分支终于开发完了。

可以提交代码了：

```bash
git add ./
git commit -m "fun008 commit finish 100% "
```

提交完成，再切换分支

8、回到主分支，合并新开发的功能。

**注意，切换分支一定要先确认分支是否干净**

有人会说你上面不是已经提交了吗？是的，但是在实际开发过程中，开发时间线拉长，各种事情交错，分支众多，根本记不住到底自己这个分支到底提交过没有啊，所以好习惯要保持，每次切换分支时，一定要先看看当前分支是否干净。

```bash
$ git status
On branch fun008
nothing to commit, working tree clean
```

确认干净了，可以切换分支了：

```bash
$ git checkout master
Switched to branch 'master'

```

然后看看现在的分支状态：

```bash
$ git log --oneline --decorate --graph --all
* aa5e554 (fun008) fun008 commit finish
* 9b4d966 fun008 的功能
| * 659454d (HEAD -> master) fix bug709
|/
| * 37c967e (dev) add dev code
|/
* c32a601 (dev_test) rename xiaocai.txt to xc.txt
* d57678e new a file test
* 4e84326 (first) first commit

```

把开发完成的功能合并到`master`

```bash
$ git merge fun008
Auto-merging new.txt
CONFLICT (content): Merge conflict in new.txt
Automatic merge failed; fix conflicts and then commit the result.

 ~/Desktop/test (master|MERGING)

```

发现有冲突了，这个是典型合并，其他分支合并到master分支时常见的问题，而且还会自动创建一个解决冲突的MERGING 分支。

根据提示可以看到 `new.txt` 文件有冲突，那就手工解决就好，实际工作中根据实际沟通情况进行合并即可：`vi new.txt`

```txt
new v1

<<<<<<< HEAD
change bug code v2
=
fun008 code v2
>>>>>>> fun008

```

修改之后：

```txt
new v1

change bug code v2

fun008 code v2
```

保存文件，然后将解决之后的添加到暂存并提交表示合并完成：

```bash
 ~/Desktop/test (master|MERGING)
$ git add new.txt

 ~/Desktop/test (master|MERGING)
$ git commit -m 'fix confilcts'
[master 2512282] fix confilcts

 ~/Desktop/test (master)
$
```

可以看到合并完成之后，`MERGING` 分支自动消失，查看一下分支状态：

```bash
$ git log --oneline --decorate --graph --all
*   2512282 (HEAD -> master) fix confilcts
|\
| * aa5e554 (fun008) fun008 commit finish
| * 9b4d966 fun008 的功能
* | 659454d fix bug709
|/
| * 37c967e (dev) add dev code
|/
* c32a601 (dev_test) rename xiaocai.txt to xc.txt
* d57678e new a file test
* 4e84326 (first) first commit

```

`master` 已经走到最前面，一切正常，合并完成，fun008也可以删除了。

9、删除已经合并的分支

因为fun008分支代码已经合并完成，所以现在不需要这个分支了，可以删除：

```bash
$ git branch -d fun008
Deleted branch fun008 (was aa5e554).

```

再次查看：

```bash
$ git log --oneline --decorate --graph --all
*   2512282 (HEAD -> master) fix confilcts
|\
| * aa5e554 fun008 commit finish
| * 9b4d966 fun008 的功能
* | 659454d fix bug709
|/
| * 37c967e (dev) add dev code
|/
* c32a601 (dev_test) rename xiaocai.txt to xc.txt
* d57678e new a file test
* 4e84326 (first) first commit

```

可以看到`fun008`分支删除成功，只是并行提交的注释还在。

### 8、分支模式

git的分支模式

- 长期分支

- 特性分支 Topic

```txt
c1 --------------------------------> master
   \
    -----d1 --- d2---- d3   -------> develop
         \       \
          ----t1  ----t2    -------> topic
```

分支的本质

Git 的分支，其实就是提交对象，HEAD指向提交对象的可变指针。

分支原理

- `.git / refs` 目录保存了分支及对应的提交对象。

- HEAD引用。当执行`git branch br_name` 时，对应的HEAD文件中有一个符号引号，指向目前所在的分支。

  ```bash
  ref: refs/heads/master
  ```

  而分支目录`.git/refs/heads` 下存放所有的分支文件，文件中都是当前分支的提交对象的hash值：如我的`master`文件内容

  ```
  2512282454d92bd3ed0c7a4abc5bded1cfd7eac6
  ```

  

### 9、git存储

有时，当一部分工作完成后，所有东西进入混乱状态，但是此时想切换到另一个分支做点别的事情，但是因为很多事情没搞好，就要对现在不完整的工作进行一次提交。针对这个问题怎么办？

**git stash 命令**

基本命令格式

```bash
git stash
```

该命令可以将为完成的修改保存到一个栈上，然后我们可以在任何时间重新应用这些改动。

存储的本质是提交，只是git默认自动提交。但是使用`git log --oneline` 是看不到的。

应用（不删除栈内容）命令：

```
git stash apply stash@{2}
```

如果不指定一个储藏，Git会认为指定的是最近的储藏。

查看现有的存储：

```bash
git stash list
```

应用储藏然后立即从栈上扔掉（删除内容）：

```bash
git stash pop
```

指定储藏的名字然后移除它：

```bash
git stash drop
```



## 七、Git 后悔

针对三个区的后悔撤销：

- 工作区。文件要被git跟踪。
- 暂存区。撤回自己的暂存。
- 版本库。撤回自己的提交。

### 1、工作目录撤回

将工作区文件的修改撤销

```bash
git checkout -- file_Name
```

Demo：在工作区取消

新建一个文件，先提交一次，

```bash
echo 'test-abc v1' > abc.txt

git add ./

git commit -m "提交好了"
```

准备修改

```bash
echo 'test-abc v2' > abc.txt

cat abc.txt
test-abc v1 test-abc v2
```

撤销修改：

```bash
git status
```



### 2、暂存区撤回

在暂存区取消暂存

```bash
git reset HEAD  <file_Name>
```

新建一个文件，然后放到暂存区

```bash
echo 'test-123 v1' > 123.txt

git add ./
```

修改文件：

```bash
echo 'test-123 v2' > 123.txt

cat 123.txt
test-123 v1 test-123 v2
```

撤销操作，在取消暂存之前，先看暂存区内容：

```bash
# 查看暂存区
git ls-files -s

# 查看对应的暂存内容
git cat-file -p  <hash>

# 查看状态
git stats

git reset HEAD  123.txt
```

### 3、版本库撤回

```bash
git commit --amend
```

其实只是重新提交一次，同时允许修改注释。

