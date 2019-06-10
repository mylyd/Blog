## 一个Git配置遇到坑的记录笔记

一个上午就这样被git的各种坑给耽误了，怪就怪在好久没用了，好吧一步一步的去解决坑吧，然后，选个时间吧这些个坑记录一下，下次拿出来看看，美滋滋~~~

- ### 第一步（当你重新配置Git环境的时候，并且与你的远程需要连接的同时）

  > 1、下载Git并安装（Windows版本），[Git下载 ](https://git-scm.com/downloads)
  >
  > 2、建立一个文件夹 *English*命名，在文件目录下右键鼠标选择Git Bash Here
  >
  > ```java
  > $ git config --global user.name "Your Name"
  > $ git config --global user.email "email@example.com"
  > ```
  >
  > 设置好用户名跟邮件地址。

- ### 第二步

  > 1、设置SSH
  >
  >  ``` java
  > $ ssh-keygen -t rsa -C "your@email.com"（请填你设置的邮箱地址）>>一直按回车就好
  >  ```
  >
  > 然后会跳出一堆话。输入命令：yes回车,就行了
  >
  > 配置后会在C盘用户目录下生成一个*.ssh* 文件，然后系统会自动在.ssh文件夹下生成两个文件，id_rsa和id_rsa.pub，用记事本打开id_rsa.pub。将全部的内容复制
  >
  > 2、打开https://github.com/，登陆你的账户，进入设置
  >
  > ![img1](<https://raw.githubusercontent.com/mylyd/mylyd.github.io/master/img/img1.png>)
  >
  > 点击add ssh key，ok！

  

- ### 下面具体看最最最最坑人的地方

> ![](C:\Users\mayn\Desktop\indexzhuxi\mylyd.github.io\img\img2.png)
>
> 看到没有，即使你整了ssh还是会出现这种问题，不要慌，这种情况1是可能你网炸了，2呢就是掉坑里了
>
> ```java
> $ ssh -T git@github.com 
> ```
>
> 输出上面这个试一下，
>
> ```java
> ssh: connect to host github.com port 22: Connection timed out
> ```
>
> 这是输出结果，ok又是一个坑，这个时候我注意到了图中信息里的 *port 22*，把这里的22换成443端口在试一次，命令如下：
>
> ```java
> $ ssh -T -p 443 git@ssh.github.com
> ```
>
> 成功,不要高兴的太早，这只是测试用的~~
>
> ##### 下面上大菜
>
> ```java
> $ git config --local -e      //编辑本地git配置
> ```
>
> ![](C:\Users\mayn\Desktop\indexzhuxi\mylyd.github.io\img\img3.png)
>
> 这里修改这个是因为有些电脑使用ssh时会出现问题，改成https就不会了，没找到ssh的解决办法目前只能改成能用https的办法了，经过百度，嘿嘿~~（注意一下，这里进去是不能直接插入的，需要按一下insert按键）
>
> 将      url = git@github.com:你的用户名/仓库名.git
>
> 改为  url = https://github.com/你的用户名/仓库名.git
>
> 然后esc   :wq保存修改回车

然后你以为结束了吗？？？？ 不可能的

```java\
$ git push origin master 
```

你会发现等你输入好你的帐户密码后发现

```java
Password for 'https://git@github.com': 
remote: Invalid username or password.
fatal: Authentication failed for 'https://git@github.com/eurydyce/MDANSE.git/'
```

这nm又是什么坑啊，日哦。。。。

![](C:\Users\mayn\Desktop\indexzhuxi\mylyd.github.io\img\img5.png)

然后，继续百度吧，看了国外的一个论坛，找到了原因，说是什么启动了2FA，需要输入个人令牌，所以那个密码输入的是你的令牌（这里还有个坑，后面说，呜呜呜呜~~）

个人令牌生成：到你的github主页 > Settings > Developer settings > Personal access token > Generate a new token

这时候会出现一个权限勾选表，上面的一个title 随便写写，下面权限根据需求勾选，然后生成的那个老长老长的一定要复制好，然后找个小记事本记下来，因为搞到这里我发现每次 push的时候都尼玛需要输入这个，晕了~

![](C:\Users\mayn\Desktop\indexzhuxi\mylyd.github.io\img\img6.png)

到这里就结束了，ok能push了。

![](C:\Users\mayn\Desktop\indexzhuxi\mylyd.github.io\img\img4.png)

不容易啊，写个东西记录一下，避免下次入坑