# Git

#### 内容摘要

版本控制，Git 特点，VS SVN，Git 基本使用

---

#### 版本控制

没有版本控制

* 备份多版本，浪费时间，浪费空间
* 难于恢复到以前正确版本
* 引发 Bug
* 解决**代码冲突**困难（多人协作）
* 代码管理困难
* 难于追溯问题代码的修改人和修改时间
* 无法进行权限控制
* 项目版本发布困难

版本控制 **Revision Control**

* Git
* SVN
* Perforce
* cvs

#### Git 特点

* 分布式，强调个人
* 离线工作
* 每日工作备份
* 可以回滚

#### Git VS SVN

* Git 是分布式，先提交到本地服务器，再合并到远程仓库
* SVN 是集中式，提交直接提交到服务器，一旦网络不畅通，就不能提交

#### Git 基本使用

`git init --bare [name]` 在当前目录下创建文件夹 **name**，本地空的 Git 仓库

```bash
~/test/git-demo $ git init --bare myFirstRepository
```

仓库目录结构

```java
myFirstRepository
	|- HEAD // 头指针
    |- description 
    |- info        
    |- refs // 版本间引用，当前版本引用
	|- config      
	|- hooks       
	|- objects // 存代码，增
```

`git init ` 在当前目录下，创建本地 Git 仓库，仓库目录会隐藏 `.git`

```bash
~/test/git-demo/second $ git init
# Initialized empty Git repository in /Users/clixin/Test/git-demo/second/.git/
```

`git clone <repo dir> .`  从远程仓库克隆到当前目录

```bash
~/test/git-dev/secondApp $ git clone ~/test/git-demo/second .
# Cloning into '.'...
# warning: You appear to have cloned an empty repository.
# done.
```

假设向当前目录添加测试文件 Hello.java

```bash
vim Hello.java
```

`git status` 查看当前提交状态和文件变化

```bash
~/test/git-dev/secondApp $ git status
## 当前目录没有任何文件
# On branch master
# No commits yet
# nothing to commit (create/copy files and use "git add" to track)

## 当前目录有文件 Hello.java
# On branch master
# No commits yet
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#	Hello.java
# nothing added to commit but untracked files present (use "git add" to track)
```

`git add [file.name]` 将 **file.name** 文件加入到待提交列表

```bash
~/test/git-dev/secondApp $ git add Hello.java
## add Hello.java  git status，未提交
# On branch master
# No commits yet
# Changes to be committed:
#  (use "git rm --cached <file>..." to unstage)
# new file:   Hello.java
```

`git commit -m [comment]` 将待提交列表的文件提交到**当前仓库 userA**， **comment** 可以写注释

```bash
~/test/git-dev/secondApp $ git commit -m "first commit Hello.java with main method"
# [master (root-commit) cab6aff] first commit Hello.java with main method
# 1 file changed, 5 insertions(+)
# create mode 100644 Hello.java
```

`git log` 查看当前仓库提交日志，根据 commit 时间排序

```bash
~/test/git-dev/secondApp $ git log
# commit cab6affa92bfb1ba357e70e6c46e74e874c08a9e (HEAD -> master)
# Author: Clixin <Clixin@foxmail.com>
# Date:   Mon Apr 9 13:17:14 2018 +0800
#  first commit Hello.java with main method
```

第一次使用，配置用户信息

```bash
git config --globale
```

#### 进阶操作

`git reset --hard [version]` 版本回退到 [版本号] ，同时更新文件

*  `HEAD^` 上一版本 `HEAD^^` 上上版本 `HEAD~10` 上10个
* 也可以是 **SHA1** 哈希值前几位，指代每一次提交版本

```bash
test/git-dev/secondApp $ git reset --hard HEAD^
# HEAD is now at cab6aff first commit Hello.java with main method
```

#### 工作区和暂存区

工作区：项目目录（ `.git` 文件夹所在目录）

版本库：`.git` 目录下文件

暂存区：`.git/stage`  `add` 操作将 **已修改文件** 放入暂存区

**master**：`commit` 将暂存区的文件提交到版本控制 **master** 分支

`git reset HEAD <file> unstaged`  从暂存区移出

#### 远程操作

`git clone <repo dir> .`  **clone** 从远程仓库克隆到当前目录

`git push origin master`  **push** 将本地仓库推送到远程仓库

* **origin** 收件人，远程仓库名， **clone** 目的地地址
* **master** 发件人，当前目录，本地仓库，分支名，主分支 **master**

```bash
git push origin master
```

`git pull` **pull** 从远程仓库拉取更新的文件

```bash
git pull
```

提交冲突

* 修改不同文件时，A 先提交，B 后提交，B 需要先将 A **pull** 下来，本地自动 **merge**，再提交
* 修改同一文件同一地方时，A 先提交，B 后提交，B 需要先将 A **pull** 下来，手动 **merge**，再提交

#### IDEA 使用 Git

#### GitHub

`.gitignore`



















