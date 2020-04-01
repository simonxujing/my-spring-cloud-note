# 一、基本概念

1. Watch：关注。当项目更新可接受到通知
2. Issue：事务卡片。发现代码bug，但目前没有成型的代码，需要讨论时用。
3. Repository： 代码仓库。
4. Fork：分支。复制他人的项目至自己的项目中，可通过push request，提交到主分支上

# 二、Git 命令

1. git status：git状态
2. git初始化用户名、邮箱：
   1. git config --global user.name 'simonxujing'
   2. git config --global user.email 'simonxujing@gmail.com'
3. git init：git初始化
4. touch config-dev.yml: 创建一个名为config-dev.yml 文件
5. git add config-dev.yml：将名为config-dev.yml 文件提交至暂存区
6. git commit -m 'desc: add xx' : 提到暂存区的文件至仓库
7. 删除文件：
   1. rm -rf xxx.yml 本地删除
   2. git rm xxx.yml 从git上删除
   3. git commit -m 'remove xxx.yml' 提交删除操作
8. git push: 在commit之后执行，将文件远程仓库