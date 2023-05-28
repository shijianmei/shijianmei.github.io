---
title: Git不常用解决方案
date: 2020-04-26 13:50:02
categories: 
- Git
tags:
---
# 上传代码到 github
- 创建一个新的库上传
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:shijianmei/hotfixMS.git
git push -u origin main

- 上传一个已经存在的库
git remote add origin git@github.com:shijianmei/hotfixMS.git
git branch -M main
git push -u origin main

# rebase和 merge 的区别:
git merge 会生成一个新得合并节点，而rebase不会.

如两个分支:test 和 master

      D---E test

     /

A---B---C---F master

使用merge合并：

      D--------E

     /          \

A---B---C---F----G   test, master

而使用rebase则：

A---B---D---E---C'---F'   test, master

使用git pull时默认是merge， 加 --rebase参数使其使用rebase方式

```plain
git pull --rebase   
```
# git如何忽略已经提交的文件

* 首先将要忽略的文件或文件夹添加到 .gitignore 文件中
* 停止追踪指定文件
```powershell
// 忽略单个文件
git rm -r --cached [要忽略的文件/文件夹] 
// 忽略所有文件
git rm -r --cached .
```

* 推送到远程仓库
```plain
git add .
git commit -m " commet for commit ....."
git push
```
然后其他开发人员pull之后, ignore规则就对其生效了.

# git怎样删除未监视的文件untracked files

删除 untracked files:

```plain
git clean -f
```
连 untracked 的目录也一起删掉
```plain
git clean -fd
```

# git上传本地库到远程仓库

1. 拉取远程仓库
```plain
git clone git@gitlab.alibaba-inc.com:CMedia-iOS/YXWAliyunOSSModule.git
cd YXWAliyunOSSModule
touch README.md
git add README.md
git commit -m "add README"
git push -u origin master
```

2. 对于已存在的文件夹或仓库
```plain
cd existing_folder
git init
git remote add origin git@gitlab.alibaba-inc.com:CMedia-iOS/YXWAliyunOSSModule.git
git add .
git commit
git push -u origin master
```
# git reset --hard 、git push --force之后如何恢复

```plain
1.git reflog
2.查看丢失的commit id
3.创建一个新的分支，在新的分支对丢失的commit id执行：
git cherry-pick 901a33f
```
# 如何删除历史记录里的大文件

适用于从一个git项目中，将体积较大的资源彻底从git中删除，包括历史提交记录。

如果仅仅在目录中删除一个文件是不够的，只要在提交记录中有这个文件，那么 .git 中就会有这个文件的信息。

用 filter-branch 可以强制修改提交信息，将某个文件的历史提交痕迹也抹去，就像从来没有过这个文件一样。

1. 找出需要删除的大文件
先执行

```plain
git gc --prune=now
```
通过如下命令找到git中保存的大小排名前5的大文件
```plain
git verify-pack -v .git/objects/pack/pack-*.idx | sort -k 3 -g | tail -5
```
可以得到如下信息:
```plain
b36ba8c5e4749183794705473321ea312b3a409d blob   46486053 11171997 77887787 1 80c1c35362872c8da6e02117a670902efcdce987
8d1dd2ee60e08b0385fc217a25ea33f36119d694 blob   62475792 7355607 206667227
267a27fef686fbdfbd25210fe36c8a224a5c1a10 blob   81598760 24710676 214022834
abe16d4278121f1a3eacdf3a966326bfb581a741 blob   109294368 28516988 98937450
80c1c35362872c8da6e02117a670902efcdce987 blob   195184864 37268015 40619772
```
最后一条就是最大的文件记录，80c1c35362872c8da6e02117a670902efcdce987 是其id，通过如下命令找到该文件的位置
```plain
git rev-list --objects --all | grep 80c1c35362872c8da6e02117a670902efcdce987
```
可以得到如下结果
```plain
80c1c35362872c8da6e02117a670902efcdce987 AWU3dControl/Frameworks/UnityFramework.framework
```
AWU3dControl/Frameworks/UnityFramework.framework 即为文件的位置，一般情况下这里的文件位置应该为文件夹/文件夹/文件的格式
 
2. 重写所有commit
```plain
git filter-branch --tree-filter "rm -f {filepath}" -- --all
```

3. 收尾
到这里，历史记录中已经没有该文件了。不过运行 filter-branch 产生的日志还是会对该文件有引用，所以还需要运行以下几条命令，把该文件的引用完全删除：

```plain
rm -Rf .git/refs/original
rm -Rf .git/logs/
git gc
git prune
```

4. push
```plain
git push -f --all
```
可以通过以下命令查看.git文件夹大小
```plain
du -sh .git
```
# git stash clear之后的恢复

```powershell
git fsck --lost-found, 找出刚才删除的提交对象和文件对象, 里面是一个一个的 dangling commit commitId
git show commitId一个一个的查看修改的内容, 找到了我们需要恢复的commitId之后
git stash apply commitId直接恢复就可以了
```

# 配置命令的别名

* 命令行设置 git 的配置 alias
```powershell
git config --global alias.<简化的字符> 原始命令
```
如: 设置获取状态
```powershell
git config --global alias.st status #代表输入 git st 就代表 git status
```
* 手动配置 ~/.gitconfig 文件
```plain
        pl = pull
        st = status
        ci = commit
        co = checkout
        br = branch
        df = diff
        unstage = reset HEAD --
        lg = log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Cres    et' --abbrev-commit
        last = log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Cr    eset' --abbrev-commit -1 HEAD
  [core]
    excludesfile = /Users/mac/.gitignore_global
    editor = /usr/local/bin/vim
  [push]
        default = simple
  [difftool "sourcetree"]
    cmd = opendiff \"$LOCAL\" \"$REMOTE\"
    path =
  [mergetool "sourcetree"]
    cmd = /Applications/Sourcetree.app/Contents/Resources/opendiff-w.sh \"$LOCAL\" \"$REMOTE\" -ancestor \"$BASE\" -merge \"$MERGED\"
    trustExitCode = true
```

