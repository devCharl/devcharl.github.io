# Git 笔记总结


## 一、git 客户端（本地仓库）的一些操作

### 1. 设置账户（需要和 GitHub 账户设置一致）

```bash
git config --global user.name xxx
git config --global user.email xxx@foxmail.com
```

### 2. 查看设置

```bash
git config --list
```

输出示例：

```
user.name=xxx
user.email=xxx@foxmail.com
```

### 3. 创建 git 本地仓库

```bash
git init
```

此时会出现提示：`Initialized empty Git repository in d://com/liu/.git`

### 4. 查看 git 状态

```bash
git status
```

一般来说会显示需要提交的文件（uncommitted）和未追踪的文件（untracked）：

- **uncommitted**：已有的，刚被修改尚未提交的
- **untracked**：原先没有的，新建的

### 5. 添加 git 文件到暂存区（需要和版本库区分）

```bash
git add <name>
```

### 6. git 提交文件

```bash
git commit -m "add a function in test.java"
```

`-m` 表示注释，为提交时的说明，必须要有！

> `git commit --amend` 可以继续修改本次 commit message

### 7. git 删除文件（夹）

```bash
git rm test.txt        # 删除文件
git rm -r filebook     # 删除文件夹
```

`git rm` 和直接删除的区别在于：`git rm` 会将此文件的操作记录删除，而直接删除仅仅是删除了物理文件，没有删除和此文件相关的记录。`git rm` 后会在版本库产生区别（有操作日志），而直接删除没有。

可以用下面两种操作在版本库中删除文件：

```bash
git rm test.txt
git commit -m 'delete a file'

rm test.txt
git commit -am 'delete a file'
```

注意：命令 `git rm` 用于删除一个文件。如果一个文件已经被提交到版本库，那么你永远不用担心误删，但是要小心，你只能恢复文件到最新版本，你会丢失最近一次提交后你修改的内容。

### 8. git 操作日志

```bash
git log --decorate --graph --oneline --all   # 显示当前及之前的版本号
git log --pretty=oneline                     # 将版本历史显示为一行，历史版本号全部显示
git log --pretty=oneline --abbrev-commit     # 将版本历史显示为一行，历史版本号部分显示
git log --graph                              # 查看分支合并图
```

### 9. 版本回退

执行版本退回后，本地工作区的内容会自动和回退到的版本库版本的内容保持同步：

```bash
git reset --hard HEAD^              # 回退到上一个版本
git reset --hard HEAD^^             # 回退到上上个版本，以此类推，一次提交即为一个版本
git reset --hard e9efa77            # 回退到 e9efa77 版本
```

### 10. git 还原操作

丢弃工作区的操作，但不会丢失暂存区的操作（`add` 操作能将更改添加到暂存区），实际上就是用版本库里的版本替换工作区的版本，无论工作区是修改还是删除，都可以「一键还原」：

```bash
git checkout -- readme.txt
```

### 11. git 暂存区撤销操作

工作区修改了文件，而且执行了 `add`，但还没执行 `commit`，暂存区还是可以撤销的：

```bash
git reset HEAD readme.txt
```

备注：`git reset` 命令既可以回退版本，也可以把暂存区的修改回退到工作区。当我们用 `HEAD` 时，表示最新的版本。

## 二、与 GitHub/Gitee 协同使用（git 代码托管服务器）

和 GitHub 相比，码云（Gitee）也提供免费的 Git 仓库。此外，还集成了代码质量检测、项目演示等功能。对于团队协作开发，码云还提供了项目管理、代码托管、文档管理的服务，5 人以下小团队免费。

### 1. 配置远程仓库免密登录

（1）在用户主目录下，看看有没有 `.ssh` 目录，如果有，再看看这个目录下有没有 `id_rsa` 和 `id_rsa.pub` 这两个文件，如果已经有了，可直接跳到下一步。如果没有，打开 Shell（Windows 下打开 Git Bash），创建 SSH Key：

```bash
ssh-keygen -t rsa -C "xxx@foxmail.com"
```

备注：一路回车，执行生成 `id_rsa` 私钥和 `id_rsa.pub` 公钥，Windows 用户在 git bash 中输入上述指令。

（2）获得 key 的内容，复制下来，添加到 GitHub 的 SSH key 中：

- Windows 位置：`C:\Users\用户名\.ssh\id_rsa.pub`
- Linux 位置：`cat ~/.ssh/id_rsa.pub`

（3）验证 key，根据提示输入 yes，添加为信任主机：

```bash
ssh -T git@github.com
# 或者
ssh -T git@git.oschina.net
```

### 2. 添加远程仓库

```bash
git remote add origin https://github.com/xxx/LearnGit.git   # https 方式
git remote add origin git@github.com:xxx/LearnGit.git        # ssh 方式
```

此处可以为 https 地址也可以是 ssh 地址，`origin` 为设置的远程仓库的别名，强烈建议使用 ssh 方式，因为 https 方式每次都要输入用户名和密码。

如果需要修改传输协议：

```bash
git remote rm <远程主机名>                              # 删除远程仓库
git remote add <远程主机名> <仓库地址>                  # 设置传输方式和目标远程仓库
git push -u <远程主机名> <本地分支名>
```

#### 码云的添加远程仓库方法

```bash
git remote add origin git@gitee.com:xxx/LearnGit.git
```

如果 `git remote add` 失败，并报错：`fatal: remote origin already exists.`

说明本地库已经关联了一个名叫 `origin` 的远程库，此时，可以先用 `git remote -v` 查看远程库信息：

```bash
origin  git@github.com:xxx/LearnGit.git (fetch)
origin  git@github.com:xxx/LearnGit.git (push)
```

表示本地库已经关联了 GitHub 上的 `origin` 远程库，需要先删除已有的 GitHub 库：

```bash
git remote remove origin
```

再关联码云的远程库（注意路径中需要填写正确的用户名）：

```bash
git remote add gitee git@gitee.com:xxx/LearnGit.git
```

因为 git 本身是分布式版本控制系统，可以同步到另外一个远程库，当然也可以同步到另外两个远程库，所以一个本地库可以既关联 GitHub，又关联码云！

使用多个远程库时，要注意 git 给远程库起的默认名称是 `origin`，如果有多个远程库，我们需要用不同的名称来标识不同的远程库。仍然以 learngit 本地库为例，先删除已关联的名为 `origin` 的远程库：

```bash
git remote rm origin
git remote add github git@github.com:xxx/LearnGit.git
git remote add gitee git@gitee.com:xxx/LearnGit.git
```

注意，远程库的名称分别叫 `github` 和 `gitee`，不叫 `origin` 了。

现在，我们用 `git remote -v` 查看远程库信息，可以看到两个远程库：

```bash
gitee   git@gitee.com:xxx/LearnGit.git (fetch)
gitee   git@gitee.com:xxx/LearnGit.git (push)
github  git@github.com:xxx/LearnGit.git (fetch)
github  git@github.com:xxx/LearnGit.git (push)
```

如果要推送到 GitHub，使用命令：

```bash
git push github master
```

如果要推送到码云，使用命令：

```bash
git push gitee master
```

这样一来，本地库就可以同时与多个远程库互相同步。

### 3. 查看远程仓库及传输协议

```bash
git remote
git remote -v    # 查看名称和详细地址
```

### 4. 删除远程仓库

```bash
git remote remove <远程主机名>
```

### 5. 推送本地分支到远程仓库

```bash
git push <远程主机名> <本地分支名>:<远程分支名>
```

如果省略远程分支名，则表示将本地分支推送与之存在「追踪关系」的远程分支（通常两者同名），如果该远程分支不存在，则会被新建。

```bash
git push origin <本地分支名>
git push origin master
```

如果当前分支与多个主机存在追踪关系，则可以使用 `-u` 选项指定一个默认主机，这样以后就可以不加任何参数使用 `git push`：

```bash
git push -u <远程主机名> <本地分支名>
# 例如：
git push -u origin master
```

### 6. 将远程仓库克隆为本地仓库

```bash
git clone git@github.com:xxx/LearnGit.git
```

注意：

1. 不能使用别名
2. 默认情况下，从远程 clone 到本地的库只能看到 `master` 分支，如果要将远程的分支同步到本地：

```bash
git checkout -b <本地分支名> <远程主机名>/<远程分支名>
```

前提是远程 `<远程主机名>` 必须存在名为 `<远程分支名>` 的分支，而且 `<本地分支名>` 和 `<远程分支名>` 最好一致。

### 7. 本地仓库更新

将远程存储库中的更改合并到当前分支中。在默认模式下，`git pull` 是 `git fetch` 后跟 `git merge FETCH_HEAD` 的缩写。更准确地说，`git pull` 使用给定的参数运行 `git fetch`，并调用 `git merge` 将检索到的分支头合并到当前分支中。使用 `--rebase`，它运行 `git rebase` 而不是 `git merge`。

以下是一些示例：

```bash
git pull <远程主机名> <远程分支名>:<本地分支名>
```

比如，要取回 `origin` 主机的 `next` 分支，与本地的 `master` 分支合并，需要写成：

```bash
git pull origin next:master
```

如果远程分支（`next`）要与当前分支合并，则冒号后面的部分可以省略：

```bash
git pull origin next
```

上面命令表示，取回 `origin/next` 分支，再与当前分支合并。实质上，这等同于先做 `git fetch`，再执行 `git merge`：

```bash
git fetch origin
git merge origin/next
```

在某些场合，Git 会自动在本地分支与远程分支之间，建立一种追踪关系（tracking）。比如，在 `git clone` 的时候，所有本地分支默认与远程主机的同名分支，建立追踪关系，也就是说，本地的 `master` 分支自动「追踪」`origin/master` 分支。Git 也允许手动建立追踪关系：

```bash
git branch --set-upstream-to=<远程主机名>/<远程分支名> <本地分支名>
```

比如 `git branch --set-upstream-to=origin/next master`，指定 `master` 分支追踪 `origin/next` 分支。

```bash
git pull origin    # 本地当前分支自动与对应的 origin 主机「追踪分支」进行合并
git pull           # 当前分支自动与唯一一个追踪分支进行合并
git pull --rebase <远程主机名> <远程分支名>:<本地分支名>
```

如果合并需要采用 rebase 模式，可以使用 `--rebase` 选项。

#### git fetch 和 git pull 的区别

（1）`git fetch`：相当于是从远程获取最新版本到本地，不会自动合并。

```bash
git fetch origin master
git log -p master..origin/master
git merge origin/master
```

以上命令的含义：

1. 首先从远程的 `origin` 的 `master` 主分支下载最新的版本到 `origin/master` 分支上
2. 然后比较本地的 `master` 分支和 `origin/master` 分支的差别
3. 最后进行合并

上述过程其实可以用以下更清晰的方式来进行：

```bash
git fetch origin master:tmp
git diff tmp
git merge tmp
```

（2）`git pull`：相当于是从远程获取最新版本并 merge 到本地。

```bash
git pull origin master
```

上述命令其实相当于 `git fetch` 和 `git merge`。在实际使用中，`git fetch` 更安全一些，因为在 merge 前，可以查看更新情况，然后再决定是否合并。

### 8. 查看分支

```bash
git branch
```

### 9. 创建分支

```bash
git branch <name>
```

### 10. 创建并切换到分支

```bash
git checkout -b <name>
```

备注：`git checkout` 命令加上 `-b` 参数表示创建并切换，相当于以下两条命令：

```bash
git branch <name>
git checkout <name>
```

### 11. 切换分支

```bash
git checkout <name>
```

切换分支后，在 git bash 中显示为绿色。

### 12. 删除分支

```bash
git branch -d <name>
```

如果分支没有合并，删除分支就表示会丢失修改，此时 git 无法使用 `-d` 删除，可使用 `-D` 强行删除：

```bash
git branch -D <name>
```

### 13. 合并分支

git 合并默认使用 Fast forward 模式，一旦删除分支，会丢掉分支信息，也就看不出来曾经做过合并：

```bash
git merge <name>
```

基于当前分支，合并另外一个分支，前提需要保证分支之间不冲突。

如果强制禁用 Fast forward 模式，即普通模式，Git 就会在 merge 时生成一个新的 commit：

```bash
git merge --no-ff -m "there is a comment" <name>
```

因为本次合并要创建一个新的 commit，所以加上 `-m` 参数，把 commit 描述写进去。工作中，肯定需要不管有没有分支被删除，都要从分支历史上查看所有的历史分支信息，所以要使用普通模式合并。

### 14. 创建 tag

（1）默认在 HEAD 版本：

```bash
git tag <tagname>
```

（2）对指定的 commit 版本创建 tag，需要先找到历史 commit 的 id：

```bash
git log --pretty=oneline --abbrev-commit
git tag <tagname> <commitid>
```

（3）创建带有说明的 tag，用 `-a` 指定标签名，`-m` 指定说明文字：

```bash
git tag -a <tagname> -m "there is a tag description" [<commitid>]
```

（4）通过 `-s` 用私钥签名一个标签，签名采用 PGP 签名：

```bash
git tag -s <tagname> -m "there is a tag description" [<commitid>]
```

必须首先安装 gpg（GnuPG），如果没有找到 gpg，或者没有 gpg 密钥对，就会报错，参考 GnuPG 帮助文档配置 Key。

### 15. 查看 tag

```bash
git tag    # 显示的 tag 不是按时间顺序排列，而是按字母顺序排列
```

如果想查看 tag 和 commit 的对应关系：

```bash
git log --pretty=oneline --abbrev-commit
```

如果想查看 tag 的详细情况：

```bash
git show <tagname>
```

### 16. 删除 tag

创建的标签都只存储在本地，不会自动推送到远程。所以，打错的标签可以在本地安全删除：

```bash
git tag -d <tagname>
```

如果标签已经推送到远程，要删除远程标签就麻烦一点：

```bash
git tag -d <tagname>
git push origin :refs/tags/<tagname>
```

### 17. 推送标签至远程

```bash
git push origin <tagname>
```

或者，一次性推送全部尚未推送到远程的本地标签：

```bash
git push origin --tags
```

### 18. 现场的保存与恢复

```bash
git stash        # 将目前的工作现场保存
git stash list   # 查看所有保存的工作现场
```

工作现场还在，Git 把 stash 内容存在某个地方了，但是需要恢复一下，有两个办法：

1. 用 `git stash apply stash@{0}` 恢复，但是恢复后，stash 内容并不删除，你需要用 `git stash drop stash@{0}` 来删除
2. 用 `git stash pop`，恢复的同时把 stash 内容也删了，这种方式省时省力

注意点：

1. 如果在分支下新建文件，而尚未执行 `add` 操作，stash 无法将新文件纳入保存的现场，因为 stash 只对被修改的被追踪的文件和暂存的变更有效，对于新文件必须先执行 `add`
2. 如果修改分支下的已被追踪的文件，不管有没有对修改的文件进行 `add` 操作，如果执行 stash，所有修改会被纳入保存的现场，而文件会恢复成修改前的状态。恢复现场后，文件又呈现被修改后的状态。特别的是，如果修改的文件在 stash 前已经被 `add` 了，恢复现场后，暂存区的内容就会清空，相当于这个文件从未被 `add` 一样

### 19. 设置 Git UI 颜色

让 Git 显示颜色，会让命令输出看起来更醒目：

```bash
git config --global color.ui true
```

### 20. 忽略特殊文件

（1）在 Git 工作区的根目录下创建一个特殊的 `.gitignore` 文件，然后把要忽略的文件名填进去，Git 就会自动忽略这些文件。不需要从头写 `.gitignore` 文件，GitHub 已经为我们准备了各种配置文件，只需要组合一下就可以使用了。所有配置文件可以直接在线浏览：<https://github.com/github/gitignore>

忽略文件的原则是：

- 忽略操作系统自动生成的文件，比如缩略图等
- 忽略编译生成的中间文件、可执行文件等，也就是如果一个文件是通过另一个文件自动生成的，那自动生成的文件就没必要放进版本库，比如 Java 编译产生的 `.class` 文件
- 忽略你自己的带有敏感信息的配置文件，比如存放口令的配置文件

比如一个完整的 `.gitignore` 文件，内容如下：

```gitignore
# Windows
Thumbs.db
ehthumbs.db
Desktop.ini

# Python
*.py[cod]
*.so
*.egg
*.egg-info
dist
build
```

（2）把 `.gitignore` 也提交到 Git：

```bash
git add .gitignore
git commit -m "there is a description"
```

就完成了！当然检验 `.gitignore` 的标准是 `git status` 命令是不是显示 `working tree clean`。

使用 Windows 的注意：如果在资源管理器里新建一个 `.gitignore` 文件，系统会非常弱智地提示必须输入文件名，但是在文本编辑器里「保存」或者「另存为」就可以把文件保存为 `.gitignore` 了。

（3）如果确实想要添加已经被 `.gitignore` 忽略的文件，可以用 `-f` 强制添加到 Git：

```bash
git add -f test.class
```

（4）怀疑 `.gitignore` 写的有问题，需要查找哪个规则写错了，可以用 `git check-ignore` 命令检查：

```bash
git check-ignore -v App.class
# .gitignore:3:*.class    App.class
```

表示 `.gitignore` 的第 3 行规则忽略了 `App.class` 这个文件，于是我们就可以知道应该修订哪个规则。

### 21. 为命令配置别名

（1）命令可以简写，用 `git st` 表示 `git status`，再比如用 `co` 表示 `checkout`、`ci` 表示 `commit`、`br` 表示 `branch`：

```bash
git config --global alias.co checkout
git config --global alias.ci commit
git config --global alias.br branch
```

以后提交就可以简写成：

```bash
git ci -m "there is a description"
```

`--global` 参数是全局参数，也就是这些命令在这台电脑的所有 Git 仓库下都有用。

（2）命令 `git reset HEAD <filename>` 可以撤销暂存区的修改（unstage），重新放回工作区。既然是一个 unstage 操作，就可以配置一个 unstage 别名：

```bash
git config --global alias.unstage 'reset HEAD'
git unstage test.py    # 等价于 git reset HEAD test.py
```

（3）配置一个 `git last`，让其显示最后一次提交信息：

```bash
git config --global alias.last 'log -1'
```

这样，用 `git last` 就能显示最近一次的提交：

```
Author: xxx <xxx@foxmail.com>
Date:   Mon Apr 23 13:52:44 2018 +0800

    add git ignore list
```

（4）还可以把 `lg` 配置成：

```bash
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```

来看看 `git lg` 的效果。

### 22. 修改配置文件

配置 Git 的时候，加上 `--global` 是针对当前用户起作用的，如果不加，那只针对当前的仓库起作用。

每个仓库的 Git 配置文件都放在 `.git/config` 文件中：

```bash
cat .git/config
```

```ini
[core]
    repositoryformatversion = 0
    filemode = false
    bare = false
    logallrefupdates = true
    symlinks = false
    ignorecase = true
[branch "master"]
[branch "dev"]
[remote "github"]
    url = git@github.com:xxx/LearnGit.git
    fetch = +refs/heads/*:refs/remotes/github/*
[remote "gitee"]
    url = git@gitee.com:xxx/LearnGit.git
    fetch = +refs/heads/*:refs/remotes/gitee/*
```

而当前用户的 Git 配置文件放在用户主目录下的一个隐藏文件 `.gitconfig` 中：

```ini
[user]
    name = xxx
    email = xxx@foxmail.com
[gpg]
    program = C:\\Program Files (x86)\\gnupg\\bin\\gpg.exe
[color]
    ui = true
[alias]
    co = checkout
    ci = commit
    br = branch
    last = log -1
    lg = log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit
```

别名就在 `[alias]` 后面，要删除别名，直接把对应的行删掉即可。配置别名也可以直接修改这个文件，如果改错了，可以删掉文件重新通过命令配置。

## 三、多人协作的工作模式

通常如下：

1. 首先将远程仓库克隆为本地仓库：

```bash
git clone git@github.com:xxx/LearnGit.git
```

2. 在本地创建和远程分支对应的分支：

```bash
git checkout -b <本地分支名> origin/<远程分支名>
```

本地和远程分支的名称最好一致。

3. 在本地分支完成任务后，可以试图用 `git push <远程主机名> <本地分支名>` 推送自己的修改
4. 如果推送失败，则表明远程分支比本地更新，需要先用 `git pull` 试图合并
5. 如果 pull 失败并提示「no tracking information」，则说明本地分支和远程分支的链接关系没有创建，用命令 `git branch --set-upstream-to=<远程主机名>/<远程分支名> <本地分支名>` 创建链接
6. 如果合并有冲突，则解决冲突，并在本地提交（`add` => `commit`）
7. 没有冲突或者解决掉冲突后，再用 `git push <远程主机名> <本地分支名>` 推送就能成功


---

> 作者: [Charles Y](https://github.com/devCharl/devcharl.github.io)  
> URL: https://devcharl.github.io/develop/git-notes-summary/  

