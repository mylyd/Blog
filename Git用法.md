### [笔记主目录](https://mylyd.github.io/myhome.html)

### Git 命令详解

​	1.在Windows上安装Git，(开始菜单里找到“Git”->“Git Bash”)

```shell
---你的名字和Email地址
$ git config --global user.name "Your Name"			#设置name
$ git config --global user.email "email@example.com"	#设置Email

---配置别名
$ git config --global alias.<name> <name>    
#加上--global是针对当前用户起作用的，如果不加，那只针对当前的仓库起作用`
#例如 $ git config --global alias.st status    写的时候就写 $ git st 
#每个仓库的Git配置文件都放在.git/config文件中，要删除别名，直接把对应的行删掉即可
```

​	2.创建一个空目录，通过`git init`命令把这个目录变成Git可以管理的仓库

```shell
---在你选择的文件夹中点击右键，选择Git Bash Here 打开
#注意，在自己的仓库中点击Git Bash Here 可以直接对该仓库进行操作
$ git init		#通过git init命令把这个目录变成Git可以管理的仓库
##注意，当前的路径一定要在设置的Git本地仓库路径里面
--如果是直接从远程拉下来的，需要在`init`好的文件里面直接clone <html地址>
```

​	 3.Git 使用命令

##### 			文件添加到缓存区(执行上面的命令，没有任何显示,说明添加成功。)
```shell
$ git add <name.后缀>			#提交<name>文件到缓存区
$ git add .  				#提交全部文件到缓存区
```
##### 			把文件提交到仓库：
```shell
$ git commit -m "<备注>"   	#提交全部git add 了的文件到仓库(本地)
```
#####   			查看改动结果
```shell
 $ git status				#查看最后一次的改动信息
```

##### 			查看具体文件改动
```shell
$ git diff <name.后缀> 	   #查看具体文件改动
$ git reflog				#查询操作信息（包括含有的`id`）
```

##### 			查看提交历史记录
```shell
$ git log						#查看历史数据
$ git log --pretty=oneline   	#查看提交历史记录（简单显示）
$ git log --graph --pretty=oneline		#查看分支合并情况
$ log --pretty=oneline --abbrev-commit		#查询历史提交的commit id
```
##### 			返回上一个版本 （HEAD^,一个 ^ 代表一个版本；或者 HEAD~100 --HEAD表示当前版本）
```shell
$ git reset --hard HEAD^
$ git reset --hard HEAD~1		#第二种写法
$ git reset --<id	>			#重新回到未来的版本--写版本的id号,般写前7位
$ git reset HEAD <name.后缀>		#销暂存区的修改
```
##### 			删除文件
```shell
$ git rm <name.后缀>       
#如果要删除远程库的文件、可以在本地文件中删除，然后push，一样具有删除效果
```
##### 			推送到远程库
```shell
$ git push -u `库名`		#保持第一次同步（不推荐）
$ git push `库名`			#推送到远程（推荐）
$ git push -all `库名`  	 #强行推送
$ git push origin --delete <branchName>  #删除远程分支
$ git push origin <tagname>    			#送某个标签到远程`
$ git push origin --tags      			#推送所有尚未推送的标签·
$ git push origin <name>			#推送分支到远程
```
##### 			从远程克隆到本地
```shell
$ git clone `地址`  #这里的地址在你远程仓库里面
##--具体：Clone or download 按钮下的 https://github.com/****/*****.github.io.git
```

##### 			关联远程库与查询远程库信息
```shell
$ git remote `库名`   #关联远程库
$ git remote          #查询远程库的库名，一般为origin
$ git remote -v         #查询远程库详细类容
```
##### 			创建分支并切换（git checkout命令加上-b参数表示创建并切换，相当于以下两条命令）
```shell
$ git checkout <name>     			#切换分支
$ git checkout -b <name>   			#创建并切换到新创建的分支，<name>是分支名
$ git checkout -- <name.后缀>		   #丢弃工作区的修改
$ git checkout -b <name1> origin/<name2>		
#从远程仓库创建本地分支(name1远程分支名、name2本地分支名；两者一般一样)
```
##### 			Branch(分支)语句的应用(分支的增、删、查、改)
```shell
$ git branch <name>      #创建分支
$ git branch 			#查看当前位于哪一个分支
$ git branch -a  		#查询远程分支
$ git branch -d <name>	#删除分支
$ git branch -D <name> 	#强行删除分支
$ git merge <name>		#合并分支（合并某分支到当前分支）
$ git merge --no-ff -m "<备注>" <name>		#合并后的历史有分支，能看出来曾经做过合并
```
##### 			查询当前目录下所有的文件	
```shell
$ ls
```
##### 			查看当前文件的具体内容
```shell
$ cat <name.后缀>
```
##### 			具体修改文件的内容
```shell
$ vi <name.后缀> 		 #退出是先按Esc退出写入模式，在写入：wq,强行退出：wq!
```
##### 			储存当前的工作（不提交）
```shell
$ git stash
$ git stash list      #查看当前储存(不提交)的工作
$ git stash apply     #恢复储存的工作--`stash内容不删除`
$ git stash pop       #恢复储存的工作--`stash内容删除`
```
##### 			抓取最新提交的分支
```shell
$ git pull 
$ git pull --set-upstream <name> origin/<name>  		#设置本地分支与远程分支的链接
```
##### 			创建标签用于查找分支(切换到需要打标签的分支)
```shell
$ git tag <tagname>			 #创建标签 ，例:git tag "v2.1"
$ git tag <tagname> <id>  	  #add merge为历史提交打标签，它对应的commit id是6224937
$ git tag         			#查看所有的标签
$ git show <tagname>  		 #查看标签详细内容
$ git tag -a <tagname> -m "<备注>" <id>  		#创建带有备注的标签
$ git tag -s <tagname> -m "<备注>" <id>  		#用私钥备注的标签
$ git tag -d <tagname>      				#删除标签
```