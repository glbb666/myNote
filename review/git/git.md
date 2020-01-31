基本功能

### 指令

#### 查看状态(`git status` )

```nginx
git status
```

- “*Changes to be committed*”中所列的内容是修改内容已经保存至缓冲区，等待进入git目录。
- “*Changed but not updated*”中所列的内容是发现修改但没有保存在缓冲区中。
- “*Untracked files*”中所列的内容是尚未被Git跟踪的内容。

#### 查询修改部分(`git diff`)

```nginx
git diff <name> #比较工作区和暂存区
git diff （master） <name> #比较分支与工作区
git diff --cached <file> #比较分支与暂存区
```

- master用分支表达式替换：HEAD、HEAD^、HEAD~1、0c5ee16a6a4c849d0ae0448caa8ff174399c7c3c
- 比较时要切换到对应分支

#### 分支合并`git rebase`(变基、衍合) 和 `git merge`

`git merge` 和 `git rebase` 都可以整合两个分支的内容，最终结果没有任何区别，但是`git rebase`使得提交历史更加整洁。

例如现在 `dev `提交了一次，`master` 在此之后也提交了一次，两个分支的状态如下：



![image](images/160d8f9f81c8d790)



`git merge` 的结果：



![image](images/160d8f9f8141bdda)



`git rebase` 的结果：



![image](images/160d8f9f802d0ba6)



#### 提交点顺序

- `git merge`后，提交点的顺序都和提交的时间顺序相同，即 `master` 的提交在 `dev `之后。
- `git rebase`后，顺序变成被`rebase`的分支（`master`）所有提交都在前面，进行`rebase`的分支（`dev`）提交都在被`rebase`的分支之后，在同一分支上的提交点仍按时间顺序排列。

#### 分支变化

- `dev` 在 `rebase master` 后，由原来的两个分岔的分支，变成重叠的分支，看起来 `dev` 是从最新的 master 上拉出的分支。

#### 什么时候使用 `rebase` / `merge`

假设场景：从 `dev` 拉出分支 `feature-a`。那么当 `dev` 要合并 `feature-a` 的内容时，使用 `git merge feature-a`；反过来当` feature-a `要更新 `dev `的内容时，使用 `git rebase dev`。

使用时主要看两个分支的"主副"关系。

#### 注意

一般来说，`rebase`后的 `dev` 和远程的`origin/dev`会发生分离，在命令行界面中会提示：

```
Your branch and 'origin/dev' have diverged,
and have 1 and 1 different commits each, respectively.
  (use "git pull" to merge the remote branch into yours)
```

这时需要用`git push -f`强制推送，覆盖远程分支。若使用了提示中的`git pull`，结果会变成合并，并产生一个合并提交点。

**慎用git push -f!**

————————————————————

#### `git merge --no-ff`

`--no-ff` 是不快速合并的意思

#### 与 `git merge` 的区别

`git merge`的结果：

被merge的分支和当前分支在图形上并为一条线，被merge的提交点逐一合并到当前分支。



![image](images/160d8f9f7dc4ea62)



`git merge --no-ff`的结果：

被merge的分支和当前分支不在一条线上，被merge的提交点还在原来的分支上，同时在当前分支上产生一个新的提交点。



![image](images/160d8f9fbc1212a4)



#### git pull --rebase

#### 与 git pull 的区别

在一般情况下，加与不加 `--rebase` 是没有区别的。

然而，从上面说的 `git rebase` 功能得知，某个分支可能与其远程分支发生分离，而当你 `pull` 时使用 `git pull`，则会变成你的本地分支和远程分支合并。

正确的做法是`git pull --rebase`，才会拉取到最新的分支。

因此推荐在任何时候 `pull` 远程分支，最好加上 `--rebase` 参数。
#### 分支和 `tag`

合理使用分支，分支的好处：

- 同时开发不同功能不冲突，可独立测试
- 可集中解决冲突
- 区分功能或未来某一版本

`tag` 的作用是对某个提交点打上标签，发布版本后打 `tag`，便于以后回滚特定版本，而不需要 `revert`。

#### 开发新功能步骤

1. 从开发分支拉一个功能分支
2. 功能分支开发和测试
3. 功能分支 `rebase` 开发分支（为什么）
4. 功能分支合并到开发分支

#### 注意：

- 一次提交做一件事，写清楚 `comment`
- 每次 `pull` 远程分支时使用 `git pull --rebase`
- 分支从哪拉出来，最后合到哪回去
- 合并之前先 `rebase`

#### 线上bug的修复

1. 从`master`拉一个fix分支（为什么是master）
2. 测试完后 `rebase master`
3. 合并回`master`
### 撤销与版本回退

#### 工作区(`revert` 和 `reset`)

`git revert`**撤销提交**，在当前提交后，新增一次提交，抵消掉**上一次提交**导致的所有变化。它不会改变过去的历史，没有任何丢失代码的风险。

```nginx
$ git revert HEAD
```

`git reset`**丢弃提交**，让最新提交的指针回到以前某个时点，该时点之后的提交都从历史中消失。执行`git reset`命令之后，如果想找回那些丢弃掉的提交，可以使用`git reflog`命令。

```nginx
$ git reset --hard [last good SHA]
```
#### 暂存区

从暂存区撤销文件，恢复到已修改未暂存状态

```nginx
$ git rm --cached [filename]//方法一
$ git reset HEAD {filename}//方法二
```
#### `git reset` 常用命令

```nginx
git reset --hard HEAD^ #回退到上一版本
git reset --hard HEAD~n #回退到上n个版本
git reset --hard 1094a #跳转到指定版本
```

`git reset`默认不改变工作区(但会改变暂存区)，`--hard`参数可以让工作区里面的文件也回到以前的状态。

1. 使用参数`--hard`，**暂存区、工作区**和 `HEAD` 指向的目录树内容相同。
2. 使用参数`--soft`，只更改 `HEAD` 的指向，暂存区和工作区不变。
3. 使用参数`--mixed`或者不带参数（默认为`--mixed`），更改引用的指向及重置暂存区，但是不改变工作区。

#### git stash

把工作区内容缓存到一个栈里，之后用 `git stash pop`取出。在未提交工作区内容，但是想切到其他分支时非常有用。

#### 注意

不建议同一时间段在不同分支都使用 `git stash`，涉及到多个分支的情形还是先  `commit` 较好，不push到远程，下次 commit 时可用 `--amend` 合到上次提交中。

