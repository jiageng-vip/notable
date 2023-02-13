---
tags: [Git]
title: Git 命令
created: '2022-04-29T16:37:15.746Z'
modified: '2022-05-02T07:35:46.865Z'
---

# Git 命令

```
【设置全局用户名&邮箱】
$ git config --global user.name "Your Name"
$ git config --global user.email "email@example.com"
```

```
【创建版本库】
git init
```

```
【添加文件（至暂存区）】
git add file1 file2
```

```
【提交文件（至本地版本库）】
git commit <file> -m <message>
```

```
【工作区状态】
git status
```

```
【查看修改内容】
git diff file
【工作区和版本库里面最新版本的区别】
git diff HEAD -- file
```

```
【版本回退（Git允许我们在版本的历史之间穿梭）】
用HEAD表示当前版本，上一个版本就是HEAD^，上上一个版本就是HEAD^^，当然往上100个版本写100个^比较容易数不过来，所以写成HEAD~100
git reset --hard commit_id
git reset --hard HEAD^
```

```
【提交日志】
git log
git log --pretty=oneline
【查看命令历史，以便确定要回到未来的哪个版本。】
git reflog 
```

```
【撤销工作区修改】
git checkout -- file
```

```
【删除文件】
git rm file
```

```
【添加远程库（-u 第一次推送时添加）】
git remote add origin url && git push -u origin master
git clone url projectName
git remote 查看远程库信息
git remote -v 显示更详细的远程库信息
```

```
【分支（当Git无法自动合并分支时，就必须首先解决冲突。解决冲突后，再提交，合并完成）】
创建分支：git branch <name>
查看分支：git branch
删除分支：git branch -d <name>
切换分支：git checkout <name>
切换分支：git switch <name>
创建+切换分支：git checkout -b <name>
创建+切换分支：git switch -c <name>
合并分支：git merge <name>   --no-ff参数可以用普通模式合并，合并后的历史有分支，能看出来曾经做过合并，而fast forward合并就看不出来曾经做过合并。
    
# 解决完冲突文件后
git add/rm <conflicted_files>
# 新分支合并
git rebase --continue
# 不需要 commit
git rebase --skip
```

```
【bug 分支】
git stash 储藏
git stash apply
git stash drop
git stash list
git stash pop
git cherry-pick
*git stash apply恢复，但是恢复后，stash内容并不删除，你需要用git stash drop来删除*
*git stash pop，恢复的同时把stash内容也删了*

修复bug时，我们会通过创建新的bug分支进行修复，然后合并，最后删除；
当手头工作没有完成时，先把工作现场git stash一下，然后去修复bug，修复后，再git stash pop，回到工作现场；
在master分支上修复的bug，想要合并到当前dev分支，可以用git cherry-pick <commit>命令，把bug提交的修改“复制”到当前分支，避免重复劳动。
```

```
【feature分支】
git branch -D <name>

开发一个新feature，最好新建一个分支；
如果要丢弃一个没有被合并过的分支，可以通过git branch -D <name>强行删除。
```

```
【多人协作】
查看远程库信息，使用git remote -v；
本地新建的分支如果不推送到远程，对其他人就是不可见的；
从本地推送分支，使用git push origin branch-name，如果推送失败，先用git pull抓取远程的新提交；
在本地创建和远程分支对应的分支，使用git checkout -b branch-name origin/branch-name，本地和远程分支的名称最好一致；
建立本地分支和远程分支的关联，使用git branch --set-upstream branch-name origin/branch-name；
从远程抓取分支，使用git pull，如果有冲突，要先处理冲突。
```
