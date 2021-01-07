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

#查看历史记录简略信息
git log --pretty=oneline

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

#版本回退：在git中HEAD代表当前的版本,上一个版本是HEAD^,上上一个版本是HEAD^^,向上100个版本为HEAD~100
git reset --hard HEAD^
git reset --hard <commit_id>

#如果工作区有修改，还没有add，想直接丢弃工作区的修改
git checkout -- <file>

#如果修改了内容，并且add到暂存区，想丢弃修改，分两步：1、回退暂存区 2、清空工作区
git reset HEAD <file>   #会回到上面的操作，然后想清空工作区，继续如上操作


```