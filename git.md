## git命令
```markdown
//初始化一个git仓库
git init

//设置当前仓库的用户名和email地址
git config --global user.name "Your Name"
git config --global user.emial "emial@example.com

//查看仓库的当前状态(增删改等)
git status

//查看difference
git diff

//查看历史记录
git log

//查看历史记录简略信息
git log --pretty=oneline

//命令历史,记录每一次操作，包括reset
git reflog

//版本回退，在git中HEAD代表当前的版本,上一个版本是HEAD^,上上一个版本是HEAD^^,向上100个版本为HEAD~100
git reset --hard HEAD^
git reset --hard commit_id

```