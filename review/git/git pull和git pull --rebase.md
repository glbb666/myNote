在一般情况下，加与不加 `--rebase` 是没有区别的。

然而，从上面说的 `git rebase` 功能得知，当某个分支使用了`git rebase`方法后，它就会与其远程分支发生分离。而当你 `pull` 时使用 `git pull`，则会变成你的本地分支和远程分支合并，并产生一个`merge commit`，这样是没有那么好看的。

要解决以上问题，不再出现自动生成的 merge commit，那么只要在执行 `git pull origin master` 的时候带上 `--rebase` 即可。因此推荐在任何时候 `pull` 远程分支，最好加上 `--rebase` 参数。

