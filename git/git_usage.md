### git的常用的命令总结

> 从事软件研发的人员，日常的工作主要是围绕着代码的编写展开，团队协作的场景下肯定离不开代码版本控制工具，目前主流的版本控制工具主要有两种svn和git。前者采用集中式的管理方式，后者采用分布式。鉴于目前使用git的开发团队数量较多，教程也从git的基本命令展开讲解，同时会附带一些实际的操作场景的命令的使用。可能有的人会产生疑问：为什么在有图形IDE开发工具的基础上，我们还需要掌握这些“原始”的git命令呢？如果你是抱着这样想法的研发人员，那么我先讲一个故事吧。从前有一个叫张三的小伙，好不容易在这个行业积攒了三年的工作经验，最经换了一份新的工作，到了新的公司新的部门，代码版本管控没有采用传统的gitlab、github，而是采用google开源的gerrit，在新的工具面前原来的IDE图形化操作的三板斧彻底失效了，加上来到新的环境，脸皮比较单薄，遇到问题始终不愿意咨询他人，虽然本地的代码已经写好了，因为不会解决冲突的缘故以至于开发周期频频delay，基于试用期的表现最终在试用期到达的前一周被主管劝退了，故事中的主人公张三是一个真实职场人士，他的经历也是真实的，因为git上命令的掌握不牢固，最终导致了劝退的结果。

---

####  git实现原理图

![原理图](https://img-blog.csdnimg.cn/img_convert/7c1a27e6ca241fbc2344af03843728c4.png)

---

#### 常用命令

- 配置相关

git config --list # 查看config配置

git config user.name #查看配置用户名

git config user.email #查看配置的邮箱

git config --global user.name “xxx” #设置git用户名

git config --global user.eamil “xxx” #设置git邮箱

git credential-manager uninstall #清除缓存的用户名和密码

---

- 仓库操作

git init #新建git仓库

git clone #克隆仓库

---

- 增加/删除文件控制

git add [file1] [file2] #添加文件到暂存区

git add . #添加所有改动文件到暂存区

git rm [file1] [file2] ＃删除工作去文件，并且将删除放入暂存区

git rm --cached [file] ＃停止追踪指定文件，该文件会保留在保存区

git mv [file-original] [file-renamed] #改名文件，并将文件改名放入暂存区

---

- 代码提交

git commit -m [message] #提交文件从暂存区到仓库

git commit [file1] [file2] … -m [message] #提交指定文件到仓库区

git commit --amend -m [message]  #将改动追加到最后一次提交

---

- 分支操作

git branch #列出所有本地分支

git branch -r #列出所有远程分支

git branch -a #列出所有分支

git branch [branch-name] #新建一个分支

git checkout  -b  [branch-name] #从当前分支切换到指定分支

git branch - #切换到上一个分支

git branch --track [branch] [remote-branch] #新建一个分支与指定的远程分支建立追踪关系

git branch --set-upstream [branch] [remote-branch] #建立追踪关系，在现有分支和指定的远程分支之间

git merge [branch] #合并指定分支到当前分支

git cherry-pick [commit] #选择一个commit，合并到当前分支

git branch -d [branch-name] #删除分支

git branch -D xxx  #删除本地分支无论是否合并

git push origin --delete [branch-name] #删除远程分支

---

- 查看信息

git status #查看暂存区的状态

git log #查看当前分支的提交记录

git diff [commit-id] #查看工作区和最新一次的区别

git show #显示文件的改动

---

- 远程操作

git fetch [remote] #下载远程仓库的所有改动

git fetch origin [branch] #拉取远程指定分支到本地

git remote -v #显示所有远程仓库

git remote add [name] [url] #增加一个远程仓库

git pull [remote] [branch] #拉取远程仓库变动并并与本地仓库合并

git push [remote] [branch] #上传本地指定分支到远程仓库

---

- 撤销代码提交

git checkout [file] #撤销对文件的改动

git checkout . #撤销所有文件的改动

git reset [file] #撤销文件的版本控制

git reset [commit-id] # 重置到指定提交点，之后的提交全部消失

git reset --hard HEAD^n #将最新的代码提交回退n次提交，不保留代码

git reset --hard HEAD^n #将最新的代码提交回退n次提交，保留n-1次的代码变动

git revert [commit-id] #新建一次提交来撤销指定某次的提交

---

- 暂存区操作

git stash #将本地的改动暂存起来

git stash save “xxx” #将本地的改动暂存起来，并添加记录信息方便查找

git stash pop #应用最近一次的暂存的修改，并删除暂存的记录

git stash apply # 应用某个存储,但不会把存储从存储列表中删除，默认使用第一个存储,即 stash@{0}，如果要使用其他个，git stash apply stash@{$num} 。

git stash list # 查看 stash 有哪些存储

git stash clear #删除所有缓存的 stash

---

- tag相关操作

git tag  #查看tag

git tag -d [tagName]   #删除tag

git tag -a V1.0 55d8e71fc7d0b8cefbb4cbee339beb9d987d9b81 -m '正式版本'  #给指定的提交打tag

git push origin V1.0  # tag同步到远程服务器

git push origin --tags  #通过所有tag到远程服务器

git ls-remote --tags  #查看远程服务器的标签

---

- fetch操作

git fetch -p origin  #清除本地分支缓存

git fetch origin [remote-branch-name]  #拉取仓库中远程指定分支到本地并创建本地分支名称


#### 实践操作

- 本地创建的文件夹推送到新建远程仓库

git init
git remote add origin 远程仓库地址
git add .
git commit -m “Initial commit”
git push -u origin master

---

- 拉取远程分支代码到本地

git checkout -b dev(本地分支名称) origin/dev(远程分支名称)

git pull origin dev(远程分支名称)

---
