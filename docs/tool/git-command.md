Git 常用命令，参考 [https://gitee.com/all-about-git](https://gitee.com/all-about-git) 。
<!-- more -->

概念：**工作区 -- `git add` --> 暂存区/缓存区 -- `git commit` --> 本地仓库 -- `git push` --> 远程仓库**

## 1 配置

```bash
# 显示当前的Git配置
$ git config --list

# 编辑Git配置文件
$ git config -e [--global]

# 设置提交代码时的用户信息
$ git config [--global] user.name "[name]"
$ git config [--global] user.email "[email address]"

# 配置会保存在当前用户目录下的 .gitconfig 文件中
```

## 2 仓库

```bash
# 在当前目录新建一个Git代码库
$ git init

# 新建一个目录，将其初始化为Git代码库
$ git init [project-name]

# 下载一个项目和它的整个代码历史
$ git clone [url]
```

## 3 增加/删除文件

```bash
# 添加指定文件到暂存区
$ git add [file1] [file2] ...

# 添加指定目录到暂存区，包括子目录
$ git add [dir]

# 添加当前目录的所有文件到暂存区
$ git add .

# 添加每个变化前，都会要求确认
# 对于同一个文件的多处变化，可以实现分次提交
$ git add -p

# 删除工作区文件，并且将这次删除放入暂存区
$ git rm [file1] [file2] ...

# 停止追踪指定文件，但该文件会保留在工作区
$ git rm --cached [file]

# 改名文件，并且将这个改名放入暂存区
$ git mv [file-original] [file-renamed]
```

## 4 代码提交

```bash
# 提交暂存区到仓库区
$ git commit -m [message]

# 提交暂存区的指定文件到仓库区
$ git commit [file1] [file2] ... -m [message]

# 提交工作区自上次commit之后的变化，直接到仓库区
$ git commit -a

# 提交时显示所有diff信息
$ git commit -v

# 使用一次新的commit，替代上一次提交
# 如果代码没有任何新变化，则用来改写上一次commit的提交信息
$ git commit --amend -m [message]

# 重做上一次commit，并包括指定文件的新变化
$ git commit --amend [file1] [file2] ...
```

## 5 分支

```bash
# 列出所有本地分支
$ git branch

# 列出所有远程分支
$ git branch -r

# 列出所有本地分支和远程分支
$ git branch -a

# 新建一个分支，但依然停留在当前分支
$ git branch [branch-name]

# 新建一个分支，并切换到该分支
$ git checkout -b [branch]

# 新建一个分支，指向指定commit
$ git branch [branch] [commit]

# 新建一个分支，与指定的远程分支建立追踪关系
$ git branch --track [branch] [remote-branch]

# 切换到指定分支，并更新工作区
$ git checkout [branch-name]

# 切换到上一个分支
$ git checkout -

# 建立追踪关系，在现有分支与指定的远程分支之间
$ git branch --set-upstream [branch] [remote-branch]

# 合并指定分支到当前分支
$ git merge [branch]

# 选择一个commit，合并进当前分支
$ git cherry-pick [commit]

# 删除分支
$ git branch -d [branch-name]

# 删除远程分支
$ git push origin --delete [branch-name]
$ git branch -dr [remote/branch]
```

## 6 标签

```bash
# 列出所有tag
$ git tag

# 新建一个tag在当前commit
$ git tag [tag]

# 新建一个tag在指定commit
$ git tag [tag] [commit]

# 删除本地tag
$ git tag -d [tag]

# 删除远程tag
$ git push origin :refs/tags/[tagName]

# 查看tag信息
$ git show [tag]

# 提交指定tag
$ git push [remote] [tag]

# 提交所有tag
$ git push [remote] --tags

# 新建一个分支，指向某个tag
$ git checkout -b [branch] [tag]
```

## 7 查看信息

```bash
# 显示有变更的文件
$ git status

# 显示当前分支的版本历史/提交日志
$ git log

# 显示简略信息
$ git log --pretty=short

# 指定文件
$ git log [file]

# 显示指定文件相关的每一次变化diff
$ git log -p [file]

# 显示commit历史，以及每次commit发生变更的文件
$ git log --stat

# 搜索提交历史，根据关键词
$ git log -S [keyword]

# 显示某个commit之后的所有变动，每个commit占据一行
$ git log [tag] HEAD --pretty=format:%s

# 显示某个commit之后的所有变动，其"提交说明"必须符合搜索条件
$ git log [tag] HEAD --grep feature

# 显示某个文件的版本历史，包括文件改名
$ git log --follow [file]
$ git whatchanged [file]

# 显示过去5次提交
$ git log -5 --pretty --oneline

# 显示所有提交过的用户，按提交次数排序
$ git shortlog -sn

# 显示指定文件是什么人在什么时间修改过
$ git blame [file]

# 显示工作区和暂存区的差异
$ git diff

# 显示工作区和当前分支最新 commit 之间的差异（本地仓库）
$ git diff HEAD

# 显示暂存区和上一个commit的差异
$ git diff --cached [file]

# 显示两次提交之间的差异
$ git diff [first-branch]...[second-branch]

# 显示今天你写了多少行代码
$ git diff --shortstat "@{0 day ago}"

# 显示某次提交的元数据和内容变化
$ git show [commit]

# 显示某次提交发生变化的文件
$ git show --name-only [commit]

# 显示某次提交时，某个文件的内容
$ git show [commit]:[filename]

# git reflog 命令可以显示整个本地仓库的 commit, 包括所有 branch 的 commit, 甚至包括已经撤销的 commit, 只要 HEAD 发生了变化, 就会在 reflog 里面看得到。
# 而 git log 只显示当前分支的 commit，并且不显示删除掉的 commit。
$ git reflog
```

## 8 远程同步

```bash
# 下载远程仓库的所有变动
$ git fetch [remote]

# 显示所有远程仓库
$ git remote -v

# 显示某个远程仓库的信息
$ git remote show [remote]

# 增加一个新的远程仓库，并命名
$ git remote add [shortname] [url]

# 取回远程仓库的变化，并与本地分支合并
$ git pull [remote] [branch]

# 上传本地仓库的指定分支到远程仓库
$ git push [remote] [branch]

# 上传本地仓库的指定分支到远程仓库，同时设置 upstream，方便后续操作
# -u 参数可以在推送的同时，将 origin 仓库的 master 分支设置为本地仓库当前分支的 upstream（上游），这个参数也只用在第一次 push 时加上，以后直接运行 git push 命令即可。同 git push --set-upstream origin master
$ git push -u [remote] [branch]

# 强行推送当前分支到远程仓库，即使有冲突
$ git push [remote] --force

# 推送所有分支到远程仓库
$ git push [remote] --all
```

## 9 撤销

```bash
# 恢复暂存区的指定文件到工作区
$ git checkout [file]

# 恢复某个commit的指定文件到暂存区和工作区
$ git checkout [commit] [file]

# 恢复暂存区的所有文件到工作区
$ git checkout .

# 重置暂存区的指定文件，与上一次commit保持一致，但工作区不变
$ git reset [file]

# 重置暂存区与工作区，与上一次commit保持一致
$ git reset --hard

# 重置当前分支的指针为指定commit，同时重置暂存区，但工作区不变
$ git reset [commit]

# 重置当前分支的HEAD为指定commit，同时重置暂存区和工作区，与指定commit一致
$ git reset --hard [commit]

# 重置当前HEAD为指定commit，但保持暂存区和工作区不变
$ git reset --keep [commit]

# 新建一个commit，用来撤销指定commit
# 后者的所有变化都将被前者抵消，并且应用到当前分支
$ git revert [commit]

# 暂时将未提交的变化移除，稍后再移入
$ git stash
$ git stash pop
```

## 10 其他

```bash
# 生成一个可供发布的压缩包
$ git archive
```

## 11 场景1：推送到远程仓库

```bash
# 初始化本地仓库 testgit
git init testgit

# 查看仓库状态，显示 nothing to be commit
# git status

# 工作区做文件夹/文件的增删改查操作
# ......

# 查看仓库状态，显示 Untracked files
# git status

# 添加工作区当前目录的所有文件到暂存区
git add .

# 查看仓库状态，显示 Changes to be committed
# git status

# 提交暂存区的文件到本地仓库
git commit -m "本次提交的备注信息"

# 查看仓库状态，显示 nothing to be commit
# git status

# 查看提交日志
# git log

# 显示所有远程仓库，显示空
# git remote -v

# 在 GitHub 上创建远程仓库 testgit
# ......

# 关联本地仓库和上述远程仓库，并将远程仓库的名字设置为 origin，方便后续操作
# 只需关联一次，后续提交分支等都不需要再执行了
git remote add origin https://github.com/cxy35/testgit.git

# 显示所有远程仓库，显示 origin https://github.com/cxy35/testgit.git
# git remote -v

# 推送本地仓库的指定分支（这里是 master）到远程仓库
# git push origin master

# 推送本地仓库的指定分支（这里是 master）到远程仓库，同时设置 upstream，方便后续操作
# -u 参数可以在推送的同时，将 origin 仓库的 master 分支设置为本地仓库当前分支的 upstream（上游），这个参数也只用在第一次 push 时加上，以后直接运行 git push 命令即可。同 git push --set-upstream origin master
git push -u origin master
# git push


# 列出所有本地分支，显示 master
# git branch

# 列出所有远程分支，显示 origin/master
# git branch -r

# 列出所有本地分支和远程分支，显示 master、remotes/origin/master
# git branch -a

# 新建一个分支，但依然停留在当前分支
# git branch fa

# 新建一个分支，并切换到该分支
# git checkout -b fa

# 切换分支，注意不同分支切换时工作区储藏 Stashing 问题
# git checkout fa

# 推送本地仓库的指定分支（这里是 fa）到远程仓库
# git push -u origin fa
```

## 12 场景2：从远程仓库获取

```bash
# 克隆一个远程仓库（默认是 master）到本地仓库，放在当前目录下的 testgit 目录中
git clone https://github.com/cxy35/testgit.git testgit

# 列出所有本地分支，显示 master
# git branch

# 列出所有远程分支，显示 origin/fa、origin/master
# git branch -r

# 列出所有本地分支和远程分支，显示 master、remotes/origin/fa、remotes/origin/master
# git branch -a

# 根据远程仓库的 fa 分支创建一个本地仓库的 fa 分支，创建完成之后进行切换
git checkout -b fa origin/fa

# 根据远程仓库的 fa 分支创建一个本地仓库的 fa 分支，只创建不切换
# git branch fa origin/fa

# fa 分支做文件夹/文件的增删改查操作
# ......

# 提交 fa 分支
# 注意由于 fa 分支就是从远程仓库克隆下来的，所以这里可以不添加 -u 参数
git push

# 从远程仓库更新
# git pull
```

---

- [Git 教程合集](https://mp.weixin.qq.com/s/S_wAUhlN1hqTjl4CwFS19Q)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)