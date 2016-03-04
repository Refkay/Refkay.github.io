---
layout:     post
title:      "Git的学习笔记"
date:       2016-02-16 19:00:00
author:     "Lemon Yang"
tags:
    - Git
---

### Git与SVN的简单比较

1. Git的版本控制是分布式的，而SVN的版本控制是集中式的。集中式的版本控制系统只有中央服务器才具备版本控制的数据库，用户主机和中央服务器的依赖度高。而分布式的版本控制系统每一个用户主机上都具备本地的版本控制的数据库。两者的工作差别可以参考下面的两张图。
![](/img/in-post/jizhongshi.png)
![](/img/in-post/fenbushi.png)
2. SVN存储的是版本间的文件差异，而Git存储的是当前版本的全部内容。两者的差别可以参考下图。
![](/img/in-post/svn-save.png)
![](/img/in-post/git-save.png)

### Git的简单配置

**Git config配置文件的设置：**

* git config --global user.name YOUR_NAME 设置全局的git操作用户名称
* git config --global user.email YOUR_EMAIL 设置全局的git操作用户的邮箱
* git config --global --add user.name YOUR_NAME 添加配置属性值
* git config --get user.name  获取当前指定的键值对应的内容
* git config --global --list 查询当前global级别的配置内容 
* git config --global --unset user.name 取消设置某个键值内容，当一个键对应多个值的时候，需要指定需要撤销的具体值

>git的的配置具备三个级别，分别是--local(针对当前仓库),--global（针对当前用户）,--system（针对系统）

**Git 为子命令和参数起别名的方式：**

* git config --global alias.co checkout 其中co是要设置的别名的名称，checkout是对应的子命令名称
* git config --gloabal alias.lol "log --online" 带参数的子命令的的别名设置


### Git基本的工作流程

1. 创建一个git仓库，使用到到命令
	
	* git init [目录名]  使用指定的目录名创建一个目录，并且在目录中初始化一个空的git仓库
	* git init 在当前目录下创建一个git仓库
2. 当我们在git仓库所在目录中添加了新建了文件之后，我们可以将新增的文件放到git仓库的暂存区。

	* git add [文件名] 将制定的文件放进暂存区中
	* git add . 将当前全部尚未放进缓存区中的文件全部放进缓存区中。
	* git add -A 作用同上。
	* git add --all 作用同上

3. 此时，可以将暂存区中的文件提交到git的历史仓库中。
	
	* git commit -m "message" 将暂存区的文件提交到历史仓库中，后面加上提交到附加信息。
	
4. 删除文件

	* git rm [文件名] 将指定的文件从工作区和暂存区中删除
	* git rm --cached [文件名] 将暂存区中该文件的引用删除(但是工作区中该文件并没有被移除)
5. 重命名文件

	* git mv [原文件名][新文件名] 
6. 添加.gitignore文件。在项目目录中会存在一些ide工具生成的中间文件等无用文件，我们使用git托管项目的时候，可以忽略这些文件。这时候可以在项目的根目录中添加.gitignore文件，并在里面输入我们的匹配条件即可。例如：

	* build/ 忽略根目录下的build目录的全部文件
	* ../build/ 忽略所有根目录及其子目录下的build目录
	* *.iml 忽略所有以.iml结尾的文件

关于上面提及到的工作区、暂存区和历史仓库三者的关系，可以参考下图

![](/img/in-post/git-dir.png)

### Git本地分支与合并

>未完待续。。。




