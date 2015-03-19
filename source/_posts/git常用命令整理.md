title: git常用命令整理
date: 2015-03-05 21:01:22
updated: 2015-03-10 14:05:01
categories: git
tags: [git,git命令]
toc: true
---
好记性不如烂博客,整理了下一些常用的命令
之后有用到更多的再慢慢更新



###配置全局变量

```
  $ git config --global user.name "Your Name"
  $ git config --global user.email "email@example.com"
```

`$ git config --global color.ui true`显示颜色
<!-- more -->
####配置别名

```
    $ git config --global alias.st status
    $ git config --global alias.co checkout
    $ git config --global alias.ci commit
    $ git config --global alias.br branch
```



`$ git config --global alias.last 'log -1' `最近一次提交
配置的内容都保存在`.git/config`中的


###查看日志

`$ git log --pretty=oneline` 按Q退出
`$ git log --stat -n` `--stat`显示简要的增改行数统计,每次提交文件的变更统计`-n`前n条
`$ git log --pretty=format:" "`控制显示的记录格式，常用的格式占位符写法及其代表的意义如下：

```
选项	 说明
%H	提交对象（commit）的完整哈希字串
%h	提交对象的简短哈希字串
%T	树对象（tree）的完整哈希字串
%t	树对象的简短哈希字串
%P	父对象（parent）的完整哈希字串
%p	父对象的简短哈希字串
%an	作者（author）的名字
%ae	作者的电子邮件地址
%ad	作者修订日期（可以用 -date= 选项定制格式）
%ar	作者修订日期，按多久以前的方式显示
%cn	提交者(committer)的名字
%ce	提交者的电子邮件地址
%cd	提交日期
%cr	提交日期，按多久以前的方式显示
%s	提交说明
```

如`$ git log --pretty=format:"%h -%an,%ar : %s" -3`
利用上面的配置别名咱们可以
git config --global alias.lg "log --color --graph 
--pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
然后`git lg` 记得用`q`退出
`$ gitk`打开图形界面
`$ git log --help`查看帮助

###版本回退

`$ git reset --hard HEAD^`
用HEAD表示当前版本，上一个版本就是HEAD^，上上一个版本就是HEAD^^，当然往上100个版本写100个^比较容易数不过来，所以写成HEAD~100。
`$ git reset --hard 3628164`可以指定回到某个版本,不用写全版本号。
`$ git reflog`可以记录之前的命令，回退之后想要回到新版本可以用它来查到版本号

####撤销修改

`$ git checkout -- readme.txt`文件在缓存区时可撤销修改
`$ git rm`掉的也可以用上述命令回复

###远程仓库

创建SSH key`$ ssh-keygen -t rsa -C "youremail@example.com"`
####关联远程仓库
`$ git remote add origin git@github.com:***/***.git`
####推送远程仓库
`$ git push -u origin master`第一次推送master分支时，加上了-u参数，Git不但会把本地的master分支内容推送的远程新的master分支，还会把本地的master分支和远程的master分支关联起来，在以后的推送或者拉取时就可以简化命令。
###分支管理
`$ git branch -a`查看所有分支 -r查看远程分支
`$ git checkout -b dev`创建
`$ git merge dev`合并
`$ git branch -d dev`删除
####贮藏当前未提交的工作
`$ git stash``
`$ git stash list`查看
`$ git stash pop`恢复并删除
`$ git stash apply stash@{0}`恢复到指定空间
####多人协作
`$ git remote -v`查看远程库
`$ git checkout -b dev origin/dev`创建远程分支对应的本地分支
`$ git branch --set-upstream branch-name origin/branch-name`建立关联
###标签
`$ git tag v1.0`创建标签，标签是关联到当前commit上的。加上commit id可以关联到对应commit上
`$ git tag` 查看标签
`$ git tag -a v0.1 -m "version 0.1 released" 3628164`用-a指定标签名，-m指定说明文字
`$ git tag -d v0.1`删除标签
`git push origin <tagname>`推送标签
`$ git push origin --tags`推送所有标签
删除远程标签要先删除本地再`$ git push origin :refs/tags/v0.9`
###其他常用命令
`$ git commit -C HEAD -a —amend `复用HEAD留言，增补提交
`$ git blame hello.html` 查明谁修改了代码 你也可以用"-L"参数在命令(blame)中指定开始和结束行:
`$ git blame -L 12,+10 hello.html` //12到22行
`$ git gc`回收垃圾
`$ git push --delete origin dev`删除远程分支
`git remote add company  https://github.com/....`关联多个远程仓库
####提交到多个远程仓库
`.git/config`配置url`

```
    [remote "all"]  
        url = https://github.com/caolinijan/test.git  
        url = https://git.oschina.net/caolinjian/test.git 
```

`$ git push test --all `











