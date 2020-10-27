### Git常用命令

#### 用户和邮箱

```bash
 -- 设置全局的用户和邮箱
 $ git config --global user.name "renyang"
 $ git config --global user.email "1261973604.qq.com"
 
 --查看设置的全局用户和邮箱：
 $ git config --global --list
```

#### 仓库相关

```bash
-- 新建目录并将目录变为git可以管理的仓库
$ mkdir learngit	创建目录
$ cd learngit	进入目录 
$ pwd	显示当前目录
$ git init	将当前目录变为git可以管理的仓库

-- 查看仓库当前状态
$ git status    
-- 查看修改内容
$ git diff	一般在git status命令看到仓库文件有修改后，才执行该命令查看修改的内容
-- 把文件添加到版本库：
$ git add [文件名]	第一步，添加文件
$ git commit -m  "提交的说明"	第二步，完成提交
```

#### 版本回退

```bash
$ git log	显示当前版本到当前版本之前的提交日志
$ git log --pretty=oneline	简化git log的输出信息 
$ git reset --hard HEAD^	HEAD表示当前版本，HEAD^表示上个版本，HEAD^^表示上上个版本
$ git reset --hard HEAD~数字	数字代表回退到之前的第几个版本，如数字为1，代表回退到上1个版本
$ git reset --hard [commit id]	跳转到该commit id的版本
$ git reflog	查看历史的命令，以便确定要回到未来的哪个版本
```

#### 撤销修改

```bash
$ git checkout -- [文件名]         --丢弃对工作区的修改
     该命令有两种情况：
      (1) 文件修改后还没执行add命令，即还未放到暂存区。现在撤销修改就回到和版本库一摸一样的状态。
      (2)  文件已经add到了暂存区后，又做了修改，现在撤销修改就回到add到暂存区的状态。
        总之：撤销修改就是让文件回到最近一次git commit 或者 git add 的状态。
        
$ git reset HEAD [文件名]	--把暂存区的修改撤销掉，重新放回工作区。这条命名适用于修改的文件add到暂存区,且未做二次修改和								 commit操作时使用。 
    场景：修改了一个文件内容，add到了暂存区且又对文件进行了修改。需要用命令将文件恢复到最初状态。
    命令执行顺序：
        1. $ git checkout -- [test.txt]	--先恢复到刚add到暂存区的状态。
     	2. $ git resrt HEAD [文件名]	--接着撤销对暂存区的修改，重新放回工作区。
		3. $ git checkout -- [文件名]	--最后再撤销修改，回到和版本库最初状态。(因为未进行提交，即版本库并未改变)
注意：若不仅 add到了暂存区还commit到了版本库，需要先进行版本回退。
```

#### 删除文件

```bash
$ git rm [文件名]	--用于从版本库中删除文件
$ git commit -m "描述信息"	--提交删除命令

-- 当误删文件时(还未commit到版本库中)，可用以下命令从版本库回复
$ git checkout -- [文件名]	--用版本库里的版本替换工作区的版本，无论工作区是修改还是删除，都可以“一键还原”
```

#### 创建本地SSH key 并绑定本地电脑和github(这里用的gitHub)

​	  (1)首先在用户主目录下，看看有没有已经生成好的.ssh目录，里面有id_rsa和id_rsa.pub两个文

​          件，若有了则直接跳到下一步。如果没有则用下面命令创建SSH key：

​      $ ssh-keygen -t rsa -C "[邮箱地址]" 

​          顺利的话会生成.ssh目录，里面有上面说的那两个文件，这两个就是SSH key的密钥对，id_rsa  

​          是私钥，不能泄露出去，id_rsa.pub是公钥，可以放心告诉别人。	

​      (2)登陆gitHub,完成绑定 

​          打开“Account settings”，“SSH Keys”页面：

​          然后，点“Add SSH Key”，填上任意Title，在Key文本框里粘贴id_rsa.pub文件的内容：

![img](E:\Program Files (x86)\有道云笔记\文件\qq7E958A6792CAC79AB02DA6F48AF48278\ac4af4dc65e04693a8841cd7fa656f0b\clipboard.png)

​      	 点“Add Key”，你就应该看到已经添加的Key：

​     

![img](E:\Program Files (x86)\有道云笔记\文件\qq7E958A6792CAC79AB02DA6F48AF48278\a17cd06c9a164ccc85cf15f5e2d92924\clipboard.png)

#### 添加远程仓库并向远程仓库进行数据推送

```bash
(1) 在gitHub上创建一个新的空仓库。(不做操作介绍)
(2) 通过命令将本地仓库与gitHub上的远程仓库进行关联。
	$ git remote add [远程库的代名称，可以自己定义] [gitHub上远程仓库的地址] 
(3) 第一次将master分支上的数据从本地推送到远程仓库上。
	$ git push -u [远程库的代名称] master	-- 第一次推送master分支是加了-u参数，Git不但会把本地的master分支内容推送的远											程新的master分支，还会把本地的master分支和远程的master分支关联起来。
(4) 此后本地提交到远程仓库就可以使用下面的命令：
	$ git push [远程库代名称] master	--master部分指定同步到哪一个分支上
```

#### 将远程仓库中的最新代码同步到本地

```bash
Git提供了两种命令来完成此功能，分别是fetch和pull,区别及用法如下：
	1.$ git fetch origin master	--执行这个命令后，就会将远程版本库上的代码同步到本地，不过同步下来的代码并不会合并到任何分									支上去，而是会存放在到一个origin/master 分支上,之后再调用 merge 命令将 
								  origin/master分支上的修改合并到主分支上即可 git merge origin/master
	2.$ git pull [远程仓库代名称] master	--pull 命令则是相当于将 fetch 和 merge 这两个命令放在一起执行了，它可以从远程											版 本库上获取最新的代码并且合并到本地
```

#### 远程仓库相关

```bash
 --  克隆远程仓库到本地
 $ git clone [远程仓库的地址]
 
 -- 查看远程仓库信息
$ git remote -v	--列出当前程序对应的所有远程版本仓库信息，含仓库名和仓库地址
$ git remote	--只是单纯的列出所有远程数据库的名字，不会展示远程仓库的地址 
$ git remote show [远程仓库的代名称]	--显示出远程仓库的详信息

-- 重命名远程仓库
$ git remote rename [原代名称] [现代名称]

-- 删除远程仓库(解除与远程库的关系)
$ git remote rm [原代名称]   
```

