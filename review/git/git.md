文件状态转换：

![img](images/16343316cbad1533)

文件存储：

![img](images/1634331848b2fff5)



## Git 文件状态

通过 `git status` 查看

- untracked 新文件未加入版本管理
- unmodify
- modified
- staged 用 git add 暂存

`git diff --staged` 或 `git diff --cached` 可查看已暂存文件和上次提交的区别

## 分支和 tag

合理使用分支，分支的好处：

- 同时开发不同功能不冲突，可独立测试
- 可集中解决冲突
- 区分功能或未来某一版本

tag 的作用是对某个提交点打上标签，发布版本后打 tag，便于以后回滚特定版本，而不需要 revert。

tag 是对某一版本的记录。

## 开发新功能步骤

1. 从开发分支拉一个功能分支
2. 功能分支开发和测试
3. 功能分支 rebase 开发分支（为什么）
4. 功能分支合并到开发分支

#### 注意：

- 一次提交做一件事，写清楚 comment
- 每次 pull 远程分支时使用 `git pull --rebase`
- 分支从哪拉出来，最后合到哪回去
- 合并之前先 rebase

## fix bug 步骤

### 测试线bug的修复

和开发步骤类似

### 线上bug的修复

1. 从master拉一个fix分支（为什么是master）
2. 测试完后 rebase master
3. 合并回master

## Git 使用技巧

### rebase 和 merge

`git rebase`一般解释为`变基`，也有解释为`衍合`。

`git merge` 和 `git rebase` 都可以整合两个分支的内容，最终结果没有任何区别，但是变基使得提交历史更加整洁。

例如现在 dev 提交了一次，master 在此之后也提交了一次，两个分支的状态如下：



![image](images/160d8f9f81c8d790)



`git merge` 的结果：



![image](images/160d8f9f8141bdda)



`git rebase` 的结果：



![image](images/160d8f9f802d0ba6)



#### 提交点顺序

- `git merge`后，提交点的顺序都和提交的时间顺序相同，即 master 的提交在 dev 之后。
- `git rebase`后，顺序变成被`rebase`的分支（master）所有提交都在前面，进行`rebase`的分支（dev）提交都在被`rebase`的分支之后，在同一分支上的提交点仍按时间顺序排列。

#### 分支变化

- dev 在 rebase master 后，由原来的两个分岔的分支，变成重叠的分支，看起来 dev 是从最新的 master 上拉出的分支。

#### 什么时候使用 rebase / merge

假设场景：从 dev 拉出分支 feature-a。那么当 dev 要合并 feature-a 的内容时，使用 `git merge feature-a`；反过来当 feature-a 要更新 dev 的内容时，使用 `git rebase dev`。

使用时主要看两个分支的"主副"关系。

#### 注意

一般来说，`rebase`后的 dev 和远程的`origin/dev`会发生分离，在命令行界面中会提示：

```
Your branch and 'origin/dev' have diverged,
and have 1 and 1 different commits each, respectively.
  (use "git pull" to merge the remote branch into yours)
复制代码
```

这时需要用`git push -f`强制推送，覆盖远程分支。若使用了提示中的`git pull`，结果会变成合并，并产生一个合并提交点。

**慎用git push -f!**

————————————————————

### git merge --no-ff

`--no-ff` 是不快速合并的意思

#### 与 git merge 的区别

`git merge`的结果：

被merge的分支和当前分支在图形上并为一条线，被merge的提交点逐一合并到当前分支。



![image](images/160d8f9f7dc4ea62)



`git merge --no-ff`的结果：

被merge的分支和当前分支不在一条线上，被merge的提交点还在原来的分支上，同时在当前分支上产生一个新的提交点。



![image](images/160d8f9fbc1212a4)



————————————————————

### git rebase -i 操作

用于整理提交和提交信息，貌似不太好用，看演示:

`git rebase -i dev` 后进入如下界面。

```
1 pick ffc75a4 修改
2 pick 6c2217f bbb
3 pick a615960 修改a 1
4 pick e4817b3 修改a 2
5 pick 8550690 修改a 3
6
7 # Rebase 3190ea4..8550690 onto 3190ea4 (5 command(s))
8 #
9 # Commands:
10 # p, pick = use commit
11 # r, reword = use commit, but edit the commit message
12 # e, edit = use commit, but stop for amending
13 # s, squash = use commit, but meld into previous commit
14 # f, fixup = like "squash", but discard this commit's log message
15 # x, exec = run command (the rest of the line) using shell
16 # d, drop = remove commit
17 #
18 # These lines can be re-ordered; they are executed from top to bottom.
19 #
20 # If you remove a line here THAT COMMIT WILL BE LOST.
21 #
22 # However, if you remove everything, the rebase will be aborted.
23 #
24 # Note that empty commits are commented out
复制代码
```

修改commit注释前面的`pick`，达到合并提交的目的，其中`r`是保留提交，修改提交信息，`f`是保留提交但是丢弃这次提交信息。

```
1 r ffc75a4 修改
2 f 6c2217f bbb
3 r a615960 修改a 1
4 f e4817b3 修改a 2
5 f 8550690 修改a 3
复制代码
```

然后进入编辑两个打了`r`的提交信息的界面，修改提交信息保存即可。

这时候再看log，刚才的5次提交已经变成2次提交。

```
commit 94784f0c163bc4b2970f73066c91fac16b64be32
Author: ***
Date:   Mon Jan 8 17:01:57 2018 +0800

    修改a

commit 52907b261821afb0c38754ba95545ff8826910db
Author: ***
Date:   Mon Jan 8 16:28:05 2018 +0800

    修改b
复制代码
```

————————————————————

### git pull --rebase

#### 与 git pull 的区别

在一般情况下，加与不加 `--rebase` 是没有区别的。

然而，从上面说的 `git rebase` 功能得知，某个分支可能与其远程分支发生分离，而当你 `pull` 时使用 `git pull`，则会变成你的本地分支和远程分支合并。

正确的做法是`git pull --rebase`，才会拉取到最新的分支。

因此推荐在任何时候 `pull` 远程分支，最好加上 `--rebase` 参数。

————————————————————

### reset 和 revert

- `git reset` 修改 HEAD 指向的位置
- `git revert`还原某一个提交，并产生新提交来记录本次还原

#### git reset 常用命令

- `git reset HEAD {filename}`: 取消暂存文件，恢复到已修改未暂存状态。
- `git reset HEAD~{n}`: 表示回退到`n`个提交之前。它也可以用来合并提交，下面的写法与 `git commit --amend` 结果是一样的。

```
git reset HEAD~1
git commit
```

- `git reset {version}`: 后面带版本号，直接回退到指定版本。
- `git reset的三种参数`:
  1. 使用参数`--hard`，暂存区、工作区和 HEAD 指向的目录树内容相同。
  2. 使用参数`--soft`，只更改 HEAD 的指向，暂存区和工作区不变。
  3. 使用参数`--mixed`或者不带参数（默认为`--mixed`），更改引用的指向及重置暂存区，但是不改变工作区。

————————————————————

### git reflog

查看提交记录的命令是`git log`，而`git reflog`的功能是查看本地操作记录，可以看到本地的`commit`, `merge`, `rebase`等操作记录，并带有版本号。

```
b3bf634 HEAD@{0}: rebase -i (finish): returning to refs/heads/feature-rebase-i
b3bf634 HEAD@{1}: rebase -i (fixup): 完善a中的判断和输出
dd19de3 HEAD@{2}: rebase -i (fixup): # This is a combination of 2 commits.
c138acf HEAD@{3}: rebase -i (reword): 完善a中的判断和输出
a7f47b2 HEAD@{5}: rebase -i (start): checkout dev
a472934 HEAD@{6}: rebase: aborting
a7f47b2 HEAD@{7}: rebase -i (start): checkout dev
a472934 HEAD@{8}: commit: 添加a输出
c84d5b7 HEAD@{9}: commit: 添加a中的判断
a5a6e64 HEAD@{10}: commit: 修改a内容
a7f47b2 HEAD@{11}: checkout: moving from dev to feature-rebase-i
复制代码
```

————————————————————

### git stash

把工作区内容缓存到一个栈里，之后用 `git stash pop`取出。在未提交工作区内容，但是想切到其他分支时非常有用。

#### 注意

不建议同一时间段在不同分支都使用 `git stash`，涉及到多个分支的情形还是先  `commit` 较好，不push到远程，下次 commit 时可用 `--amend` 合到上次提交中。

