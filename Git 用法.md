### Git 命令详解

​	1.在Windows上安装Git，(开始菜单里找到“Git”->“Git Bash”)

```shell
$ git config --global user.name "Your Name"
$ git config --global user.email "email@example.com"
---你的名字和Email地址
```

​	2.创建一个空目录，通过`git init`命令把这个目录变成Git可以管理的仓库

```shell
---在你选择的文件夹中点击右键，选择Git Bash Here 打开
#注意，在自己的仓库中点击Git Bash Here 可以直接对该仓库进行操作
$ git init
```

​	3.编写一个文件，放在这个目录下、子目录

```shell
---文件添加到仓库(执行上面的命令，没有任何显示,说明添加成功。)
$ git add <name.后缀>
$ git add .  //提交全部文件
---把文件提交到仓库：
$ git commit -m "<备注>"   
---查看改动结果
 $ git status
---查看具体文件改动
$ git diff <name.后缀> 
---查看提交历史记录
$ git log
---查看提交历史记录（简单显示）
$ git log --pretty=oneline
---返回上一个版本 （HEAD^,一个 ^ 代表一个版本；或者 HEAD~100 --HEAD表示当前版本）
$ git reset --hard HEAD^
 ` git reset --hard HEAD~1
---重新回到未来的版本
$ git reset --`这里写版本的id号，用log查看，一般写前7位`
---查询操作信息（包括含有的`id`）
$ git reflog
---丢弃工作区的修改
$ git checkout -- <name.后缀>
---撤销暂存区的修改
$ git reset HEAD <name.后缀>
---删除文件
$ git rm <name.后缀>
--- 推送到远程库
$ git push -u `库名`
$ git push `库名`
---克隆一个本地库
$ git clone `地址`
---关联远程库
$ git remote `库名`
---创建分支并切换（git checkout命令加上-b参数表示创建并切换，相当于以下两条命令）
$ git branch <name>      `创建分支`
$ git checkout <name>     `切换分支`
$ git checkout -b <name>    `--<name> 是分支名`
---查询分支
$ git branch 
---合并分支（合并某分支到当前分支）
$ git merge <name>
---删除分支
$ git branch -d <name>
$ git branch -D <name>    `强行删除`
---查看分支合并情况
$ git log --graph --pretty=oneline
---查询当前所有的文件
$ ls
---查看当前文件的具体内容
$ cat <name.后缀>
---修改文件的内容
$ vi <name.后缀>  `退出是先按Esc退出写入模式，在写入：wq,强行退出：wq!
---合并后的历史有分支，能看出来曾经做过合并
$ git merge --no-ff -m "<备注>" <name>
---储存当前的工作（不提交）
$ git stash
---查看储存的工作（上面的那种）
$ git stash list
---恢复储存的工作
$ git stash apply     `stash内容不删除`
$ git stash pop       `stash内容删除`
---查看远程库的信息
$ git remote
$ git remote -v         `详细的显示`
---推送分支
$ git push origin <name>
---从远程仓库创建本地分支
$ git checkout -b <name> origin/<name>
---提交本地分支到远程仓库
$ git push origin <name>
---抓取最新提交的分支
$ git pull 
$ git pull --set-upstream <name> origin/<name>  `设置本地分支与远程分支的链接`
---创建标签用于查找分支(切换到需要打标签的分支)
$ git tag <tagname>
$ git tag <tagname> <id>    `add merge提交打标签，它对应的commit id是6224937`
$ git tag         `查看所有的标签`
$ git show <tagname>   `查看标签详细内容`
$ git tag -a <tagname> -m "<备注>" <id>  `创建带有备注的标签`
$ git tag -s <tagname> -m "<备注>" <id>  `用私钥备注的标签`
$ git tag -d <tagname>      `删除标签`
$ git push origin <tagname>    `推送某个标签到远程`
$ git push origin --tags       `推送所有尚未推送的标签·
---查询历史提交的commit id
$ log --pretty=oneline --abbrev-commit
---配置别名
$ git config --global alias.<name> <name>    
`加上--global是针对当前用户起作用的，如果不加，那只针对当前的仓库起作用`
#例如 $ git config --global alias.st status    写的时候就写 $ git st 
#每个仓库的Git配置文件都放在.git/config文件中，要删除别名，直接把对应的行删掉即可
```

​	