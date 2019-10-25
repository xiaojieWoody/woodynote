# 使用场景

###  本地项目和github仓库关联

```shell
# 在git上创建一个新仓库 https://github.com/xiaojieWoody/jvm_queue.git
# 将本地代码推送到新仓库
echo "# jvm_queue" >> README.md
git init
git add .
git commit -m "init"
git remote add origin https://github.com/xiaojieWoody/jvm_queue.git
[git pull origin master --allow-unrelated-histories]
git push -u origin master
```

### 

## Git三剑客

* 集中式VCS
  * 有集中的版本管理服务器
  * 具备文件版本管理和分支管理能力
  * 集成效率有明显地提⾼
  * 客户端必须时刻和服务器相连
* 分布式VCS
  * 服务端和客户端都有完整的版本库
  * 脱离服务端，客户端照样可以管理版本
  * 查看历史和版本⽐较等多数操作，都不需要访问服务器，比集中式 VCS 更能提高版本管理效率
* git特点
  * 最优的存储能⼒、非凡的性能、开源的、很容易做备份、支持离线操作、很容易定制工作流程

### Git

* 配置user信息

  ```shell
  git config --global user.name 'name'
  git config --global user.email 'xxx@email'
  # config的三个作用域local>global>system，缺省等同于local
  # 显示config的配置，加--list
  git config --list --global
  # 清除设置
  git config --unset --global user.name
  ```

* 建Git仓库

  * 用Git之前已有项目代码

    ```shell
    cd 项目代码所在文件夹
    git init
    ```

  * 用Git之前还没有项目代码

    ```shell
    cd 某个文件夹
    git init your_project # 会在当前路径下创建和项目名称同名的文件夹
    cd your_project
    ```

* 命令

  ```shell
  # 更改文件名
  git mv readme readyou
  # 还原本地工作区和暂存区
  git reset --hard
  # 查看本地分支
  git branch -v
  # 查看本地和远程分支
  git branch -av
  # 创建并切换到分支
  git checkout -b temp 425c13sd31
  # 创建本地分支与远程分支对应（先git branch -av查看远程分支）
  git checkout -b feature/add_git_commands(本地) github/feature/add_git_commands(远程)
  # 查看提交日志
  git log --all --graph
  # 查看最近的几条commit记录
  git log -n8 --all --graph
  git log --oneline --all -n4 --graph
  # 查看命令
  git help --web log
  # 修改最近一次commit的msg
  git commit --amend
  # 还原暂存区修改
  git reset --hard HEAD [-- 文件名1 文件名2]
  # 还原工作区修改
  git checkout
  # 工作区和暂存区回到某个commit状态
  git reset --hard 5bsd131sdf
  # 比较两个分支中某个文件的差异
  git diff temp master -- index.html
  git diff branchName branchName -- index.html
  git diff commitId commitId -- index.html
  # 删除某个文件
  git rm 文件名
  # 将工作区更改内容保存，稍后恢复
  git stash
  git stash list
  做其他紧急任务后，恢复
  git stash apply        # list中stash信息还存在
  git stash pop          # list中stash信息不存在
  ```

* 自带图形工具-gitk

  ```shell
  gitk --all 
  ```

* 分支

  ```shell
  # 查看本地分支
  git branch -av
  # 删除分支
  git branch -d 2332sfs33
  ```

### GitHub

* 本地项目与远程仓库关联

  ```shell
  git remote add github 仓库地址
  git remote -v
  git fetch github master
  git branch -av
  git merge --allow-unrelated-histories github/master
  git push github master
  ```

  ```shell
  # 根据远程分支克隆一条本地分支
  git checkout -b feature/add_git_commands(本地) origin/feature/add_git_commands(远程)
  ```

* 解决冲突

  ```shell
  1. 合并后，手动修改有冲突的文件
  2. git commit -am'Resolved conflict by hand...'
  3. git status
  4. git push [远程]
  ```

* 查找资料

  ```shell
  # github中搜索 条件搜索 in、stars
  git 最好 学习 资料 in:readme stars:>1000
  ```

* 搭建自己的github博客

  ```shell
  # All github search
  blog easily start in:readme stars:>5000
  https://github.com/barryclark/jekyll-now
  ```

### GitLab

* 持续集成CI
  1. Files——.gitlab-ci.yml
  2. Settings——CI/CD——runner
  3. deploy