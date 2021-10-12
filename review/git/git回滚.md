# git reset 和 git revert

## git reset

```
develop ----1      3-----
             \   /
branch a       a
```

`develop`将`a`分支合并后，想要不留痕迹的撤回合并。这个时候用`git reset`就是很好的选择了。

具体的操作步骤如下：

1. 切换分支到`develop`
2. `git log `查看当前分支日志

例如我的日志是这样的：

```
commit 3
Merge 1 2
Author: admin <admin@163.com>
Date: Wed May 30 15:00:00 2018 +0800

    Merge branch 'feature/a' into 'develop'

    close a

    See merge request !20

commit 2
Author: admin <admin@163.com>
Date: Wed May 30 14:00:00 2018 +0800

    close a

commit 1
Author: admin <admin@163.com>
Date: Wed May 30 13:00:00 2018 +0800
    init project
```

我要将`develop`回退到合并之前的状态，那就是退到` commit 1`这了，将`commit`号复制下来。退出编辑界面。

讲一下git reset参数定义，

具体见官网

```
--soft 回退到指定版本，保留暂存区与工作区。（回退后a分支修改的代码被保留并标记为add的状态（git status 是绿色的状态））
--mixed 回退到指定版本，重置暂存区，但工作区不变。等同于git reset。
--hard 回退到指定版本，重置暂存区与工作区。
--merge 和--hard类似，只不过如果在执行reset命令之前你有改动一些文件并且未提交，merge会保留你的这些修改，hard则不会。【注：如果你的这些修改add过或commit过，merge和hard都将删除你的提交】
--keep 和--hard类似，执行reset之前改动文件如果是a分支修改了的，会提示你修改了相同的文件，不能合并。如果不是a分支修改的文件，会移除缓存区。git status还是可以看到保持了这些修改。
```

a分支的代码我不需要了，以后应该也不需要了。因此：

```
git reset 1（粘贴过来的commit号） --hard
```

a分支代码我还是想要的，只是这个提交我不想要了：

```
git reset 1
```

`git log`查看一下：

```
commit 1
Author: admin <admin@163.com>
Date: Wed May 30 13:00:00 2018 +0800
    init project
```

是我想要的没错了，可以push到远端去了。（这一步很危险，一定要确认好reset的结果确实是你想要的结果，否则，丢失了代码就不要怨我了）

```
git push origin develop
![rejected] develop -> develop (non-fast-forward)
error: 无法推送一些引用到 'git@github.cn:...'
提示：更新被拒绝，因为您当前分支的最新提交落后于其对应的远程分支。
。。。
```

好吧，我的分支确实落后于远程的develop分支。我需要`--force`。

```
git push origin develop --force
Total 0 (delta 0), reused 0 (delta 0)
To git@...
+ 83***...23***** develop -> develop (forced update)
```

⚠️当`HEAD`不指向当前分支的时候，是不能进行`git reset`操作的。

## git revert

还是这个例子

```
develop ----1      3-----
             \   /
branch a       a
```

还是之前的需求，不想要合并a，只想要没合并a时的样子。

操作步骤：

1. 切分支到develop：`git checkout develop`
2. 查看日志：`git log `，还是上面的日志：

```
commit 3
Merge 1 2
Author: admin <admin@163.com>
Date: Wed May 30 15:00:00 2018 +0800

    Merge branch 'feature/a' into 'develop'

    close a

    See merge request !20

commit 2
Author: admin <admin@163.com>
Date: Wed May 30 14:00:00 2018 +0800

    close a

commit 1
Author: admin <admin@163.com>
Date: Wed May 30 13:00:00 2018 +0800
    init project
```

这次和`git reset` 不同的是我不能复制 `commit 1`这个`commit`号了，我需要复制的是`commit 2`的`commit`号。执行`revert`之后会在暂存区产生一个`uncommitted changes`，里面的内容是对`commit 2`所添加的内容的撤销，我们可以把它简称为`commit 4`。

比如`commit 2`添加了一行`123`，`commit 4`就会删除该行的`123`。

### 区别

从回滚这一操作来说

1. ` git revert` 会撤销提交，在当前提交后，新增一次`commit`，新的`commit`的内容和要`revert`的内容正好相反,抵消掉上一次提交导致的所有变化。

2. `git reset`**丢弃提交**，让最新提交的指针指向以前某个时点，该时点之后的提交都从历史中消失。执行`git reset`命令之后，如果想找回那些丢弃掉的提交，可以使用`git reflog`查看历史命令，这样就可以看到之前的新版本的commit_id。⚠️注意，git reset --hard会删除被修改的代码。当进行`git reset`操作，如果远程已经有之前的代码，本地的`commit`会比远程分支落后，需要强推`git push -f`。

   另外，虽然可以用`git reflog`查看历史命令，但是在别的电脑上是无法获取到你的历史命令的，所以这种方法不安全，万一你的电脑突然坏了，这时候就无法回到未来的版本。

从日后继续`merge`之前的老版本来说

1. `git revert`用一次逆向的`commit`“中和”之前的提交，因此日后合并老的`branch`时，这部分改变不会再次出现。`git reset`是直接让最新提交的指针指向以前某个时点，因而和老的`branch`再次`merge`时，这些被回滚的`commit`应该还会被引入。

   对于这一点，我们可以看看以下的对比：

   #### git revert

   现在的需求是我之前已经把`a`分支`revert`了，但是我现在又需要a分支的代码了，我之前都写过一遍总不能再重新写一遍了。我首先想到的方法，把`a`分支再`merge`到`develop`不就好了。
   
   ```
   git merge a
   ```
   
   结果
   
   ```
   Already up-to-date
   ```
   
   这是因为我们之前提交合并的`a`分支的代码还在，因此我们并不能在重新合并`a`分支。
   
   解决办法：`revert`之前`revert`的`commit`。在上面的例子中就是`git revert 4`。于是又新增了一个`commit`，把之前`revert`的代码又重新`revert`回来了。[具体参考](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fgit%2Fgit%2Fblob%2Fmaster%2FDocumentation%2Fhowto%2Frevert-a-faulty-merge.txt)
   
   #### git reset
   
   依旧是上面的需求。
   
   ```
   develop -----1-----4-------5------6
                 \   / \     / \    /
   feature         b      c       d
   ```
   
   现在我将`develop reset`到`commit 4`这里，继续提交我以后的代码
   
   ```
   develop -----1-----4-------7------8
                 \   / \\     /
   feature         b     \ e
                           \------5----d
                             \-c-/
   ```
   
   我现在想重新合并d分支的代码
   
   ```
   develop -----1-----4-------7------8------9
                 \   / \\     /             /
   feature         b     \ e              /
                           \------5----d/
                             \-c-/
   ```
   
   合并过来大概是这个样子的。也就是说`d`分支之前已经合并过`c`分支，因此如果还想要合并`d`分支的话，`c`分支的代码就会同步的带过来。

## 应用场景

1. develop分支已经合并了`a`、`b`、`c`、`d`四个分支，我忽然发现`b`分支没用，这个时候就不能用`reset`了，因为使用`reset`之后`c`和`d`的分支也同样消失了(合并`c`,`d`的指针在`b`之后)。这时候只能用`git revert b`分支`commit`号，这样`c`和`d`的代码依然还在。
2. `git reset` 有很多种用法。它可以被用来移除提交快照，尽管它通常被用来撤销暂存区和工作区的修改。不管是哪种情况，它应该只被用于本地修改——你永远不应该重设和其他开发者共享的快照。