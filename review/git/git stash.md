- `git stash`：执行存储，没有备注，查找时不方便识别。
- `git stash save "save message"` : 执行存储时，添加备注，方便查找。
- `git stash -u` ：存储未追踪的文件。

- `git stash list `：查看`stash`了哪些存储。
- `git stash show` ：显示做了哪些改动，默认`show`第一个存储,如果要显示其他存贮，后面加stash@{$num}，比如第二个 `git stash show stash@{1} `从0开始。
- `git stash show -p `: 显示第一个存储的改动，如果想显示其他存存储，命令：`git stash show stash@{$num} -p` ，比如第二个：`git stash show stash@{1} -p`。
- `git stash apply` ：应用某个存储，但**不会把缓存堆栈中对应记录删除**，默认使用第一个存储，即`stash@{0}`，如果要使用其他个，`git stash apply stash@{$num} `， 比如第二个：`git stash apply stash@{1}`。
- `git stash pop `：恢复之前缓存的工作目录，**将缓存堆栈中的对应记录删除**，并将对应修改应用到当前的工作目录下,默认为第一个`stash`,即`stash@{0}`，如果要应用并删除其他`stash`，命令：`git stash pop stash@{$num}` ，比如应用并删除第二个：`git stash pop stash@{1}`。
- `git stash drop stash@{num}` ：丢弃`stash@{num}`存储，从列表中删除这个存储。
- `git stash clear` ：删除所有缓存的`stash`。

#### 注意

不建议同一时间段在不同分支都使用 `git stash`，涉及到多个分支的情形还是先  `commit` 较好，不push到远程，下次 commit 时可用 `--amend` 合到上次提交中。