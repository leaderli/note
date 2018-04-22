工作区指当前操作目录

缓存区指git add后的区

版本库指git版本控制的区

HEAD指向的是当前分支，默认分支是master

```shell
git add  					#添加到仓库
git branch -d dev			#删除分支	gbd
git checkout -- readme.txt #丢弃工作区的修改
git checkout -b dev			#创建并切换 gcb
git checkout master			#切换到master版本 gcm
git commit -m "messeage" 	#提交到仓库
git difftool				#使用ksdiff工具
git init 					#把当前目录变成git可以管理的仓库
git log   					#查看提交历史
git log --graph --pretty=oneline --abbrev-commit #有图形 glol
git log --pretty=oneline						
git merge --no-ff -m "merge with no-ff" dev #合并要创建一个新的commit，所以加上-m参数，把commit描述写进去。
git merge dev				#合并dev分支
git reflog 				 #记录命令历史
git reset					#回退版本
git reset --hard HEAD^   #回退到上一个版本
git reset HEAD readme.txt   #撤回缓存区的修改
git rm readme.txt			#删除版本库文件 
git stash 					#暂存当前工作区 
git stash list				#查看stash纪录 gstl
git stash pop				#弹出stash内容 gstp
git status  				#查看当前工作区状态
```

```
git@github.com:leaderli/git.git
```

