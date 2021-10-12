```
# 新建本地分支，但不切换
git branch <branch-name> 
# 查看本地分支
git branch
# 查看远程分支
git branch -r
# 查看本地和远程分支
git branch -a
# 删除本地分支
git branch -d <branch-nane>
git branch -D <branch-nane>（强制删除）
# 删除远程分支
git branch -d -r branchname 
# 重新命名分支
git branch -m <old-branch-name> <new-branch-name>
# 查看本地各个分支最后一个提交对象的信息
git branch -v
# 查看哪些分支已经合并到当前分支
git branch --merged
# 查看哪些分支还没有合并到当前分支
git branch --no-merged
```

