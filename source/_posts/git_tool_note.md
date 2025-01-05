---
title: Git工具-基础使用
date: 2019-11-15
categories:
  - 工作笔记
tags:
  - git
---

### 代码拉取到本地

#### 与主分支保持同步
```
git checkout main
git pull origin main
git checkout feature/my-new-feature
git merge main
```
#### 代码拉取冲突处理

**1、保留并继续合并代码**

```
localhost:bookstore liaoxb$ git pull
Updating d369e85..67e8e8a
error: Your local changes to the following files would be overwritten by merge:
	test.txt
Please commit your changes or stash them before you merge.
Aborting
localhost:bookstore liaoxb$
localhost:bookstore liaoxb$ echo 'local change test' > test.txt
localhost:bookstore liaoxb$ git pull
Updating d369e85..67e8e8a
error: Your local changes to the following files would be overwritten by merge:
	test.txt
Please commit your changes or stash them before you merge.
Aborting
```

git stash 暂存本地的代码修改
git pull 拉取远端最新代码
git stash pop 合并暂存的本地代码，解决已冲突的文件

```
localhost:bookstore liaoxb$ git stash
Saved working directory and index state WIP on master: d369e85 update
localhost:bookstore liaoxb$ git pull
Updating d369e85..67e8e8a
Fast-forward
 test.txt | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
localhost:bookstore liaoxb$ git stash pop
Auto-merging test.txt
CONFLICT (content): Merge conflict in test.txt
```

**2、远端覆盖本地代码**

git reset –hard 回滚到上个版本
git pull

```
localhost:bookstore liaoxb$ git reset --hard
HEAD is now at d369e85 update
localhost:bookstore liaoxb$ git pull
Updating d369e85..67e8e8a
Fast-forward
 test.txt | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
```

### 创建分支并推送到远端

```sh
# 创建新分支,同时切换到bugfix1
localhost:bookstore liaoxb$ git pull
Already up to date.
localhost:bookstore liaoxb$ git checkout -b bugfix1
Switched to a new branch 'bugfix1'

# 修改并提交代码
localhost:bookstore liaoxb$ echo 'bugfix test' > test.txt
localhost:bookstore liaoxb$ git add .
localhost:bookstore liaoxb$ git commit -m "bugfix test"
[bugfix1 2eafe98] bugfix test
 1 file changed, 1 insertion(+), 3 deletions(-)
 
# 新分支push到远端
localhost:bookstore liaoxb$ git push --set-upstream origin bugfix1
Counting objects: 3, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 259 bytes | 259.00 KiB/s, done.
Total 3 (delta 1), reused 0 (delta 0)
remote: Powered by GITEE.COM [GNK-3.0]
remote: Create a pull request for 'bugfix1' on Gitee by visiting:
remote:     https://gitee.com/wisecloud/bookstore/pull/new/wisecloud:bugfix1...wisecloud:master
To https://gitee.com/wisecloud/bookstore.git
 * [new branch]      bugfix1 -> bugfix1
Branch 'bugfix1' set up to track remote branch 'bugfix1' from 'origin'.
```

### 代码合并到主分支

#### 方法一：git merge

```sh
# 切至合并分支
localhost:bookstore liaoxb$ git branch
* bugfix1
  master
localhost:bookstore liaoxb$ git checkout master
Switched to branch 'master'
Your branch is up to date with 'origin/master'.

# 执行合并操作
localhost:bookstore liaoxb$ git merge bugfix1
Updating d369e85..2eafe98
Fast-forward
 test.txt | 4 +---
 1 file changed, 1 insertion(+), 3 deletions(-)
 
# git push到远端
localhost:bookstore liaoxb$ git push
Total 0 (delta 0), reused 0 (delta 0)
remote: Powered by GITEE.COM [GNK-3.0]
To https://gitee.com/wisecloud/bookstore.git
   d369e85..2eafe98  master -> master
```

#### 方法二：git cherry-pick
```sh
# 切换到master分支
git checkout master

# Cherry pick 操作
git cherry-pick <commit_1> <commit_2> 

# 推送
git push
```

> 执行合并操作期间，若遇到代码冲突，需要先解决后，才能继续合并提交

#### 代码合并冲突处理

```sh
# 解决完冲突后，将修改的文件重新加入暂存区
git add .

# 继续剩余的提交
git cherry-pick --continue

# 推送
git push
```

```sh
# 遇到冲突，放弃合并，回到操作前的样子
git cherry-pick --abort

# 遇到冲突，放弃合并，不回到操作前的样子
git cherry-pick --quit
```

### 本地和远端删除已废弃分支

```sh
# 删除远端分支
localhost:bookstore liaoxb$ git branch -r
  origin/HEAD -> origin/master
  origin/bugfix1
  origin/master
localhost:bookstore liaoxb$ git branch -d origin/bugfix1
Deleted remote-tracking branch origin/bugfix1 (was 2eafe98).

# 删除本地分支
localhost:bookstore liaoxb$ git push origin --delete origin/bugfix1
remote: Powered by GITEE.COM [GNK-3.0]
To https://gitee.com/wisecloud/bookstore.git
 - [deleted]         bugfix1
localhost:bookstore liaoxb$ git branch -r
  origin/HEAD -> origin/master
  origin/master
```

### 分支引入错误代码，需要回滚

回滚至某次commit，之后的所有commit全部会丢失

```sh
git reset --hard <commit_id>
git push --force
```
