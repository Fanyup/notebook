git config --global user.name [随便写]

git config --global user.name [随便写]

git init 初始化

git status 查看当前状态

git add [文件] 提交到暂存区

git commit -m "注释" 提交到本地库

git refolg 查看日志（版本号）

git branch -v 查看分支

git branch dev （创建名为dev的分支）

git checkout dev （将主线切换到dev分支下，指针指向改变了）

(可以在该分支/副本下修改文档内容)

（修改后内容需要重新add并上传本地库）

git merge dev（切换回主线后合并merger分支，把副本内容同步更新） 

git remote -v查看当前别名

git remote add [别名] [远程地址]

git push [别名] [分支] 本地库推送到远程库

git pull [别名] [分支] master 拉取

git clone 克隆

cd 切换/

默认别名origin

团队内成员拉取

全局设置乱码报错：git config --global core.quotepath false

报错：http链接上传请求超时和网络有关，可以用SSH密钥连接。

1. 注册成员信息

2. git bash -->git clone git@github.com:Fanyup/coursework.git

3. 添加本地用户信息，改全局变量乱码报错。

4. 切分支git checkout dev

5. 添加docx文档：vim xx-医疗器械信息系统.docx

6. 上传远程库，再切回master合并分支，上传远程库。
