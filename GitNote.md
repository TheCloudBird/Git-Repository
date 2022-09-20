# Git笔记

Git是分布式的版本管理工具

## 一、git与GitHub的关系

Github是网站，是基于Git的代码托管平台

## 二、Git的安装

安装完成后校验：cmd下输入 git --version

Git安装的几个启动方式

1. Git bash 支持Linux命名的Git控制台 ()
2. Git CMD 支持window命令的控制台
3. git GUI  git的可视化界面

苹果电脑自带Git

## 三、git与github的配合使用

(1) 配合github, 查看开源项目的代码(下载)

   下载代码的指令为： git clone + 链接地址   例如    git clone https://github.com/qos-ch/slf4j.git

   在进行此操作之前需要进入到存储代码的地址。

(2)配置git,获取更改的权限

   配置项：用户名和邮箱(此用户名和邮箱为github上的)

   命令： git config --list 显示配置列表

​               git config --global user.name "${用户名}"

​               git config --global user.email "{邮箱}"

(3) 可选的配置秘钥，才能使用以git开头的链接地址    例如 git@github.com:qos -ch/slf4j.git

​       a) 在本地生成秘钥  ssh-keygen -t rsa -C "${使用的邮箱}"

​           执行命令后，会提醒你设置文件夹保存key，默认是.ssh文件夹

​           不用设置密码直接回车后，生成Id_rsa.pub秘钥文件

​        b) 在GitHub上配置秘钥

​               在GitHub的官网上，用户右键 -> setting  -> SSH and GPG keys -> new ssh key

​       c)  验证是否添加成功

​               使用ssh -T git@github.com  或者 使用git clone  + git链接

## 四、git的使用

(1) 新建远程仓库

​          仓库是管理项目的最基础的目录。

​            在github上创建一个远程仓库(本地也是可以的)。

(2) 将本地磁盘上的内容上传到远程仓库

1. 使用git bash 进入项目目录。

2. 执行 git init 就会创建git的暂存区，目录名称为.git

3. 从工作区将文件传送到暂存区(.git目录)

    命令为 git add 文件名   或 git add *  (*代表所用的文件)

      git commit -m "这一次提交的描述"

4. 查看工作区的状态   git status

      当工作区中文件更改后，git status会提示修改的文件

5. 恢复被修改的文件

     git checkout 文件名

6. 查看修改的文件 不一样的部分（被修改的部分）

   git diff

7. 查看提交的历史版本

   git log

8. 恢复文件到指定的版本

   git reset --hard 版本号

   git reset --hard HEAD~ 返回上一次提交

   git reset --hard HEAD~~ 返回上上次提交

## git的理论知识

1. 工作区与暂存区

   工作区：就是存放代码的地方(项目文件夹)

   暂存区：临时存放改动的地方 (通常在新建文件时使用)

   本地仓库：安全存放代码的地方(当前目录)

   远程仓库：(github)

   git add 将代码从工作区  转到  暂存区  (把文件托管到git)

   git commit   将代码从暂存区 转到    本地仓库 (存在一笔提交记录)

   git push  将代码从本地仓库   转到 远程仓库 

2. 分支

   每一个程序员是一个分支，他们之间的工作互不影响，分支之间是相互隔离的

   创建仓库时，自动创建一个master的主干。

     具体使用场景：

   ​          master   一般用来存放已经上线的代码

   ​         开发新的功能时，创建新的分支，每个人使用自己的分支

   ​         最终还要进行合并。但是合并时要解决合并时的冲突

3. 分支使用

   (1) 创建分支

   ​             git branch 分支名       默认拉取master上的代码到新创建的分支中,即将代码复制分支中

   ​             git brance  查看所有的分支

    (2) 切换分支

   ​            git checkout  分支名      切换到指定的分支

​       (3) 合并分支

​               当前分支是需要合并过来的代码分支

​               git merge 分支名

​      (4) 解决冲突

​      (5) 推送到远程仓库

​            git push origin 本地仓库分支 ： 远程仓库分支        

## git仓库

git仓库分为本地仓库和远程仓库

1. git init 新建一个本地代码库

2. git clone 从远程拷贝

      拿到远程仓库的修改 git pull   (拉取)

      提交给远程仓库修改 git push  (推送)

3.  一般工作的操作不知

      (1) 如果要创建一个自己的分支 确认要拉取代码的分支

      (2) 确保是最新  git pull

      (3) 然后执行创建分支的流程(从master拉取)

      (4) 如果需要，再合并一次代买

   ​    (5) 写好代码后，在推送给远程仓库

   ​    (6) 将分支代码在合并回master