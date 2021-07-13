$ git config --global user.name "caozhenguo-coder"
$ git config --global user.email "15270275057@sina.cn"

本地仓库:
git init		//新建&初始化本地仓库
git clone		//克隆一个远程仓库

提交&回滚：
git add	test.txt/git rm 	#提交至暂存区/从Git暂存区域的追踪列表中移除
git commit -m "描述"/git reset    #提交至仓库/从Git仓库回滚至暂存区
git push	   	#推送至远程仓库	

将该文件
[root@linuxprobe linuxprobe]# git rm --cached database
Git不像其他版本控制系统那样跟踪文件的移动操作，如果要修改文件名称，则需要使用git mv命令

============================
查看数据：
$ git config --list
git status|log|show
[root@linuxprobe linuxprobe]# git log -2    #只想看最近几条记录
[root@linuxprobe linuxprobe]# git log -p -1 #仅查看最近一次的差异
	--stat参数来简要的显示数据增改行数，这样就能够看到提交中修改过的内容、对文件添加或移除的行数，并在最后列出所有增减	行的概要信息（仅看最近两次的提交历史）：
[root@linuxprobe linuxprobe]# git log --stat -2
	还有一个超级常用的--pretty参数，它可以根据不同的格式为我们展示提交的历史信息，比如每行显示一条提交记录：
[root@linuxprobe linuxprobe]# git log --pretty=oneline
	以更详细的模式输出最近两次的历史记录：
[root@linuxprobe linuxprobe]# git log --pretty=fuller -2
	还可以使用format参数来指定具体的输出格式，这样非常便于后期编程的提取分析哦，常用的格式有：

%s	提交说明。
%cd	提交日期。
%an	作者的名字。
%cn	提交者的姓名。
%ce	提交者的电子邮件。
%H	提交对象的完整SHA-1哈希字串。
%h	提交对象的简短SHA-1哈希字串。
%T	树对象的完整SHA-1哈希字串。
%t	树对象的简短SHA-1哈希字串。
%P	父对象的完整SHA-1哈希字串。
%p	父对象的简短SHA-1哈希字串。
%ad	作者的修订时间。

	另外作者和提交者是不同的，作者才是对文件作出实际修改的人，而提交者只是最后将此文件提交到Git版本数据库的人。
	查看当前所有提交记录的简短SHA-1哈希字串与提交者的姓名：
[root@linuxprobe linuxprobe]# git log --pretty=format:"%h %cn"d97dd0b Liu Chuan

==============================
还原数据：
	可以用git reflog命令来查看所有的历史记录：
[root@linuxprobe linuxprobe]# git reflog
	找到历史还原点的SHA-1值后，就可以还原文件了，另外SHA-1值没有必要写全，Git会自动去匹配：x
[root@linuxprobe linuxprobe]# git reset --hard 5cee15b
	如是只是想把某个文件内容还原，就不必这么麻烦，直接用git checkout命令就可以的
==================================

21.2.6 管理标签

	在Git中打标签非常简单，给最近一次提交的记录打个标签：
[root@linuxprobe linuxprobe]# git tag v1.0
	查看所有的已有标签：
[root@linuxprobe linuxprobe]# git tag
	查看此标签的详细信息：
[root@linuxprobe linuxprobe]# git show v1.0
	还可以创建带有说明的标签，用-a指定标签名，-m指定说明文字：
[root@linuxprobe linuxprobe]# git tag v1.1 -m "version 1.1 released" d316fb
	删除标签
[root@linuxprobe linuxprobe]# git tag -d v1.0

===================================
21.3 管理分支结构

	首先创建分支：
[root@linuxprobe linuxprobe]# git branch linuxprobe
	切换至分支：
[root@linuxprobe linuxprobe]# git checkout linuxprobe
	查看当前分支的情况（会列出该仓库中所有的分支，当前的分支前有＊号）：
[root@linuxprobe linuxprobe]# git branch
	使用"git merge"命令来将指定的的分支与当前分支合并：
[root@linuxprobe linuxprobe]# git merge linuxprobe
	确认合并完成后，就可以放心地删除linuxprobe分支了：
[root@linuxprobe linuxprobe]# git branch -d linuxprobe

21.3.3 内容冲突
	创建一个新分支并切换到该分支命令：
[root@linuxprobe linuxprobe]# git checkout -b linuxprobe

21.4 部署Git服务器
	首先分别在Git服务器和客户机中安装Git服务程序：
[root@linuxprobe ~]# yum install git
	然后创建Git版本仓库，一般规范的方式要以.git为后缀：
[root@linuxprobe ~]# mkdir linuxprobe.git
	修改Git版本仓库的所有者与所有组：
[root@linuxprobe ~]# chown -Rf git:git linuxprobe.git/
	初始化Git版本仓库：
[root@linuxprobe ~]# cd linuxprobe.git/
[root@linuxprobe linuxprobe.git]# git --bare init
	其实此时Git服务器就已经部署好了，但用户还不能向你推送数据，也不能克隆你的Git版本仓库，因为我们要在服务器上开放至少	一种支持Git的协议，比如HTTP/HTTPS/SSH等，现在用的最多的就是HTTPS和SSH，我们切换至Git客户机来生成SSH密钥：
[root@linuxprobe ~]# ssh-keygen 
	将客户机的公钥传递给Git服务器：
[root@linuxprobe ~]# ssh-copy-id 192.168.10.10
	此时就已经可以从Git服务器中克隆版本仓库了（此时目录内没有文件是正常的）：
[root@linuxprobe ~]# git clone root@192.168.10.10:/root/linuxprobe.git
	初始化下Git工作环境：
[root@linuxprobe ~]# git config --global user.name "Liu Chuan"
[root@linuxprobe ~]# git config --global user.email "root@linuxprobe.com"
[root@linuxprobe ~]# git config --global core.editor vim
	但是这次的操作还是只将文件提交到了本地的Git版本仓库，并没有推送到远程Git服务器，所以我们来定义下远程的Git服务器吧：
[root@linuxprobe linuxprobe]# git remote add server root@192.168.10.10:/root/linuxprobe.git

==================================================================
报没有权限错误解决办法：
The requested URL returned error.403 Forbidden while accessing!

vi .git/config

[remote "origin"]
	url = https://github.com/用户名/仓库名.git
修改为：
[remote "origin"]
	url = https://用户名:密码@github.com/用户名.git

出现错误的原因是github中的README.md文件不在本地代码目录中。
也就是说我们需要先将远程代码库中的任何文件先pull到本地代码库中，才能push新的代码到github代码库中。
使用如下命令：git pull --rebase origin master
然后再进行上传: git push -u origin master
==================================================================

$ git remote add origin url


GITHUB搭建个人网站：

   访问路径：
	https://用户名.github.io/仓库名

   搭建步骤：




