## git命令
```shell
#初始化一个git仓库
git init

#设置当前仓库的用户名和email地址
git config --global user.name "Your Name"
git config --global user.emial "emial@example.com"

#查看仓库的当前状态(增删改等)
git status

#查看difference
git diff

#查看历史记录
git log
git log --pretty=oneline    #查看历史记录简略信息
git log --graph #查看分支合并图

#命令历史,记录每一次操作，包括reset
git reflog

#三个概念  工作区(当前操作)>>>>>暂存区(add以后)>>>>>仓库(commit以后)

#查看工作区和暂存区的区别
git diff

#查看暂存区和仓库的区别
git diff --cached

#查看工作区和仓库的区别
git diff HEAD <filename>

#当前操作添加到暂存区
git add <file>
git add .
git add -A

#将暂存区提交到版本库
git commit -m <commit_describe>

#将本地版本仓库提交到远程库
git push
git push -u origin master   #第一次提交时候加-u会添加和远程库分支的关联

#版本回退：在git中HEAD代表当前的版本,上一个版本是HEAD^,上上一个版本是HEAD^^,向上100个版本为HEAD~100
git reset --hard HEAD^
git reset --hard <commit_id>

#如果工作区有修改，还没有add，想直接丢弃工作区的修改。换言之就是：用版本库的直接替换工作区的版本
git checkout -- <file>

#如果修改了内容，并且add到暂存区，想丢弃修改，分两步：1、回退暂存区 2、清空工作区
git reset HEAD <file>   #会回到上面的操作，然后想清空工作区，继续如上操作

#关联远程库
git remote add origin <xxxxxxx>

#分支相关操作

#查看分支
git branch
git branch -a   #所有分支，包含远程的分支

#创建分支
git branch <name>

#切换分支
git switch <name>   #新版本命令
git checkout <name>

#创建加切换分支
git switch -c <name>
git checkout -b <name>

#删除分支
git branch -d <name>
git branch -D <name>    #强制删除分支

#合并某分支到当前分支（默认是Fast forward模式；这种模式下删除分支，会丢掉分支信息）
git merge <name>
#分支合并（强制禁用Fast forward模式；merge时生成新的commit）
git merge --no-ff -m <commit-message> <branch-name>
#拉取某个指定提交的代码
git cherry-pick <commit-id>

#工作现场存储（当临时要改一个bug，但是当前还没改完不可提交，可以先暂存）
git stash
#查看暂存列表
git stash list

#两种方式恢复当前暂存的工作区
#第一种
git stash apply #恢复后需要手动删除
git stash drop  #删除
#第二种
git stash pop   #恢复的同时吧stash删除

#恢复指定的stash
git stash apply stash@{0}

#将本地分支和远程分支关联
git branch --set-upstream-to <branch-name> origin/<branch-name>

#在本地创建和远程分支对应的分支
git checkout -b branch-name origin/branch-name

#rebase操作可以把本地未push的分叉提交历史整理成直线
git rebase

#创建标签
#打一个新标签
git tag <name>

#查看标签
git tag

#查看标签信息
git show <tagname>

#指定标签信息
git tag -a <tagname> -m "blablabla..."

#推送一个本地标签
git push origin <tagname>

#推送全部未推送过的本地标签
git push origin --tags

#删除一个本地标签
git tag -d <tagname>

#删除一个远程标签
git push origin :refs/tags/<tagname>
```