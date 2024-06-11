---
title: "git常用操作指南"
date: "2020-07-11"
lastMod: "2020-07-11"
categories: ["it"]
tags: ["git"]
---

**创建分支**：`git branch <name>`，创建叫name的分支，但仍然停留在当前分支。

**删除分支**：`git branch -d <name>`，参数为-D则为强制删除。

**切换分支**：`git switch <name>`

**创建+切换分支**：`git switch -c <name>`

上方两条命令一个意思：如果分支存在则只切换分支。不存在则创建叫name的分支，然后切换到该分支。相当于两条命令：git branch <name>，git switch <name>

**查看分支**

git branch：查看本地分支，当前分支前面会标一个*号。

git branch -r：查看远程分支。

git branch -a：查看本地分支和远程分支，远程分支会用红色表示出来（如果你开了颜色支持的话）。

git branch -vv：查看本地分支对应的远程分支。

**重命名分支**

git branch -m oldName newName

---

git add -u
git reset --hard，清理暂存区和工作空间的修改（没有commit的内容）
git mv比直接mv后add和rm更好

分离头指针，HEAD指向的不是分支，而是commit，一般用于临时快速测试验证
git branch -d删除不掉的原因
git commit --amend 修改最近一次提交的注释

git rebase -i <commit id>，变基，可修改历史提交注释，合并提交

git diff，工作区与暂存区的差别
git diff --cached，HEAD与暂存区的差别
git diff -- [files]，查看指定文件的差别，多个文件空格分隔

git reset HEAD [file
]，恢复暂存区，与HEAD一致
git reset --head <commit id>，撤销提交

git stash
git stash apply
git stash pop
git stash list

.gitignore：
加/，包含文件夹及其里面的文件
不加/，包含文件夹及其里面的文件，以及名为这个的文件

git reset --hard <commit id>，回退到某次提交

github flow
gitlab flow（带生产分支，环境分支）

分支合并方式（github）
merge commit，当前常用
squash & merge，变更集合并，分支不合并
rebase & merge，变更集不合并，分支不合并

测试gitlab不同的merge request
