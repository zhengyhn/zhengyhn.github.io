---
{
  "title": "Git Cheat Sheet in Our Team",
  "subtitle": "Generic subtitle",
  "date": "2018-09-26",
  "slug": "Git Cheat Sheet in Our Team"
}
---
<!--more-->

由于近期有团队成员对git不是很熟悉，加上团队对git操作没有规范，通过此文规范团队的git规范，同时让成员学会一些常用的git操作，提高工作效率，减少错误。

其实本人也是半桶水，git有很多技巧以及高级用法，我都没用上，只是记录一些工作中经常使用的技巧和个人推崇的规范。

## Git流程

结合团队的情况，由于我们追求快速迭代，业务不是特别复杂，并且把任务拆分很细，所以基本每个人独立开发一个任务，开发完立即发布，没有太多任务之间的依赖，所以制定Git流程如下：

接到任务 --> 基于本地master分支新建任务分支 --> 开发 --> 在该分支上构建docker镜像部署到测试环境测试 --> 提pull request --> 合并到远程master分支

整个开发过程基本上会使用到下面的命令:
```

# 切换到master本地分支
git checkout master
# 拉取远程master分支代码并合并到本地master分支
git pull origin master

# 基于本地master分支新创建本地分支feat-refract-register
git checkout -b feat-refract-register
# 查看当前修改状态
git status
# 将修改放入暂存区
git add -A
# 将暂存区的修改放入本地仓库
git commit -m "refract register"
# 将修改放入暂存区
git add -A
# 将暂存区的修改放入本地仓库，并合并到上一个commit中
git commit --amend
# 将本地分支feat-refract-register推送到远程的同名分支
git push origin feat-refract-register -f

# 切换到master本地分支
git checkout master
# 拉取远程master分支代码并合并到本地master分支
git pull origin master

# 重新以本地master分支为base，将修改置于该base之上，并修复冲突
git rebase master
# 将本地分支feat-refract-register推送到远程的同名分支
git push origin feat-refract-register -f
```

## 保持本地master分支最新

养成习惯，在新建分支之前，保持本地master分支最新，可以防止在一份很旧的代码上修改，以碰到bug或者奇怪的问题。
```

# 切换到master本地分支
git checkout master
# 拉取远程master分支代码并合并到本地master分支
git pull origin master
```

## 新建分支规范

由于我们的业务简单，而且发布快速，不会出现将所有改动合并成一个release一起发布的情况，所以制定新建分支规范如下：

- 任务分支: feat-xxxx
- 修bug分支: fix-xxxx

除非特殊情况，每次新建分支应该在本地master分支的基础上创建。
```

# 基于本地master分支新创建本地分支feat-refract-register
git checkout -b feat-refract-register
```

## commit规范

### commit message应该要详细清晰

下面是好的message:
```

commit 72508447e05deec792832f5286d27b25a373be1a (HEAD -> master)
Author: yuanhang zheng <yuanhang.zheng@qq.com>
Date:   Thu Sep 27 23:36:27 2018 +0800

    1. 增加了注册功能
    2. 增加redis缓存加快查询功能
```

而这一个是差的message:
```

commit 7f3cbe4ff557550807999df6604f5b1220821fc2 (origin/master, origin/HEAD)
Author: yuanhang zheng <yuanhang.zheng@qq.com>
Date:   Mon Sep 3 22:11:32 2018 +0800

    solve some problems
```

### 提pr之前相对于master分支应该只有一个commit

由于我们的任务一般情况下只有一个人在开发，所以所有的改动应该合并成一个commit，通过下面的方法来实现。
```

# 将修改放入暂存区
git add -A
# 将暂存区的修改放入本地仓库
git commit -m "refract register"

# 切换另外一个分支开发另外一个功能
# 切换回这个分支，继续开发

# 将修改放入暂存区
git add -A
# 将暂存区的修改放入本地仓库，并合并到上一个commit中
git commit --amend
```

如果是多个人共用一个分支开发，每次提交应该使用一个独立的commit，如果使用--amend，另外一个人pull代码将会冲突，等到开发完成提pr之前再把多个commit合并成一个。

## 如何将多个commit合并成一个

这是工作中比较常用的技巧，有两种方法。

### 使用rebase

首先使用git log找到要合并的commit里面最早的一条的hash值，复制下来，记做aaa吧。
然后，通过git rebase -i aaa ，这个时候会出现一个commit的页面，在最顶部默认全部commit前面是pick:
```

pick dcaaa6c add more problems
pick 33b40c3 solve more problems
pick 7f3cbe4 solve some problems
pick 1fd23bb test

# Rebase 43ca4bf..1fd23bb onto 43ca4bf (4 commands)
#
# Commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# x, exec = run command (the rest of the line) using shell
# d, drop = remove commit
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

下面有注释说明，pick意思是使用这个commit，我们要合并成一个commit，就把前面的改成s(squash)，意思是使用这个commit，但是会合并到上一个commit，变成这样子：
```

s dcaaa6c add more problems
s 33b40c3 solve more problems
s 7f3cbe4 solve some problems
s 1fd23bb test

```

保存退出之后，将会出现下一个页面:
```

# This is a combination of 5 commits.
# This is the 1st commit message:

solve some problems

# This is the commit message #2:

add more problems

# This is the commit message #3:

solve more problems

# This is the commit message #4:

solve some problems

# This is the commit message #5:

test

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Mon Jun 25 07:39:33 2018 +0800
#
# interactive rebase in progress; onto 43ca4bf
# Last commands done (4 commands done):
#    s 7f3cbe4 solve some problems
#    s ca161f2 test
# No commands remaining.
# You are currently rebasing branch 'master' on '43ca4bf'.
#
```

这个时候，我们就要把commit的message合并成一个了，比如这样子：
```

test solve some problems

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Mon Jun 25 07:39:33 2018 +0800
#
# interactive rebase in progress; onto 43ca4bf
# Last commands done (3 commands done):
#    s 33b40c3 solve more problems
#    s 7f3cbe4 solve some problems
# Next command to do (1 remaining command):
#    pick 1fd23bb test
# You are currently rebasing branch 'master' on '43ca4bf'.
#
```

再保存退出，这个时候git log来看就变成一个commit了。

### 使用reset

git reset也是可以做到合并commit的效果，与其说合并commit，不如说是把前面的commit去掉，再提交一个commit。

首先git log找到要合并的commit里面，最早的commit，并找到在它之前的一个commit的hash值，记做aaa吧。
然后，通过git reset --soft aaa来将commit回到aaa的历史状态，--soft表示工作区的代码不丢弃，虽然回到那个历史状态，但是改动的代码还是会保留。
最后，再git commit一下，编辑commit message，提交就是一个commit了。


## 在当前任务分支强制推送到远程分支

做了--amend的commit和rebase或reset之后，往往需要强制推送到远程分支，由于一个分支一般只有一个人开发，强制更新是没有问题的，如果是master，在非特殊情况下是不允许的。所以我们有：
```

# 将本地分支feat-refract-register推送到远程的同名分支
git push origin feat-refract-register -f
```

## 应该经常rebase最新的master代码

我们之前的用法是在当前分支合并master的代码，这样子会增加一条merge的commit，每次merge master的代码，就会增加多一个commit，污染了整个commit的历史，而且不适合回滚。

如果我们时刻保持只有一个commit，并且经常rebase master的代码，那提pr的时候将会非常顺利，不会有冲突，提交也是非常干净的。

rebase的命令非常简单，在本地同步了远程master代码之后，在任务分支上，执行下面命令：
```

# 重新以本地master分支为base，将修改置于该base之上，并修复冲突
git rebase master
```

rebase的时候有可能会有冲突，解决冲突之后，就用git add放到暂存区，然后用git rebase --continue继续进行rebase。

如果当前改动提交了太多commit，那rebase将会很痛苦，因为rebase的原理会基于master分支应用每一个commit，如果每一个commit都有冲突，将要解决多次冲突，很麻烦，这就是为什么要时刻保持只有一个commit的原因。

## 如何回滚代码

git revert可以直接回滚代码，首先git log找到你要回滚的commit的hash值，记做aaa吧，然后:
```

git revert aaa
```

它的原理是，增加一个commit，对原commit做反向操作。

操作完成之后，还要把本地的分支推送到远程分支，以达到回滚的效果。

## 本地的commit弄乱了，如何同步远程分支的代码并保留我的改动?

这是一个小技巧，同样是用git reset来实现的。

首先，找到一个很久以前的commit的hash值，记做aaa吧。
然后，git reset --soft aaa，它会将commit回退到aaa的状态，但是代码保留现在的。
接着，git pull origin master，同步远程分支的代码下来，再commit现在的改动就行了。


