---
title: git八股
date: 2026-02-11
category: 'git'
---

# 常用的git命令

- **git clone**：克隆远程仓库到本地。
- **git init**：在当前目录初始化一个新的Git仓库。
- **git add** ：将文件添加到暂存区，准备提交。
- **git commit -m "commit_message"**：提交暂存区的改动到本地仓库，附带提交信息。
- **git status**：查看工作区、暂存区的状态，显示文件的修改情况。
- **git diff**：显示工作区与暂存区之间的差异。
- **git diff --staged**：显示暂存区与最后一次提交之间的差异。
- **git log**：显示提交日志，包括提交哈希、作者、日期等信息。
- **git branch**：列出所有分支，当前分支前会有一个星号。
- **git checkout** ：切换到指定分支。
- **git checkout -b** ：创建并切换到新分支。
- **git merge** ：将指定分支合并到当前分支。
- **git pull** ：拉取远程仓库的更新并合并到当前分支。
- **git push** ：将本地分支的更新推送到远程仓库。
- **git stash**
- **git reset --soft**
- git cherry-pick
- git revert
- git reflog

# git rebase和git merge的区别

`git rebase` 和 `git merge` 都是用于合并分支的 `Git` 命令，这两个命令都能将一个分支合并到另一个分支，但两者合并方式有很大不同

- **git merge**: 将一个分支的更改合并到另一个分支，创建一个新的`merge commit`，将两个分支的历史合并在一起，这个`merge commit`会在分支历史中保留，可以清晰的看到那些分支合并到了柱分枝，合并后形成分叉结构。 ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/04ce3184ea72496b979db9ac07f570fc~tplv-k3u1fbpfcp-image.image#?w=800&h=536&e=svg&a=1&b=50cfa1)
- **git rebase**：两个分支在合并到时候，会将整个分支和合并到另一个分支的顶端。首先找到两个分支的共同`commit`记录，然后提取之后所有的`commit`，然后将这个`commit`记录添加到另一个分支的最前面，两个分支合并后的`commit`记录就变成线性记录 ![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1fb99a25a021412aacad84432eea7378~tplv-k3u1fbpfcp-image.image#?w=800&h=545&e=svg&a=1&b=51cfa1)

#

作者：wakaka378
链接：https://juejin.cn/post/7272009063406272571
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
