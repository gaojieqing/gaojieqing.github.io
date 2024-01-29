## 什么是git
* 分布式  
分布式是相对集中式来说，集中式就是把数据集中保存在服务节点。所有的开发者都从服务节点获取数据。比如SVN 就是典型的集中式版本控制系统。  
分布式与之相对的，git 数据不止保存在服务器上，同时也会保存在本地计算机上，也就是说代码从仓库完整的镜像下来之后，每个人电脑上都是一个完整的代码库。  
* 完整性
    * 克隆现有的仓库，在多人合作的开发的情况下，git clone 克隆下该仓库服务器几乎所有的数据，执行git clone的时候，远程仓库中每个文件的每一个版本都会被拉取下来。所以每个开发者本地都是一个完整的版本。可以用任何一个克隆下来的用户端来重建服务器上的仓库。
    * 从git 数据层面来说，直接记录快照，而非差异比较
    * git每次提交更新，对当时文件制作一个快照，保存文件的索引，如果文件没有修改，保留一个链接指向上一个版本
    * git中所有的数据存储都计算校验和，然后以校验和来引用，所有修改任何文件 git都能发现

## git 内部原理
### .git目录结构
* 从根本来讲git是一个内容寻址,核心是一个键值对数据库（修改插入任何内容返回一个键值），将修改的文件保存成数据对象、树对象、提交对象
* 在新目录或者已有目录执行git init时，会创建一个.git 目录，这个目录是 git 的核心，几乎包含所有git 的存储和操作对象
* 在项目目录下使用 命令 tree .git ，查看该目录的结构
```
tree .git
.git
├── HEAD
├── config
├── description
├── hooks
│   ├── applypatch-msg.sample
│   ├── commit-msg.sample
│   ├── fsmonitor-watchman.sample
│   ├── post-update.sample
│   ├── pre-applypatch.sample
│   ├── pre-commit.sample
│   ├── pre-merge-commit.sample
│   ├── pre-push.sample
│   ├── pre-rebase.sample
│   ├── pre-receive.sample
│   ├── prepare-commit-msg.sample
│   ├── push-to-checkout.sample
│   └── update.sample
├── info
│   └── exclude
├── objects
│   ├── info
│   └── pack
└── refs
    ├── heads
    └── tags
```

### .git文件简介
* `description` 对于空的Git仓库，此文件内容为空。对于非空的Git仓库，描述该项目的文本，一般仅供 git Web程序使用。
* `config` 是一个 txt 文件，里面记录了当前仓库的 git 设置，如作者信息、文件模式等。
    * \[`core`\]：包含与 Git 核心功能相关的配置选项，如仓库路径、忽略文件权限等。
    * \[`remote "<remote-name>"`\]：用于定义与远程仓库的连接和交互的配置选项，可以指定远程仓库的 URL、分支跟踪等。
    * \[`branch "<branch-name>"`\]：用于定义分支相关的配置选项，如分支的追踪关系、合并策略等。
    * \[`user`\]：用于设置 Git 用户的姓名和邮箱地址，这些信息会出现在提交记录中。
    * \[`alias`\]：用于定义 Git 命令的别名，可以简化常用命令的输入。
    > 注意： config 文件夹是每个 Git 仓库独立的，不会被版本控制。如果要对全局的 Git 配置进行更改，可以使用 `git config --global` 命令来修改全局配置文件。
* `info` 存放仓库的一些辅助性信息，包含一个全局性排查（global exclude）主要放不希望记录在 `.gitignore` 文件中的忽略模式。
* `hooks` 是一个特殊的目录，其中包含了可以在 git 执行任何操作前后运行的脚本。
* `objects` 存放的是 git 的对象，比如关于仓库中的文件、提交等的数据（commits、trees和blobs）。我们稍后会对此进行深入探讨。
* `refs` 是用来存放引用的目录。例如，`refs/heads` 里存放的是分支的引用，而 `refs/tags` 则存放的是标签的引用。我们将进一步深入了解这些文件的内容。
* `HEAD` 表示仓库的当前 head。根据你设置的默认分支，它可能是 `refs/heads/master` 或 `refs/heads/main` 或其他你设定的名字。实际上，它指向 `refs/heads` 这个文件夹，并关联了一个名为 `master` 的文件，但该文件目前还不存在。只有在你完成首次提交后，`master` 文件才会生成。
* `logs` 存储了每个引用（分支、标签等）的修改历史。
* `COMMIT_EDITMSG` 存储了当前正在编辑的提交信息，在执行 `git commit` 命令时，会先打开这个文件，让你编辑提交信息。
* `ORIG_HEAD` 当我们进行了一些「危险操作」时，比如 `git reset`、`git merge`、`git rebase` 等操作时，Git 会将当前 `HEAD` 指向的的 Commit-ID 原值保存至 `ORIG_HEAD` 文件内。需要注意的是，类似 `git commit` 等操作并不会更新 `ORIG_HEAD` 的内容。这样的话，加入我们执行了一些「误操作」时，可以利用 `git reset --hard ORIG_HEAD` 回退至上一步。
* `FETCH_HEAD` 指的是某个分支在远程仓库上最新的状态。每一个执行过 `git fetch` 操作的本地仓库都会存在一个 `FETCH_HEAD` 列表，这个列表保存在 .git/FETCH_HEAD 文件中。`FETCH_HEAD` 文件中的每一行对应着远程仓库的一个分支。当前本地分支指向的 `FETCH_HEAD` 就是该文件中的「第一行」对应的分支。接着，与 `FETCH_HEAD` 相关的是 `git pull` 操作。`git pull` 等价于 `git fetch` + `git merge FETCH_HEAD` 两个步骤的结合。
* `packed-refs` 是一个存储远程引用的文件。包含了远程分支和标签的引用信息，这些引用指向远程仓库中的特定提交。`packed-refs` 文件的目的是提高性能，当引用过多时，会将其中一些引用以压缩形式存储在该文件中，以减少 .git 文件夹的大小和读取时间。在该文件中，每个引用包含一个 SHA-1 值，该值指向特定的提交。这样，在使用 Git 命令时，Git 可以更快速地访问和检索这些引用。
* `index` 是一个二进制文件，也成为暂存区（staging area）或者索引（index）。暂存区是贯穿于整个Git使用流程的重要概念，所以index文件就很重要。由于是二进制所以我们无法查看具体内容，但是可以用`git ls-files --stage`命令查看暂存区里面的文件
> 注意： index 文件是 Git 维护的，不应手动修改或删除这个文件，以免导致仓库状态不一致。Git 会自动更新 `index` 文件，以反应当前暂存区的状态。

### objects
* 初始化一个test  
在.git/objects 可以看到目录进行初始化，创建了pack和info子目录
* 手动存入数据  
`echo 'test content' | git hash-object -w -–stdin`使用命令存储数据对象，这时候返回一个40个字符的校验和，这是一个SHA-1 哈希值
* 40位哈希值是什么？  
前面说到object是存储数据内容，这个就是 git 存储的方式：一个文件的内容加上特定的头部信息一起的SHA-1校验和。取其中前2位作为文件夹名字后面38位作为文件名。
* 如何取出数据？  
前面说到写入数据返回键值，那么又如何根据键值读取到文件内容 `git cat-file -p [SHA-1]`
* 文件内容是否可还原项目版本？  
显然是不可能，这个只是存储了文件内容没有保存多个文件之间的结构
* 查看哈希文件类型  
`git cat-file –t [SHA-1]`

```
$ git init test
$ cd test
$ find .git/objects
.git/objects
.git/objects/pack
.git/objects/info
$ echo 'test content' | git hash-object -w -–stdin
d670460b4b4aece5915caf5c68d12f560a9fe3e4
$ git cat-file -p d670460b4b4aece5915caf5c68d12f560a9fe3e4
test content
$ echo 'version 1' > test.txt
$ git hash-object -w test.txt
83baae61804e65cc73a7201a7252750c76066a30
$ echo 'version 2' > test.txt
$ git hash-object -w test.txt
1f7a7a472abf3dd9643fd615f6da379c4acb3e3a
$ find .git/objects -type f
.git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4
.git/objects/1f/7a7a472abf3dd9643fd615f6da379c4acb3e3a
.git/objects/83/baae61804e65cc73a7201a7252750c76066a30
```

### 五种存储对象
* `blob` 是一堆字节，通常是二进制标识，保存的是文件的内容。当我们对文件进行修改时，变化的文件会生成一个`blob`对象，记录这个版本文件的内容，针对没有变化的文件就指向上一个版本的指针，不会生成新的`blob`对象。
* `tree` 有点类似于目录，像是操作系统中的文件夹，解决了文件名保存的问题允许多个文件组织到一起。包含一个或者多个`tree`对象。
* `commit` 指向一个树对象，并包含一些表明作者以及父`commit`的元数据。记录我们的提交历史和提交的信息。
* `tag` 指向一个`commit`对象，包含一些元数据。
* `refrerence` 指向一个commit或者tag 对象。存储在.git/refs/文件下，文件可能包含（heads，remotes，stash，tags）

## git 工作区
#### 四个工作区域
Git 有四个工作区域：工作区域（Working Directory）、暂存区（Stage\Index）、本地仓库（Repository）、远程仓库（Remote Directory）
![](https://static-resource-1g8.pages.dev//picgo/git%E5%B7%A5%E4%BD%9C%E5%8C%BA%E7%8A%B6%E6%80%81.png)
* `workspace`：工作区，就是平时存放项目代码的地方。
* `index`：暂存区，用于临时存放你的改动，事实上它只是一个文件（.git/index），保存即将提交到文件列表信息。
* `repository`：仓库区（或版本库），就是安全存放数据的位置，这里面有你提交到所有版本的数据。其中 `HEAD` 指向最新放入仓库的版本。
* `remote`：远程仓库，托管代码的服务器，可以简单的认为是你项目组中的一台电脑用于远程数据交换。

#### 工作流程\[简单\]
git的工作流程一般分为以下几步：
1. 在工作目录中添加、修改、删除文件(`modified`)；
2. 将需要进行版本管理的文件放入暂存区(`staged`)；
3. 将暂存区的文件提交到本地仓库(`committed`)；
4. 将本地仓库的文件推送到远程仓库；
因此，git管理的文件有三种状态：已修改(`modified`)、已暂存(`staged`)、已提交(`committed`)。

#### 文件的四种状态
版本控制就是对文件的版本控制，要对文件进行修改、提交等操作，首先要知道文件当前在什么状态，不然可能会提交了现在还不想提交的文件，或者要提交的文件没提交上。  
git不关心文件两个版本之间的具体差别，而是关心文件的整体是否有改变，若文件被改变，在添加提交时就生成文件新版本的快照，而判断文件整体是否改变的方式就是用`SHA-1`算法计算文件的校验和。
![](https://static-resource-1g8.pages.dev//picgo/git%E6%96%87%E4%BB%B6%E7%9A%84%E5%9B%9B%E7%A7%8D%E7%8A%B6%E6%80%81.png)
1. `Untracked`：未跟踪，此文件在文件夹中但并没有加入到git库，不参与版本控制。通过`git add`状态变为`Staged`
2. `Unmodified`：文件已经入库，未修改，即版本库中的文件快照内容与文件夹中完全一致，这种类型的文件有两种去处，1. 如果被修改，而变成`Modefied`,2. 如果使用`git rm`移除版本库，则成为`Untracked`文件。
3. `Modified`：文件被修改过，仅是修改，并没有进行其他操作，这个文件也有两个去处。通过`git add`变为`Staged`，通过`git checkout -- file`撤销修改，回到`Unmodified`状态，这个`git checkout`即从库中取出文件，覆盖当前修改。
4. `Staged`：暂存状态。执行`git commit`之后，暂存区的文件提交到本地库，这时库中的文件和本地文件又变为一致，文件为`Unmodify`状态，执行`git reset HEAD filename`取消暂存，文件状态为`Modified`。  
![](https://static-resource-1g8.pages.dev//picgo/git%E7%8A%B6%E6%80%81%E6%B5%81%E8%BD%AC.png)

## Git 如何存储数据实现运行
### git clone 发生了什么？
1. 为 git 仓库 中的每个 branch ，在本地创建一个远程跟踪分支 branch
2. 复制远程仓库下 object/ 文件夹中的内容到本地仓库
3. 为所有远程分支创建本地的跟踪分支 `.git/refs/remote/***`，远程分支中当前活跃的分支 `.git/HEAD`
4. 在当前分支执行 git pull，自动在 config 写入 origin ，写入对应的地址

### git pull 和git fetch
1. 相当于是从远程获取最新版本并merge到本地
2. `git fetch` 相当于是从远程获取最新到本地，不会自动merge， merge到本地；创建并更新所有远程分支的本地远程分支；`git fetch origin branch1` ；设定当前分支的 FETCH_HEAD为远程服务器的branch1分支；`git fetch origin branch1:branch2`使用远程branch1分支在本地创建branch2，但是不会切换到branch2分支
3. `git pull` 是首先，基于本地的FETCH_HEAD记录，对比本地的FETCH_HEAD记录与远程仓库的版本号，然后执行`git fetch`当前分支指向远程的最新数据，然后利用`git merge` 将其与本地的当前分支合并；`git fetch origin master` `git merge FETCH_HEAD`
4. 这将更新git remote 中所有的远程repo 所包含分支的最新commit-id, 将其记录到.git/FETCH_HEAD文件中 

### git add 发生了什么？
1. 在 .git/object/ 文件夹中添加修改或者新增文件对应的 blob 对象；在 .git/index 文件夹中写入该文件的名称及对应的 blob 对象名称；
2. `git add .` 提交新文件(new)和被修改(modified)文件，不包括被删除(deleted)文件
3. `git add -A .`  （`git add --all`）-A 表示将所有的已跟踪的文件的修改与删除和新增的未跟踪的文件都添加到暂存区。
4. `git add -u .` 提交tracked file和删除的文件添加到暂存区， add -u 不会提交新文件（untracked file） ，被删除的文件被加入到暂存区再被提交并推送到服务器的版本库之后这个文件就会从git系统中消失了。
5. `git ls-files -s` 显示暂存的条目的相关信息（模式位，文件哈希后的值，暂存号和文件名）。

### git commit 发生了什么？
1. 新增 tree 对象，有多少个修改过的文件夹，就会添加多少个 tree；
2. 新增 commit 对象，其中的 tree 指向最顶端的 tree，此外还包含一些其它的元信息， tree 对象中会包含一级目录下的子 tree 对象及 blob 对象，由此可构建当前 commit 的文档快照。tree对象可以包含一个tree 或者多个tree对象记录，每一个记录含有一个指向Blob或者子tree对象的哈希值
3. 我们 commit 之后，这些信息就会存放在 object 下面。如果要查看  当前分支 根目录的最新提交指向的 tree 对象`git cat-file -p master^{tree}`
4. `git commit -m “message”` `git commit --amend` 追加提交，在不增加新的 commit的前提下追加到上一次commit 
5. commit  指向树的索引，那么暂存区创建了提交对象，那么暂存区的内容还在吗？ 树对象，暂存区没变，创建的树对象就是同答案是肯定的commit命令时会根据暂存区创建一个，也就是不会重复创建
6. `git commit` 命令创建了一个commit对象，实际上底层调用的是 `git commit-tree` 命令。每一次我们 commit 的时候，都会记录上一个 commit 的 SHA-1值，这样一个个的 commit 就串起了我们的提交记录
7. git怎么知道新 commit 对象的上一个 commit 对象的呢？实际上就是上面commit-tree 命令中的第二个参数---父提交对象，每个提交对象都保存了上一个父提交对象的引用，这样就串起来一个提交历史记录了 
8. 前面讲到git add 已经把文件的索引保存了暂存区，所以commit 不用创建对象
9. 如果暂存区存在目录关系，就会先创建树对象来记录文件和目录关系。创建一个树对象，代表当前项目快照，这个树对象里面包含的就是上述信息，也就是所有要保存记录的数据。 然后用这个树对象，配置中的user.name和email，以及当前的时间戳和 -m 参数后面的内容生成提交对象

### git checkout 发生了什么？
1. `git checkout` 实际操作的是 HEAD，`.git/HEAD` 指向本地仓库当前操作的分支，实际也是直接或者间接的指向了某个 commit 对象
2. `git checkout HEAD` 的缩写 ，这个操作是 可以清除没有添加到缓存的更改，但是不会影响已经缓存的更改，原因在于其实缓存过的文件已经是另外的一个文件了
3. `git checkout`  撤销本地修改 `git reset` 后面会讲到两者区别。既可以带一个commit号，又可以带着一个路径
4. `git checkout  分支名字`，切换分支，刚刚提到了 `.git/HEAD` 中的内容，更新工作区域内容为 切换的分支的 所指向的 commit 对象的内容
5. 我们切换分支的时候 git 内部又是如何操作运行的，这要归功于前面提到的.git/HEAD `cat .git/HEAD` 查看HEAD 内容HEAD指向的是当前分支名master。那么master 对应的又是当前最新的一次提交id `cat .git/refs/heads/master` 假如我们再一次提交， HEAD对应的ref没有变化，但是master对应的commit ID却会改变。如果我们切换到分支A 那么HEAD 同时会指向分支A

### git reset 发生了什么？
1. 撤销add 的操作：`git reset HEAD <file> ` 取消add后的操作并保留工作区的修改 `git checkout -- <file>` 如果继续执行这个 可撤回工作区的修改 ,但是不能包含没有跟踪的文件
2. 撤销git commit操作：`git reset --soft <commit_id>` 可以回退到某个commit并保存之前的修改(可用合并后面几个commit) <commit_id>从git log中取，取前7位即可`git reset --hard <commit_id>`  回退到某个commit不保留之前的修改,包含没有跟踪的文件，就像没有发生过一样
3. 撤销git push操作：`git revert<commit_id>` git revert 会产生一个新的 commit，它和指定 SHA 对应的 commit 是相反的（或者说是反转的）。 任何从原先的 commit 里删除的内容会在新的 commit 里被加回去，任何在原先的 commit 里加入的内容会在新的 commit 里被删除。因为它并不会改变历史
4. push 之后删除一些不该提交的文件：`git rm file`  删除远程和本地 `git rm --cached filename` 删除远程保留本地

### reset 和 checkout
![](https://static-resource-1g8.pages.dev//picgo/reset%E5%92%8Ccheckout.png)
1. 假设我们有 master 和 develop 分支，它们分别指向不同的提交；我们在分支develop 所以HEAD 指向它，
2. 如果我们运行 git reset master， develop 会和 master 指向同一个提交
3. 而如果我们运行 git checkout master 的话，develop 不会移动，HEAD 自身会移动，指向 master。
4. reset 会移动 HEAD 分支的指向，而 checkout 则移动 HEAD 自身。

### reset 和 revert
![](https://static-resource-1g8.pages.dev//picgo/reset%E5%92%8Crevert.png)
1. git reset的作用是修改HEAD的位置，即将HEAD指向的位置改变为之前存在的某个版本
2. git revert是用于“反做”某一个版本，以达到撤销该版本的修改的目的
3. 如果想要撤销版本二，但又不想影响撤销版本三的提交，就可以用 git revert 命令来反做版本二，生成新的版本四，这个版本四里会保留版本三的东西，但撤销了版本二的东西。
4. reset 是回到指定的commit 的版本，指针移动；revert 指针后移；
5. revert 是删除指定commit 操作内容，指定的commit 之前和之后都不受影响。与此同时这个操作一个新的commit 进行提交
6. reset 撤销某次提交但是此次之后的修改会退会到暂存区
7. 用法1： 当前commit为HEAD，然后每一个commit加一得到要撤销的commit 为HEAD~4； git revert HEAD~4
8. 用法2：git revert commitid
7. 回退连续几个： git revert X...Y （x,y]

### git branch 发生了什么？
1. 前面在介绍 `.git/refs/heads` 的时候分支的本质就是 一个指向提交对象的 可变的指针；创建一个分支，相当于往一个文件中写入 41 字节（40 个字符一个换行符）`.git/refs/remotes` 存储 远程分支，也可以说是一个本地的备份
2. `git branch -vv`：查看本地分支和远程分支的映射关系 `git push --set-upstream origin branch-name` 本地分支关联到远程分支 `git branch -d dev` 删除分支，如果有未merge的会失败；有必要的情况下，删除远程分支：`git push origin --delete branch-name`；如果删除不了可以强制删除，`git branch –D branch-name`
3. `git branch –r`  查看远程分支列表 `git branch -a`  查看所有的分支包含远程和本地 `git branch 分支名` 创建分支 `git branch -m oldName newName` 重新命名
4. `git checkout --track origin/branch_name` 本地会新建一个分支名叫 branch_name ，会自动跟踪远程的同名分支 branch_name

### git merge  发生了什么？
1. 是一个合并一个融合，希望在当前分支上往前走 `git merge` 命令用于合并指定分支到当前分支。也就是说你要合并到哪个分支就切换到哪个分支，然后merge 指定的分支
2.  fast-forward（无merge节点）：实现无节点merge，其实是用到merge指令的默认特性——快进（Fast-forward），因为快进方式的merge不会产生节点，只会移动分支的HEAD指针
3. 有merge 节点：Merge 有冲突手动解决完冲突后，add 冲突的文件，再commit，这样已经产生了一个merge节点。如果要求强制产生merge节点，使用 --no-ff 参数告诉merge，我们不需要快进（no-ff : no fast forward）`git merge --no-ff -m "merged dev" dev`
4. 单节点merge：需要参数 squash 可以理解为压缩提交squash和no-ff非常类似，区别是不会保留对合入分支的引用。也就是说，squash把dev分支的代码压缩在一起，在master分支只提交一次。`git merge --squash dev` `git commit –m "squash dev"`
![](https://static-resource-1g8.pages.dev//picgo/%E4%B8%89%E7%A7%8D%E5%90%88%E5%B9%B6.png)

### git rebase 发生了什么？
1. 找到两个分支最近的共同祖先，根据当前分支的提交历史生成一系列补丁文件，然后以基底分支最后一个提交为新的提交起始点，应用之前生成的补丁文件，最后形成一个新的合并提交。从而使得变基分支成为基底分支的直接下游。rebase一般被翻译为变基。
2. 永远不要rebase 一个在中央存在的分支，只能自己使用的私有的分支
3. 对一个分支做“变基”操作，这意味着改变这个branch的初始commit(我们知道commits本身组成了一颗树）。它会在新的base上一个一个地运行这个分支上的所有commits
4. 注意rebase往往会重写历史，所有已经存在的commits虽然内容没有改变，但是commit本身的hash都会改变!!!）
5. 首先保证本地代码是最新的：`git checkout master` -> `git pull`；在branch1 分支rebase ：`git checkout branch1`  -> `git rebase master`；一般会存在冲突 ，手动解决完冲突后，这个和merge 不一样的是rebase 冲突是一个一个解决的`git add –u`  -> `git rebase --continue`；branch1 分支 push 到远程
6. `git rebase --onto` 如果合并两个分支的时候，要跳过某一个分支，这时候用到 onto 参数。假设topic 的上游是next ，`git rebase --onto master next topic` 将next 之后的提交，放在master 分支上
7. `git rebase -i` 交互式rebase，可以修改提交信息，删除提交，合并提交等操作。`git rebase -i HEAD~3` 对最近三次提交信息进行修改。`git rebase –abort` 终止rebase 操作，回到rebase 前的状态。`git rebase –skip` 跳过当前提交，继续执行后面的操作。`git rebase –continue` 继续rebase 操作。

### 修改多个提交信息
1. `git commit –amend`
2. `git rebase -i <commitid>` 打开变基脚本之后需要修改pick 为reword
3. `git rebase -i HEAD~4`

### Rebase风险
1. 一旦分支中的提交对象发布到公共仓库，千万不要对该分支进行变基
2. `git rebase branch` 和`git rebase commit-id` 都重置了提交的 SHA-1的校验和，如果你把变基的提交推送到远端，别人更新到代码，内容相同却有不同的SHA-1值，因此会进行再一次的合并，于是两个作者commit 信息完全相同的提交 SHA-1 不同
3. 谨记：一旦分支中的提交对象发布到公共仓库，就千万不要对该分支进行衍合操作

## 工作中的git
### 撤销add修改
* `git reset HEAD <file>` 取消add 操作但是工作区修改还在
* `git checkout -- <file>` 取消add 操作，工作区修改也取消了
* `git reset HEAD` 如果后面什么都没有加 就是取消这次add 所有的文件
* `git reset –hard HEAD` 全部的更改都恢复，谨慎

### 撤销commit
* `git reset –soft HEAD^` 这样仅仅撤销commit ，不撤销 add 。代码保留,HEAD^指上一个版本可以写 HEAD~1,也可以具体的commit-id
* `git reset --hard <commit_id>` 删除工作区，撤销commit ，撤销 add。就想没有发生过一样，代码不会保留。这个需要谨慎操作
* `git reset --mixed HEAD^` 不删除工作区代码改动，撤销commit 并且撤销 add 操作和`git reset HEAD^` 效果一样
* `git commit –amend`  修改最近的 commit 信息；项目初始化提交的第一个commit `git update-ref -d HEAD`

### 删除某个commit
* `git reset –hard commit-id` 删除某个commit，保留工作区代码
* `git rebase –i commit-id` (commit-id 为要删除的commit的下一个commit号 ) 编辑 drop 目标commit

### 挑选commit提交
* 场景：在test分支开发了 几个功能的commit 点，但是有个commit 提交的功能突然不能上线，但是其他的正常上线
* Master 的commit1 为公共节点，然后切分支提交了 commit2 和commit3 以及commit4 三个功能节点的提交，与此同时，主分支提交了commit5，这时候如果我们不要commit3上线，只有commit2 和commit4
* 这时候可以在test 分支上拉新的分支test1 ，然后利用git rebase –i 去除commit3 的提交，然后此时剩下commit2 和commit4；
然后git rebase master  然后切到master merge test1 分支
* 如果后期想要发布commit3 功能，可以：
    * `rebase master 然后切到主分支` `merge test 分支`;
    * `git cherry-pick commit-id`，cherry-pick可以跟多个commit-id..commit-id

### stash不小心drop如何恢复？
* `git stash save ‘message’` 添加备注方便查找；`git stash show` 后面加stash@{$num}，比如第二个 `git stash show stash@{1}`
* `git stash apply` :应用某个存储，但是不会从列表中删除；`git stash pop` ：命令恢复之前缓存的工作目录，将缓存堆栈中的对应stash删除；
比如应用并删除第二个：`git stash pop stash@{1}`
* `git stash drop stash@{$num}` ：丢弃stash@{$num}存储，从列表中删除这个存储
* 如果删除了，首先`git fsck --lost-found`；会看到 一条一条的记录 类似 `dangling commit 7010e0447be96627fde29961d420d887533d7796`；`git show 7010e0447` 查看是否是你需要的内容；找到自己的内容然后`git merge 7010e0447`；然后还原了git stash drop, git stash clear 的内容

### 版本回退
* 假如做了三次提交回退到上一个版本`git reset --hard HEAD^` 比如回退到前100个版本 `git reset --hard HEAD~100` 即可
* 这时候`git log` 已经回退到第二次提交，如果我们发现又想回到第三次提交
* 通过`git reflog` 查看所有的操作记录（包括已被删除的commit记录和reset的操作）
* 再次利用`git reset –-hard commitId` 回到希望的版本

### 推送本地分支到远程不同名分支
* `git checkout –b feature origin/feature/content_20190829_5481`
* 修改push 的时候，会提示错误，也可以 `git push origin HEAD:feature/content_20190829_5481`
* git默认设置push.default 是simple 模式，要求两边分支是一样的，而upstream 模式则不会有这个要求，所以可以设置：`git config --global push.default upstream`