---
title: "Git 分支相关"
layout: post
date: 2017-06-18
tags:
- Other
- Git
categories: GIT
blog: true
excerpt: Git 分支创建，合并删除，冲突解决等
---

# Git 分支相关

- 创建分支：git branch <name>; git checkout -b myfeature develop: 基于develop创建分支
- 查看分支：git branch
- 切换分支：git checkout <name>
- 合并某分支到当前分支：git merge <name>
- 删除分支：git branch -d <name>
- 查看分支合并情况：$ git log --graph --pretty=oneline --abbrev-commit
- 创建+切换分支：git checkout -b <name>
- 查看远程库： git remote -v
- 推送分支：git push origin dev
- 克隆远程分支：git clone https://github.com/benson-lin/benson-lin.github.io.git。默认之后克隆master分支，如果需要在dev开发，需要创建远程origin的dev分支到本地（git checkout -b dev origin/dev），然后就可以提交了（git push origin dev）
- 本地代码与远程同步：git pull，是pull本地分支对应的远程分支代码，而不是master；(当因冲突而push失败时，应该先pull到本地，并解决冲突后在push； pull过程中提示“no tracking information”，则说明本地分支是自己新建的，此时直接执行git pull肯定是无效的，因为和远程分支没有创建链接关系，要用命令git branch --set-upstream branch-name origin/branch-name或git branch --set-upstream-to=origin/<branch> new-dev)


## 分支管理策略

在实际开发中，我们应该按照几个基本原则进行分支管理：

首先，master分支应该是非常稳定的，也就是仅用来发布新版本，平时不能在上面干活；

那在哪干活呢？干活都在dev分支上，也就是说，dev分支是不稳定的，到某个时候，比如1.0版本发布时，再把dev分支合并到master上，在master分支发布1.0版本；

**你和你的小伙伴们每个人都在dev分支上干活，每个人都有自己的分支，时不时地往dev分支上合并就可以了**。


并不是一定要把本地分支往远程推送，那么，哪些分支需要推送，哪些不需要呢？

- master分支是主分支，因此要时刻与远程同步；
- dev分支是开发分支，团队所有成员都需要在上面工作，所以也需要与远程同步；
- bug分支只用于在本地修复bug，就没必要推到远程了，除非老板要看看你每周到底修复了几个bug；
- feature分支是否推到远程，取决于你是否和你的小伙伴合作在上面开发。


## 例子

```linux
bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/test (master)
$ vim readme.txt

bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/test (master)
$ git add .

bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/test (master)
$ git commit -m "readme.txt"
[master (root-commit) c3a3865] readme.txt
 1 file changed, 2 insertions(+)
 create mode 100644 readme.txt

bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/test (master)
$ git branch dev

bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/test (master)
$ git branch
  dev
* master

bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/test (master)
$ git checkout dev
Switched to branch 'dev'

bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/test (dev)
$ git branch
* dev
  master

bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/test (dev)
$ vim readme.txt

bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/test (dev)
$ git add .

bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/test (dev)
$ git commit -m "dev modify"
[dev 6151da4] dev modify
 1 file changed, 2 insertions(+)

bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/test (dev)
$ git checkout master
Switched to branch 'master'

bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/test (master)
$ vim readme.txt

bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/test (master)
$ git add .
g
bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/test (master)
$ git commit -m "master"
[master a6ed0de] master
 1 file changed, 2 insertions(+)

bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/test (master)
$ git status
On branch master
nothing to commit, working tree clean

bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/test (master)
$ git merge dev
Auto-merging readme.txt
CONFLICT (content): Merge conflict in readme.txt
Automatic merge failed; fix conflicts and then commit the result.

bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/test (master|MERGING)
$ git status
On branch master
You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)

        both modified:   readme.txt

no changes added to commit (use "git add" and/or "git commit -a")

bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/test (master|MERGING)
$ git add .

bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/test (master|MERGING)
$ git commit -m "new"
[master d49fbb6] new

bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/test (master)
$ git branch
  dev
* master

bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/test (master)
$ git log
commit d49fbb6140a593dfd3f2d1a08c88317657b63d3b (HEAD -> master)
Merge: a6ed0de 6151da4
Author: benson-lin <1096101803@qq.com>
Date:   Sun Jun 18 15:11:30 2017 +0800

    new

commit a6ed0deec09889ca86d725b5e91e31ee4e438d30
Author: benson-lin <1096101803@qq.com>
Date:   Sun Jun 18 15:10:45 2017 +0800

    master

commit 6151da4b035de9002b4e68ccce0d80abe47fb35d (dev)
Author: benson-lin <1096101803@qq.com>
Date:   Sun Jun 18 15:08:24 2017 +0800

    dev modify

commit c3a3865ff9a5428851019616a0a74b2fa7c78eb3
Author: benson-lin <1096101803@qq.com>
Date:   Sun Jun 18 15:06:55 2017 +0800


bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/test (master)
$ git log --oneline
d49fbb6 (HEAD -> master) new
a6ed0de master
6151da4 (dev) dev modify
c3a3865 readme.txt

bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/test (master)
$ git branch -d dev
Deleted branch dev (was 6151da4).

bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/test (master)
$ git branch
* master

bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/test (master)
$ git log --graph --pretty=oneline --abbrev-commit
*   d49fbb6 (HEAD -> master) new
|\
| * 6151da4 dev modify
* | a6ed0de master
|/
* c3a3865 readme.txt

```


## 实战 


### 1. BUG紧急修复且有正在开发代码在工作区问题

概览：master合并merge解决好的bug后，不要先把dev解印，先合并master，获取里面的bug方案后，在解印。解印时会有提示冲突，需手动改一次文件。

1：在  dev 下正常开发中，说有1个bug要解决，首先我需要把dev分支封存stash

2：在master下新建一个issue-101分支，解决bug，成功后

3：在master下合并issue-101

4：在 dev  下合并master，  这样才同步了里面的bug解决方案

5：解开dev封印stash pop，系统自动合并 & 提示有冲突，因为封存前dev写了东西，此时去文件里手动改冲突

6：继续开发dev，最后add，commit

7：在master下合并最后完成的dev

代码过程如下：

```linux
在此插入代码

1： $ git stash

2： $ git checkout master
   $ git checkout -b issue-101
    //去文件里修bug
    $ git add README.md
    $ git commit -m "fix-issue-101"

3： $ git checkout master
   $ git merge --no-ff -m "m-merge-issue-101" issue-101
   $ git branch -d issue-101

4： $ git checkout dev
    $ git merge --no-ff -m "dev-merge-m" master

5： $ git stash pop
            //提示冲突，去文件手动改正
            Auto-merging README.md
            CONFLICT (content): Merge conflict in README.md

6： //继续开发 ... ... ，完成后一并提交
    $ git add README.md
    $ git commit -m "fixconflict & append something"

7： $ git checkout master
    $ git merge --no-ff -m "m-merge-dev" dev
    $ git branch -d dev
	
	
### 2. 删除无用分支且分支上有未合并代码问题


git branch -D dev

​```linux
bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/test2
$ git init
Initialized empty Git repository in C:/Users/bensonlin/Desktop/test2/.git/

bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/test2 (master)
$ git branch dev
fatal: Not a valid object name: 'master'. //需要有commit才有真正意义是master分支，然后才能创建其它分支

bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/test2 (master)
$ vim readme.txt

bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/test2 (master)
$ git add .

bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/test2 (master)
$ git commit -m "re"
[master (root-commit) ab6748e] re
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 readme.txt

bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/test2 (master)
$ git branch dev

bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/test2 (master)
$ git checkout dev
Switched to branch 'dev'

bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/test2 (dev)
$ touch dev.txt

bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/test2 (dev)
$ git add .

bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/test2 (dev)
$ git commit -m "dev.txt"
[dev 3b091e6] dev.txt
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 dev.txt

bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/test2 (dev)
$ git checkout master
Switched to branch 'master'

bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/test2 (master)
$ git branch -d dev
error: The branch 'dev' is not fully merged.
If you are sure you want to delete it, run 'git branch -D dev'.

bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/test2 (master)
$ git branch -D dev
Deleted branch dev (was 3b091e6).

```


## 3. 远程如何新建分支

push一个新的分支会自动创建新分支

```linux
bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/111/test (master)
$ git checkout -b dev
Switched to a new branch 'dev'

bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/111/test (dev)
$ touch dev.txt

bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/111/test (dev)
$ git push origin dev
Username for 'https://github.com': benson-lin
Total 0 (delta 0), reused 0 (delta 0)
To https://github.com/benson-lin/test.git
 * [new branch]      dev -> dev
```


## 4. 远程创建新分支后，其他人怎么得到这个新分支

直接git pull

```linux
bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/222/test (master)
$ git pull
From https://github.com/benson-lin/test
 * [new branch]      dev        -> origin/dev
Already up-to-date.
```

## 5. 得到新分支信息后，如何与远程关联

先本地新建分支，再git branch --set-upstream-to=origin/dev new-dev关联

```linux
$ git checkout -b new-dev
Switched to a new branch 'new-dev'

bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/222/test (new-dev)
$ git branch --set-upstream-to=origin/dev new-dev
Branch new-dev set up to track remote branch dev from origin.

bensonlin@bensonlin-PC1 MINGW64 ~/Desktop/222/test (new-dev)
$ git branch
  master
* new-dev
```
