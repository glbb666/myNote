# git学习笔记

## 1.什么是git

git是一个分布式版本控制系统。换句话说，git就是一个文本历史版本管理工具。它可以管理文件的修改，并记录下来每次操作的修改部分。在Git管理下，每个文件的修改、删除，Git都能跟踪，以便任何时刻都可以追踪历史，或者在将来某个时刻可以“还原”。

🧡`github`有以下功能

- `fork`：复制克隆别人的项目，自己可以做出修改。相当远原有项目开了一个分支
- `star`：收藏别人的项目
- `watch`：观察别人仓库的更新
- `pull request`：`fork`别人的项目后，向原项目发起更新合并请求
- `issue`：问题，一般为发现的代码`bug`，解决后可以`close`

## 2.创建版本库

这个库其实就是本地的一个路径，只不过这个路径下的文件，是被git管理的。

```
git init  #在当前目录创建git库
```

注意：

- git库创建的时候，尽量保证文档路径中没有中文。
- 刚刚建立版本库时，没有任何文件处于监控状态。所以需要手动添加。
- 在版本库中，只有文本的修改可以被跟踪（具体修改的内容）。二进制文件无法跟踪，但是可以监控文件大小的改变。

## 3.git库中版本管理

### 1.git的配置

需要配置全局的用户信息

```nginx
git config --global user.name "xiaoming"
git config --global user.email "xm@sina.com"
```

### 2.git的结构

本地git由3部分构成：

- 工作区：代码编写，修改，添加。
- 暂存区：暂时保存修改，等待批量提交到分支
- 分支：保存的一个个的版本。

文件目录就是工作区，但是有一个隐藏目录`.git`，这个不算工作区，而是Git的版本库（包括分支和缓存区）。

### 3.git操作

#### a.查看状态

```nginx
git status
```

- 🍁红色文件名：
  - “*Changed but not updated*”中所列的内容是发现修改但没有保存在暂存区中。
  - “*Untracked files*”中所列的内容是尚未被Git跟踪的内容。
- 🌳绿色文件名：“*Changes to be committed*”中所列的内容是修改内容已经保存至暂存区，等待进入git目录。
- 🈚️文件名：“Nothing to commit,working directory clean”说明工作树已经清空。所有修改的内容都被提交到分支中。当工作区和

#### b.查询修改部分

```nginx
git diff <name> #比较工作区和暂存区
git diff （master） <name> #比较分支与工作区
git diff --cached <file> #比较分支与暂存区
```

- master用分支表达式替换：HEAD、HEAD^、HEAD~1、0c5ee16a6a4c849d0ae0448caa8ff174399c7c3c
- 比较时要切换到对应分支

#### c.添加至暂存区

```nginx
git add <name>#name部分是文件路径
git add ./ #把当前目录的所有修改都提交到暂存区
git add . #把项目中的所有修改都提交到暂存区
```

- 提交时，提交的是修改，而不是文件。如果添加到暂存区后，又进行了修改，必须再次添加。

#### d.提交到本地分支

```nginx
git commit -m ["String"] [<filename>/-a]
git commit --all -m ["String"]#这是一个一次性的操作，修改的内容不经过暂存区，直接提交到分支上
```

- `-m`表示`message`，字符串是对提交的内容的说明（必须）。

如果不加`-m`，会进入`vim`编辑器的模式

```nginx
git commit
```

可以进行编辑，然后`esc`，接着`:q`退出

如果不想继续进行提交，可以`esc`，接着`:q!`退出，此时会终止提交

#### e.撤销与版本回退

- 修改工作区，未添加到暂存区

```nginx
git checkout -- <name>
#这时撤销操作会，将工作区撤销到与暂存区相同的状态。
#如果暂存区为空，会撤销到版本库的状态。
```

- 添加到暂存区，没有添加到本地分支

```nginx
git reset HEAD <file>
#这个操作会将暂存区依照当前版本重置，也就相当于将暂存区文件，退回到工作区。当需要彻底撤销操作时，结合checkout撤销
```

- 本地分支版本回退

  `Head`永远指向最新的一个版本

```nginx
git log #查看分支的历史提交日志，每提交一次都会生成一个记录
git log --oneline #可以看到简洁版的日志
git reflog #查看本地分支的版本切换日志
git reset --hard HEAD^ #回退到上一版本，相当于HEAD~0
git reset --hard HEAD~n #回退到上n个版本
git reset --hard 1094a #跳转到指定版本
#Git的版本回退速度非常快，因为Git在内部有个指向当前版本的HEAD指针，当你回退版本的时候，Git仅仅是把HEAD修改了
```

ps：如果日志过长，显示不全会导致cmd出现问题，这时输入Q来退出

#### f.文件删除

```nginx
git rm <name>
git commit
```

## 4.远程仓库

- 查看本地库绑定的远程库

     ```nginx
     git remote #简略信息
     git  remote -v #详细信息
     ```

- 添加远程库

    ```nginx
    git remote add name git@github.com:xxx/xxx.git
    #name为远程库的名字
    ```

- 解绑本地库与远程库6

    ```nginx
    git remote rm origin
    ```

- 将本地文件件添加到github


1. 先在本地创建ssh公钥和秘钥
2. 将公钥复制到github“SSH Keys”页面（让自己获得修改提交权限，其他人都是只读模式）
3. 在github新建一个仓库
4. 使用git remote add origin git@github.com:xxx/xxx.git把本地库和远程库关联起来。
5. 使用git push -u origin master将本地分支推动到远程库。（添加-u操作，不但推送内容，还会将分支也关联起来，便于之后的提交）

- 从github克隆代码

     ```nginx
     git clone git@github.com:michaelliao/gitskills.git
     #克隆下来的库默认叫origin
     #同时会建git立一个和远程库同名的路径
     ```

## 5.分支管理

每次提交，Git都把它们串成一条时间线，这条时间线就是一个分支。

- 查看分支

     ```nginx
     git branch #命令会列出所有分支，当前分支前面会标一个*号。
     git log --graph #可以看到分支合并图。
     git log --graph --pretty=oneline --abbrev-commit
     #简化分支合并图
     ```

     💥如果没有分支被`tracking`，`git branch`没有内容，需要`git add`成功，让分支被`tracking`之后，`git branch`才会显示被`tracking`的分支

- 切换分支

     ```nginx
     git checkout dev
     ```

- 创建分支

     ```nginx
     git branch xxx #新建分支
     git checkout -b dev #新建并切换到分支
     ```

- 分支重命名

     ```nginx
     git branch –m oldname newname
     ```

- 删除分支

     ```nginx
     git branch -d dev
     git branch -D dev #强制删除未合并的分支
     ```

- 合并分支

     把当前分支与指定分支（`dev`）进行合并

     当前分支指的是`git branch`命令输出的有*的分支

     ```nginx
     git merge dev #快速模式，不保留合并历史
     git merge --no-ff dev #保留合并历史记录
     ```

- 分支合并原理

     - `HEAD`指向的就是当前分支。
     - Git创建一个分支，就是增加一个`dev`指针，并修改`HEAD`的指向，工作区的文件都没有任何变化！所以生成的分支的速度非常快。
     - 从现在开始，对工作区的修改和提交就是针对`dev`分支了，比如新提交一次后，`dev`指针往前移动一步，而`master`指针不变。
     - 合并就是直接把`master`指向`dev`的当前提交，就完成了合并。
     - 合并结束后，就可以将原来的分支删除了。

- 合并异常处理

     - 当两个分支对同一个文件有不同的版本，且版本不是新旧关系，就要去处理两个版本之间的冲突。
     - 通过使用合并命令出现异常时，主分支会进入MERGING模式，它会将合并分支的内容合并入主分支内文件内（也就是主分支的文件内有两套代码），并标注有冲突的两个版本。
     - 这时候其实已经完成合并，只不过主分支内有两套版本，我们需要手动修改成最新版本，然后再提交一次。

- 合并模式

     - 一般如果没有冲突的情况下，可以使用快速模式迅速进行合并，但是快速模式会导致，合并分支没有记录。如果将原有分支删除掉，log中将不再显示git合并信息。
     - 如果是合并异常，手动处理的话，由于版本冲突，没有使用快速合并。这是log中会记录合并历史
     - 使用git merge --no-ff -m "merge with no-ff" dev，就可以在分支历史上就可以看出分支信息

- 紧急修复bug

     - 在项目的开发过程中，一旦出现加急情况，需要立刻放下当前工作的分支，立刻新建分支去处理bug

     - 但是git只有版本库（commit）才会保存版本，也就说暂存区和工作区是公用的，所以在修复工作开始之前，需要将代码commit保存，将暂存区清理干净，再去修复bug，保证修复合并的代码中，没有正在开发，未打算合并的部分。

     - 如果工作只进行到一半，而版本库有明确的版本规范，暂时还没法提交。但要先修复bug，如何将暂存区清空呢？可以先把工作区暂时保存，待bug修复完再恢复。注意这里只会保存工作区，暂存区的内容不会记录，所以记得再add添加。

     - ```nginx
          git stash #保存工作区
          git stash list #显示存储库
          git stash apply #恢复存储的第一个工作区（不删除）
          git stash pop #恢复存储的第一个工作区（删除）
          git stash drop #删除存储的第一个工作区
          git stash apply stash@{0}  #恢复指定的工作区
          ```

## 6.Github多人合作开发

​	我们要知道，一个大项目一个人是很难完成开发的，需要多个人去完成。当使用git上的多人开发有两种类型，分别对应维护者，和贡献者。

- 维护者：有仓库主人邀请一些人参加项目的开发，这些人共用一个代码库，具有对这个仓库的读写权限，通过不同的分支进行合作开发。这些人都有合并分支的能力。
- 贡献者：这些人没有收到邀请，没有原仓库的写入权限，但是可以通过fork，克隆一个一模一样的库进行开发。然后通过pull requests来发出仓库间合并的请求。由仓库的维护者，来审核代码，处理冲突，然后合并，或是驳回请求。

多人开发流程：

1. 仓库主人建立仓库，并添加维护者。
2. 维护者收到邮件，并同意加入，获得权限。
3. 通过clone将远程仓库克隆到本地。
4. 按照项目分支规范联合开发。

- 注意：在克隆的时候只会克隆master分支，需要克隆其他分支需要将这样新建分支。即指定远程的新建分支。

     ```nginx
     git checkout -b dev origin/dev
     新的解决办法：先git branch -a列出本地和远程所有分支，比如某个远程分支为 origin/daily/1.4.1，然后再直接使用git checkout origin/daily/1.4.1
     ```

- 从本地推送分支，使用`git push origin branch-name`，如果推送失败，先用`git pull`抓取远程的新提交；
- 建立本地分支和远程分支的关联，使用`git branch --set-upstream branch-name origin/branch-name`；

## 7.使用标签管理版本

​	在更新一个版本的时候，一般都是使用哈希值来表示一个代码版本，但是这样不方便定位和查找。所以发布一个版本时（一个重要的提交），在版本库中打一个标签（tag），这样，就唯一确定了打标签时刻的版本。标签其实它就是指向某个commit的指针，但是标签不能移动，指向固定的位置。所以，tag就是一个让人容易记住的有意义的名字，它跟某个commit绑在一起。

- 添加标签

     ```nginx
     git tag v1.0 #最新提交的commit上的打标签
     git tag v0.9 f52c633 #在指定commit上的打标签
     git tag -a v0.1 -m "version 0.1 released" 1094adb
     #在指定commit上的打标签 并添加说明
     ```

- 显示标签

     ```nginx
     git tag <tagname> #显示指定标签
     git tag #显示所有标签
     ```

- 删除标签

     ```nginx
     git tag -d v0.1 #删除本地标签
     git push origin :refs/tags/<tagname> #删除远程标签
     ```

- 推送标签

     ```nginx
     git push origin <tagname> #推送指定标签
     git push origin --tags #推送所有标签
     ```

- 注意

     ​	因为创建的标签都只存储在本地，不会自动推送到远程。所以，打错的标签可以在本地安全删除。

## 8.其他git技巧

### 1.忽略文件

​	我们知道，在github中有一些文件是不需要上传的，比如一些缩略图，中间文件等等。我们可以在git目录上设置一个.gitignore文件，里面通过相关的语法，来告诉git需要忽略哪些文件，这样便于我们上传文件。

```
#               表示此为注释,将被Git忽略
*.a             表示忽略所有 .a 结尾的文件
!lib.a          表示但lib.a除外
/TODO           表示仅仅忽略项目根目录下的 TODO 文件，不包括 subdir/TODO
build/          表示忽略 build/目录下的所有文件，过滤整个build文件夹；
doc/*.txt       表示会忽略doc/notes.txt但不包括 doc/server/arch.txt
 
bin/:           表示忽略当前路径下的bin文件夹，该文件夹下的所有内容都会被忽略，不忽略 bin 文件
/bin:           表示忽略根目录下的bin文件
/*.c:           表示忽略cat.c，不忽略 build/cat.c
debug/*.obj:    表示忽略debug/io.obj，不忽略 debug/common/io.obj和tools/debug/io.obj
**/foo:         表示忽略/foo,a/foo,a/b/foo等
a/**/b:         表示忽略a/b, a/x/b,a/x/y/b等
!/bin/run.sh    表示不忽略bin目录下的run.sh文件
*.log:          表示忽略所有 .log 文件
config.php:     表示忽略当前路径的 config.php 文件
 
/mtk/           表示过滤整个文件夹
*.zip           表示过滤所有.zip文件
/mtk/do.c       表示过滤某个具体文件
 
被过滤掉的文件就不会出现在git仓库中（gitlab或github）了，当然本地库中还有，只是push的时候不会上传。
 
需要注意的是，gitignore还可以指定要将哪些文件添加到版本管理中，如下：
!*.zip
!/mtk/one.txt
 
唯一的区别就是规则开头多了一个感叹号，Git会将满足这类规则的文件添加到版本管理中。为什么要有两种规则呢？
想象一个场景：假如我们只需要管理/mtk/目录中的one.txt文件，这个目录中的其他文件都不需要管理，那么.gitignore规则应写为：：
/mtk/*
!/mtk/one.txt
 
假设我们只有过滤规则，而没有添加规则，那么我们就需要把/mtk/目录下除了one.txt以外的所有文件都写出来！
注意上面的/mtk/*不能写为/mtk/，否则父目录被前面的规则排除掉了，one.txt文件虽然加了!过滤规则，也不会生效！
 
----------------------------------------------------------------------------------
还有一些规则如下：
fd1/*
说明：忽略目录 fd1 下的全部内容；注意，不管是根目录下的 /fd1/ 目录，还是某个子目录 /child/fd1/ 目录，都会被忽略；
 
/fd1/*
说明：忽略根目录下的 /fd1/ 目录的全部内容；
 
/*
!.gitignore
!/fw/ 
/fw/*
!/fw/bin/
!/fw/sf/
说明：忽略全部内容，但是不忽略 .gitignore 文件、根目录下的 /fw/bin/ 和 /fw/sf/ 目录；注意要先对bin/的父目录使用!规则，使其不被排除。

```

### 2.设置简化命令

​	有些git命令会比较长，如果经常使用，可以自定义git命令，达到更快git操作速度。原理就是通过配置文件，在输入命令的工程中做了字符串替换。

```nginx
git config --global alias.st status
git config --global alias.unstage 'reset HEAD'
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```

- 配置Git的时候，加上`--global`是针对当前用户起作用的，如果不加，那只针对当前的仓库起作用。

- 每个仓库的Git配置文件都放在`.git/config`文件中。
- 配置别名也可以直接修改这个文件，如果改错了，可以删掉文件重新通过命令配置

### 3.git pull 和 git fetch 有什么区别？

- 首先，你的每一个操作都是要指明【来源】和【目标】的，而对于 pull 来说，【目标】就是当前分支；

- 其次，你得清楚 git 是有 tracking 的概念的，所谓 tracking 就是把【来源】和【目标】绑定在一起，节省一些操作是需要输入的参数。

- 那么，假设你的 master 和 develop 都是 tracking 了的，于是：
```nginx
当你在 master 下，git pull等于 fetch origin，然后 merge origin/master；
当你在 develop 下，git pull等于 fetch origin，然后 merge origin/deve
```


