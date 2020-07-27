---
title: Git Command
date: 2020-07-27 22:22:20
tags:
categories:
  - Git

---
```
mkdir mygit
git init
git add readme.txt
git commit -m "git test"
git status
git diff readme.txt
git log //查看修改内容
如果嫌输出信息太多，看得眼花缭乱的，可以试试加上--pretty=oneline参数：
git log --pretty=oneline
git reset --hard HEAD^ //回退到上一个版本
git reset --hard HEAD^ //回退到上上个版本
git reset --hard HEAD~100 //回退到上100个版本

git reflog //记录每次commit记录
git reset --hard 1094a //版本号从reflog获得

git checkout -- readme.txt //把readme.txt文件在工作区的修改全部撤销，git checkout -- file命令中的--很重要，没有--，就变成了“切换到另一个分支”的命令，我们在后面的分支管理中会再次遇到git checkout命令。

git reset HEAD <file>可以把暂存区的修改撤销掉（unstage），重新放回工作区

//删除文件
git rm test.txt
git commit -m "rm test.txt"


```
<!--more-->
```
git branch查看分支，*表示目前所在的分支
git checkout -b dev：
    git checkout命令加上-b参数表示创建并切换，相当于以下两条命令：
    $ git branch dev
    $ git checkout dev 切换分支
git merge dev 合并dev分支到master上
git branch -d dev 删除dev分支


Git鼓励大量使用分支：

查看分支：git branch

创建分支：git branch <name>

切换分支：git checkout <name>或者git switch <name>

创建+切换分支：git checkout -b <name>或者git switch -c <name>

合并某分支到当前分支：git merge <name>

删除分支：git branch -d <name>

总结：先创建分支，原话修改代码，最后合并到master上
```



# [git-ssh 配置和使用](https://segmentfault.com/a/1190000002645623)

https://segmentfault.com/a/1190000002645623