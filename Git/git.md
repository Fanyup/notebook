# Git

maven是整个项目的管理，git实际是版本的管理。

## 什么是svn

**svn是集中式版本控制系统**，版本库集中放在中央服务器。缺点：

服务器单点故障；服务器挂了，你的代码就没了。容错性也不太好。对中文支持良好，图形界面。

哪一个服务器做svn服务器。（装SVN Server)

SSM搭建好框架的项目放到啊服务器上，windows系统就能完成。

有个url地址一般是公司内网的地址，svn管理员创建用户。

用户做各自的模块。

![chrome_hwON1qclZh.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_hwON1qclZh.png)

Git：分布式，去中心化思想，所有版本每个人的机器都有一个完整的版本库。

git是分布式管理系统，没有中央服务器。

开发员1和2每天写完代码提交到自己的本地仓库。有个共享版本库。

把内容按元数据方式存储.git

## 工作流程

![chrome_HZtPIQhApo.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_HZtPIQhApo.png)

你可以把你的idea项目目录当作你的工作区

![chrome_HeGRoeMkuu.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/chrome_HeGRoeMkuu.png)

创建版本库（本地仓库）

`git init`生成一个隐藏目录

![explorer_vo1BEX6RFG.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/explorer_vo1BEX6RFG.png)

添加文件，必须得是存在的`git add xxx.txt`由工作区提交到暂存区

提交`git commit xxx.txt`提交到本地仓库

退出和vim方式一样

- 初始在正常模式，按 i （小写，即按 I 键）进入插入模式，写文本；

- 写完后按 Esc 回到正常模式，输入 :wq 

显示）保存更改并退出编辑器。

修改默认编辑器为vim：

` git config –global core.editor vim `

## 查看文件变更，仓库状态

`git status (-s)`

## 提交修改文件

`git commit a.txt -m (注释)`

message

## 查看修改历史

就是查看日志

`git log`

简化`git log --oneline`

`git log --reverse`取反，时间正序显示

## 差异比较

`git diff`

（好像没装）

## 还原修改

## 删除文件

`git rm a.txt`

## 移动或重命名工作区文件

`git mv a.txt b.txt`

# 远程仓库github托管

SSH安全协议Secure Shell

非对称加密rsa-公钥加密、私钥解密

公钥给github，Git Bash生成私钥自己留着

`ssh-keygen -t rsa`

把.pub 给github （Settings-SSH...)

给它就可以将本地仓库内容同步到远程仓库repository了

**在该项目下：**

`git remote add [别名] git@github.com:(githubId)/远程仓库名.git`

`git push -u [别名] master`

master是版本号

## 从远程仓库克隆clone

`git clone git@github.com:(id)/项目名.git`

SSH

## 远程仓库取代码

fetch(不会自动merge合并)

pull（获取并合并到本地）

一般先用fetch 再用merge

`git fetch`

## 解决文件冲突

A改了版本并提交，B也改了版本，但此时Push会失败，因为此时的文件不是原先的文件了。

# 分支

## 查看&创建&切换

`git branch -v`查看分支

![mintty_3joXHUry8b.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/mintty_3joXHUry8b.png)

![ApplicationFrameHost_zi2xdf2jcD.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/ApplicationFrameHost_zi2xdf2jcD.png)

具体版本

## 合并

在当前分支下合并git-hot分支

`git merge hot-fix`

## 合并冲突

合并分支只会修改你的主分支。

当前文件指针HEAD，指向分支。

分支底下指的是具体版本。

## 认师，加入团队（获取权限）

获取邀请函

# IDEA部署

![idea64_p2tvnlLrs3.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_p2tvnlLrs3.png)

cmd--where git

# IDEA项目上传到github（含修改教程）

参见教程[本地idea上创建的项目上传到github仓库中](https://blog.csdn.net/AdminGuan/article/details/102633474)

## 关于上传超时问题

TIME OUT报错时间推迟到1分钟就OK了。多试几次

![idea64_VmwBOwgmlp.png](https://raw.githubusercontent.com/Fanyup/cloudimg/master/img/idea64_VmwBOwgmlp.png)
