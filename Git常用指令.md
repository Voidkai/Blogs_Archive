# Git常用指令

## 新建代码库相关指令

1. 在当前目录新建一个Git代码库：`$ git init`
2. 新建一个指定项目名称的Git代码库：`$ git init [Project-name]`
3. 下载一个项目和它的整个代码提交历史（例如从**Github**中下载）:`$ git clone [url]`



## Git增删文件指令

1. 添加指定文件或文件夹到暂存区：`$ git add [file1] [fiel2] | [dir]...`

2. 添加当前目录的所有文件到暂存区：`$ git add .`

3. 删除指定的文件,并将这次删除保留在工作区：`$ git rm [file1] [file2]| [dir]`




## Git代码提交指令

1. 提交暂存区到仓库区：`$ git commit -m "message"`



## Git分支操作指令

1. 列出所有本地分支：`$ git branch`或本地分支`$ git branch -r`
2. 合并指定分支到当前分支：`$ git merge [branck]`



## Git远程同步指令

1. 显示所有的远程仓库：`$ git remote -v`
2. 取回远程仓库的变化并与本地分支合并：`$ git pull [remote] [branch]`
3. 上传本地指定分支到远程仓库：`$ git push [remote] [branch]`



## 在本地修改后的文件需要以下几步与远程github同步

```bash
$ git add .
$ git commit -m "message"
$ git pull origin master
$ git push origin master
```

