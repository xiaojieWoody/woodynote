## Git初体验

```shell
#下载安装
#在使用git进行版本管理之前，需要进行一个配置，这个配置是告诉git你的用户名以及你的邮件账号
git config --global user.email "example@qq.con"
git config --global user.name "woody"
#对一个文件夹目录进行git初始化操作，这个初始化就是为了让git对这个文件夹中的文件进行版本管理,初始化完成之后，在当前目录下会有一个.git文件夹，这个文件夹就是版本库。现在就可以用git对其下的文件进行一个版本的控制，就是可以对它进行一个版本的回退，回到当前等等
git init
```

## 基础操作

```shell
git status
# 将工作区中的内容add到暂存区
git add
# 将暂存区的内容commit到提交区
git commit
```

## 版本切换

```shell
# 查看git日志信息
git log
# 简化日志信息
git log --pretty=oneline 
# 根据每次的commitid进行一个版本的切换
git reset --hard commitid
# 直接回退到上一个版本
git reset --hard HEAD^
# 回退到上上个版本
git reset --hard HEAD^^
# 如果进行版本回退之后，发现commitid最新的没了，可以使用git reflog进行所有的commitid的查询
git reflog
```

## 工作区、暂存区和提交区

```shell
#在工作区进行了修改，然后git status会有一个提示，告诉你下一步需要干嘛
git status
# 正常的思维，你会进行git add
git add
# 后悔当前的操作，git checkout -- gupao.txt  会撤销工作区中的一个修改，也就是说不需要进行git add操作了
git checkout -- gupao.txt
# 对于已经在暂存区的内容，使用git checkout -- gupao.txt  不可行了
# 回到提交区中的最新版本
git reset HEAD gupao.txt       
# 接下来需要做的就是再次使用git checkout -- gupao.txt，将修改拉回到工作区，把工作区的修改内容清空
git checkout -- gupao.txt
```

## 分支

```shell
git reset --hard commitid
# HEAD    如果说内容已经add到暂存区，此时要想撤销的话，需要先回到最新的一个commitid上     HEAD   HEAD^  HEAD^^
# HEAD就表示当前最新的版本的commitid，也就是最新的指针指向
# HEAD的一个补充：HEAD头指针指向的是当前分支最新的commitid
# 创建一个分支
git checkout -b dev_wang
# git branch可以查看当前所有的分支情况，并且可以看到目前所处的分支(*)
git branch
# 强制性的删除该分支
git branch -D dev_wang
```

## 分支合并和冲突解决

* 当远程仓库的版本内容如果和本地仓库的内容不一致，需要先git pull，把远程仓库的内容拉下来到最新版本才行，然后进行手动解决冲突

```shell
# 各自在自己的分支上开发完成之后，需要将开发的内容合并到主分支上去。这个时候成为merge
# 需求:一个新的开发人员dev_zhang
1. git checkout -b dev_zhang;
2. 进行文件的修改并且add，commit
3. 此时在小张的分支上多了一个commitid，这时候需要把这个小张修改的内容进行版本的发布，就需要把小张的修改内容合并到master分支上
4. 切换到master分支，合并dev_zhang的开发内容
# 合并的操作：快速合并   在master分支上，git merge dev_zhang;
git merge dev_zhang;
git branch -d dev_zhang;
```

```shell
# 项目开发人员很多，2个，小张，小李，合作开发一个项目
# 需求：小张，小李合作开发一个项目，这个项目两个人负责不同的模块
# 小张：商品管理的模块
# 小李：订单管理的模块
# master分支作为一个版本发布的分支，不应该进行直接在上面开发
# 1. git checkout -b dev_zhang
# 2. git checkout -b dev_li
# 3. 分别在小张和小李的分支上进行开发之后,发现master分支上并没有小张小李的开发内容
# 4. 小张和小李的开发内容发布到master分支
# 分支的合并，合并的冲突的问题
# 需要手动解决冲突，并且再去进行add,commit的操作
git checkout -b test
```

## git config和配置别名

```shell
# git config配置git的命令
# git config -l   查看所有的配置信息
# 仓库级别的配置：当前仓库级别下的.git>config文件
# 全局级别的：当前用户之下表示的是全局级别的
# 系统级别：在我们的git安装目录下etc
git config --global --add user.name itcrazy2016
git config --global --unset user.name
```

```shell
# 比较实用命令
git status   
git add    
git commit 
git log --pretty=oneline
# 配置别名进行简化
git config --global alias.st status    #表示用st代表status
git st #表示查看用户状态
git cm #提交commit
git log one #查看一行信息
```

## 打标签和忽略文件

```shell
# 打标签
# 用一种比较独特的方式去记住每个版本
# 给最新版本的id打上一个标签，将最新版本的commitid对应上v1
git tag v1
# 查看一下当前仓库的标签列表
git tag
# 给之前已经错过的commitid去打上一个标签
git tag v1pre d619d86
# 给标签加上一个说明
git tag v2.0 -m "这里打上了一个标签"
# 删除标签
git tag -d v1
```

```shell
# 忽略文件
# 仓库的根目录下创建一个.gitignore 文件
# 这个规则你要让git能够看懂，告诉git让他不要帮你去管理这个文件了
*.class
```

## 本地仓库和远程仓库

```shell
# 搭建自己的私有仓库的话，让别人不可见，怎么做呢？Gitlab
```

## github和码云

```shell
# 把本地仓库gupaogit   上传到github上去进行
# 1. 在远程仓库github创建一个对应的项目比如gupaogit    repository仓库
```

![image-20191212153843187](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191212153843187.png)

## 操作远程仓库和本地仓库

```shell
# 要让两者有关联，在本地仓库中配置一下它和远程仓库的关系
# 当前本地仓库是否有远程仓库，如果有，那么它的远程仓库是什么
git remote
# 1. 本地仓库中:
git remote add origin 远程仓库的地址？git@github.com:itcrazy2016/gupaogit.git
# origin 本地仓库和远程仓库的地址进行一个关联
# 2. 推送代码，那么就不是指定远程仓库
git push -u origin master
# 这样关联之后，接下来就是把代码推送到远程仓库上
# 本地仓库和远程仓库进行关联之后，就可以进行的推送了
# 将本地仓库的内容推送到远程仓库
git push -u origin master
```

```shell
# 没有权限
# 需要添加权限
# A.需要在本地中生成一个ssh key
# 在自己的计算机中增加一个安全ssh key 盖上了一个章，就表示你这个电脑认证后的ssh_key
ssh-keygen -t rsa -C "itcrazy2016@163.com"   
# B.需要把这个key告诉github/码云
# 把公钥放到ssh key
# 保证数据传输的一个安全性
```

![image-20191212154407033](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20191212154407033.png)

```shell
# 如果说是在其他分支进行的修改，需要进行一个分支的合并
# 要推送其他分支的，指定一下要推送的分支即可
git push -u origin dev_zhang;
```

```shell
# 新来了一个哥们，小王，需要进行一个开发
# 需要把远程仓库的代码拉倒本地进行开发
# 1. clone 克隆操作
# 前提是 sshkey也要添加完成
git clone git@github.com:itcrazy2016/gupaogit.git
```

