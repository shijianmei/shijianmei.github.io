---
title: Git常用命令
date: 2020-03-25 13:33:25
categories: 
- Git

tags:
---
我每天使用 Git ，但是很多命令记不住。

一般来说，日常使用只要记住下图6个命令，就可以了。但是熟练使用，恐怕要记住60～100个命令。
![](https://raw.githubusercontent.com/shijianmei/blog_Images/main/git%E5%B8%B8%E7%94%A8%E5%91%BD%E4%BB%A4/git.png)

git 常用命令.png

下面是我整理的常用 Git 命令清单。几个专用名词的译名如下。

>Workspace：工作区
>Index / Stage：暂存区
>Repository：仓库区（或本地仓库）
>Remote：远程仓库
# 新建代码库

在当前目录新建一个Git代码库

```plain
$ git init
```

新建一个目录，将其初始化为Git代码库

```plain
git init [project-name]
```

下载一个项目和它的整个代码历史

```plain
$ git clone [url]
```

下载一个代码，并指定分支

```plain
$ git clone -b [branchName] [url]
```
# 配置

Git的设置文件为.gitconfig，它可以在用户主目录下（全局配置），也可以在项目目录下（项目配置）。

显示当前的Git配置

```plain
$ git config --list
```

编辑Git配置文件

```plain
$ git config -e [--global]
```

设置提交代码时的用户信息

```plain
$ git config [--global] user.name "[name]"
```
$ git config [--global] user.email "[email address]"
# 增加/删除文件

添加指定文件到暂存区

```plain
$ git add [file1] [file2] ...
```

添加指定目录到暂存区，包括子目录

```plain
$ git add [dir]
```

添加当前目录的所有文件到暂存区

```plain
$ git add .
```

添加每个变化前，都会要求确认

对于同一个文件的多处变化，可以实现分次提交

```plain
$ git add -p
```

删除工作区文件，并且将这次删除放入暂存区

```plain
$ git rm [file1] [file2] ...
```

停止追踪指定文件，但该文件会保留在工作区

```plain
$ git rm --cached [file]
```

改名文件，并且将这个改名放入暂存区

```plain
$ git mv [file-original] [file-renamed]
```

# 代码提交

提交暂存区到仓库区

```plain
$ git commit -m [message]
```

提交暂存区的指定文件到仓库区

```plain
$ git commit [file1] [file2] ... -m [message]
```

提交工作区自上次commit之后的变化，直接到仓库区

```plain
$ git commit -a
```

提交时显示所有diff信息

```plain
$ git commit -v
```

使用一次新的commit，替代上一次提交

如果代码没有任何新变化，则用来改写上一次commit的提交信息

```plain
$ git commit --amend -m [message]
```

重做上一次commit，并包括指定文件的新变化

```plain
$ git commit --amend [file1] [file2] ...
```

# 分支

列出所有本地分支

```plain
$ git branch
```

列出所有远程分支

```plain
$ git branch -r
```

列出所有本地分支和远程分支

```plain
$ git branch -a
```

新建一个分支，但依然停留在当前分支

```plain
$ git branch [branch-name]
```

新建一个分支，并切换到该分支

```plain
$ git checkout -b [branch]
```

新建一个分支，指向指定commit

```plain
$ git branch [branch] [commit]
```

新建一个分支，与指定的远程分支建立追踪关系

```plain
$ git branch --track [branch] [remote-branch]
```

切换到远程的一个分支 ,并映射到本地的一个分支

```plain
git checkout -b master_network origin/master_network
```

把本地分支提交到远程

```plain
$ git push origin [branch]
```

切换到指定分支，并更新工作区

```plain
$ git checkout [branch-name]
```

切换到上一个分支

```plain
$ git checkout -
```

建立追踪关系，在现有分支与指定的远程分支之间

```plain
$ git branch --set-upstream [branch] [remote-branch]
```

合并指定分支到当前分支

```plain
$ git merge [branch]
```

选择一个commit，合并进当前分支

```plain
$ git cherry-pick [commit]
```

删除分支

```plain
$ git branch -d [branch-name]
```

删除远程分支

```plain
$ git push origin --delete [branch-name]
$ git branch -dr [remote/branch]
```

更改本地和远程分支的名

```plain
git branch -m old_branch new_branch Rename branch locally 
git push origin :old_branch Delete the old branch 
git push --set-upstream origin new_branch Push the new branch, set local branch to track the new remote
```

# 标签

列出所有tag

```plain
$ git tag
```

新建一个tag在当前commit

```plain
$ git tag [tag]
```

新建一个tag在指定commit

```plain
$ git tag [tag] [commit]
```

删除本地tag

```plain
$ git tag -d [tag]
```

删除远程tag

```plain
$ git push origin --delete tag <tagname>
```

查看tag信息

```plain
$ git show [tag]
```

提交指定tag

```plain
$ git push [remote] [tag]
```

提交所有tag

```plain
$ git push [remote] --tags
```

新建一个分支，指向某个tag

```plain
$ git checkout -b [branch] [tag]
```

# 查看信息

显示有变更的文件

```plain
$ git status
```

显示当前分支的版本历史

```plain
$ git log
```

显示commit历史，以及每次commit发生变更的文件

```plain
$ git log --stat
```

搜索提交历史，根据关键词

```plain
$ git log -S [keyword]
```

显示某个commit之后的所有变动，每个commit占据一行

```plain
$ git log [tag] HEAD --pretty=format:%s
```

显示某个commit之后的所有变动，其"提交说明"必须符合搜索条件

```plain
$ git log [tag] HEAD --grep feature
```

显示某个文件的版本历史，包括文件改名

```plain
$ git log --follow [file]
$ git whatchanged [file]
```

显示指定文件相关的每一次diff

```plain
$ git log -p [file]
```

显示过去5次提交

```plain
$ git log -5 --pretty --oneline
```

显示所有提交过的用户，按提交次数排序

```plain
$ git shortlog -sn
```

显示指定文件是什么人在什么时间修改过

```plain
$ git blame [file]
```

显示暂存区和工作区的差异

```plain
$ git diff
```

显示暂存区和上一个commit的差异

```plain
$ git diff --cached [file]
```

显示工作区与当前分支最新commit之间的差异

```plain
$ git diff HEAD
```

显示两次提交之间的差异

```plain
$ git diff [first-branch]...[second-branch]
```

显示今天你写了多少行代码

```plain
$ git diff --shortstat "@{0 day ago}"
```

显示某次提交的元数据和内容变化

```plain
$ git show [commit]
```

显示某次提交发生变化的文件

```plain
$ git show --name-only [commit]
```

显示某次提交时，某个文件的内容

```plain
$ git show [commit]:[filename]
```

显示当前分支的最近几次提交

```plain
$ git reflog
```

# 远程同步

下载远程仓库的所有变动

```plain
$ git fetch [remote]
```

显示所有远程仓库

```plain
$ git remote -v
```

显示某个远程仓库的信息

```plain
$ git remote show [remote]
```

增加一个新的远程仓库，并命名

```plain
$ git remote add [shortname] [url]
```

取回远程仓库的变化，并与本地分支合并

```plain
$ git pull [remote] [branch]
```

上传本地指定分支到远程仓库

```plain
$ git push [remote] [branch]
```

强行推送当前分支到远程仓库，即使有冲突

```plain
$ git push [remote] --force
```

推送所有分支到远程仓库

```plain
$ git push [remote] --all
```

# 撤销

恢复暂存区的指定文件到工作区

```plain
$ git checkout [file]
```

恢复某个commit的指定文件到暂存区和工作区

```plain
$ git checkout [commit] [file]
```

恢复暂存区的所有文件到工作区

```plain
$ git checkout .
```

重置暂存区的指定文件，与上一次commit保持一致，但工作区不变

```plain
$ git reset [file]
```

重置暂存区与工作区，与上一次commit保持一致

```plain
$ git reset --hard
```

重置当前分支的指针为指定commit，同时重置暂存区，但工作区不变

```plain
$ git reset [commit]
```

重置当前分支的HEAD为指定commit，同时重置暂存区和工作区，与指定commit一致

```plain
$ git reset --hard [commit]
```

重置当前HEAD为指定commit，但保持暂存区和工作区不变

```plain
$ git reset --keep [commit]
```

强推到远程

```plain
$ git push origin HEAD --force
```

新建一个commit，用来撤销指定commit

后者的所有变化都将被前者抵消，并且应用到当前分支

```plain
$ git revert [commit]
```

暂时将未提交的变化移除，稍后再移入

```plain
$ git stash
$ git stash pop
```

参考:[https://git-scm.com/book/zh/v2](https://git-scm.com/book/zh/v2)

