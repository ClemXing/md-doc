# GIT



```shell
git config core.filemode false


#提交 $ID （中指定文件）的内容
git show $id 
git show $id:$file


#将所有untracked file 一次性删除
git  clean  -f

#在工作区中删除文件，并加入暂存区（在暂存区中暂存该删除动作），为提交做准备
git rm [file]
#重命名文件，并加入暂存区（在暂存区中暂存该删除动作），为提交做准备
git mv [file-original] [file-renamed] 
#将文件取消版本控制，但在本地保留
git rm --cached filename #相当于git add的一个逆过程

git diff          #显示工作区和暂存区的文件差异
git diff --cached #显示暂存区和版本库当前版本的文件差异


#撤销工作区修改（使工作区与暂存区同步）
git checkout -- fileName
##检出版本号为 $id 的一个文件，到工作目录以及暂存区
git checkout $id -- $file

git checkout --theirs $id -- $file

#修补式提交，即生成一个新提交，替换掉当前分支最末的提交
git commit --amend
#生成新提交，用于恢复指定的已发布提交所做出的修改
git revert [commit]
#把另一个分支上的指定提交之修改（指操作）应用到当前分支
git cherry-pick -x [commit]

#撤销最后的提交
git revert HEAD #将会新建一个提交
#撤销指定的提交
git revert $id #将会新建一个提交

#本地分支都用rebase（使提交更优雅），远程分支都用merge（记录真实的提交历史）
# 拉取最新代码
git stash
git pull --rebase #等效为fetch + rebase；而git pull等效为fetch + merge
git stash pop





# 切换分支 cherry-pick
git checkout -b team3_audio origin/slp/nvr/team3/audio
git pull --rebase
//执行cherry-pick
//解决冲突
git cherry-pick --continue

#改变提交顺序
git rebase -i HEAD^^ 

#
git reset origin/slp/hz/nvr/vcn6800v1/vcn6800v1-20200514 --hard
git pull origin slp/hz/nvr/vcn6800v1/vcn6800v1-20200514



#配置
[alias]
  co = checkout
  ci = commit
  st = status -sb
  br = branch
  lg = log --all --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset'--abbrev-commit --date=relative
  ls = log --pretty=format:'%h %ad | %s%d [%an]' --graph --date=short
  type = cat-file -t
  dump = cat-file -p
```
## 日志

```shell
#查看日志
git log
git log --pretty=oneline
git log --oneline
git reflog #多了信息：HEAD@{数字}，表示指针回到当前这个历史版本需要走多少步

#所有分支的提交历史
git log --all --graph --decorate

#文件的变更历史和变更内容
git log --stat -[number(提交数量)]  #提交中更改的文件
git log --follow [file] #列出当前文件的版本历史，包括重命名的动作
git log -p $file $dir/ec/tory/
```

## 版本

```shell
#参数默认为mixed，索引默认为HEAD
#本地库的指针移动的同时，重置暂存区，重置工作区
git reset --hard [索引] 

#本地库的指针移动的同时，重置暂存区，但是工作区不动
# git reset [commit] 删除[commit]之后的提交，但保持工作区内容不变
git reset --mixed [索引] 
#拉取最近一次提交到版本库的文件到暂存区 并且该操作不影响工作区（当我们把工作区的某个文件弄乱了 我们就可以使用该命令 把版本库中的那个文件拉到暂存区 然后在拉回工作区）
git reset HEAD -- filename  # "."表示一次性撤销所有放入暂存区的文件

#本地库的指针移动的时候，暂存区，工作区都不动
git reset --soft [索引] 

```

## 分支

```shell
#所有的本地分支和远程分支
git branch -a

#新建一个本地分支
git branch [branch-name]
#查看分支
git branch -vv

#将 branch2 变基到 branch1
git checkout $branch2 #切换到指定分支，并更新工作区
git rebase $branch1  #将当前分支的所有修改都移到指定分支上

#将 $branch2 合并到当前分支
git merge $branch2

#基于分支 $other 创建分支 $new_branch；然后切换到新分支
git checkout -b $new_branch $other
git checkout -b $local_branch origin/$remote_branch

# 删除分支$branch
git branch -d $branch
```

## GitHub

```shell
git remote -v
#关联远程
git remote add md https://github.com/ClemXing/md-doc.git
#推送
git push md master
```

### SSH

```shell
#生成ssh
ssh-keygen -t rsa -C ClemXing
```

