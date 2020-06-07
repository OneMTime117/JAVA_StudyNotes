# Git与GitHub

1、Git是一款免费、开源的分布式版本控制系统，和SVN类似

2、GitHub 提供基于git的版本托管服务的网络平台，因此Git 只是GitHub管理项目的工具，而GitHub提供了多种对于项目代码、等其他文件的社交功能

# GitHub使用

- Repository

仓库的意思，即你的项目，你想在 GitHub 上开源一个项目，那就必须要新建一个 Repository ，如果你开源的项目多了，你就拥有了多个 Repositories 。

- Issue

问题的意思，举个例子，就是你开源了一个项目，别人发现你的项目中有bug，或者哪些地方做的不够好，他就可以给你提个 Issue ，即问题，提的问题多了，也就是 Issues ，然后你看到了这些问题就可以去逐个修复，修复ok了就可以一个个的 Close 掉。

- Star

这个好理解，就是给项目点赞，但是在 GitHub 上的点赞远比微博、知乎点赞难的多，如果你有一个项目获得100个star都算很不容易了！

- Fork

这个不好翻译，如果实在要翻译我把他翻译成分叉，什么意思呢？你开源了一个项目，别人想在你这个项目的基础上做些改进，然后应用到自己的项目中，这个时候他就可以 Fork 你的项目，这个时候他的 GitHub 主页上就多了一个项目，只不过这个项目是基于你的项目基础（本质上是在原有项目的基础上新建了一个分支，分支的概念后面会在讲解Git的时候说到），他就可以随心所欲的去改进，但是丝毫不会影响原有项目的代码与结构。

- Pull Request

发起请求，这个其实是基于 Fork 的，还是上面那个例子，如果别人在你基础上做了改进，后来觉得改进的很不错，应该要把这些改进让更多的人收益，于是就想把自己的改进合并到原有项目里，这个时候他就可以发起一个 Pull Request（简称PR） ，原有项目创建人就可以收到这个请求，这个时候他会仔细review你的代码，并且测试觉得OK了，就会接受你的PR，这个时候你做的改进原有项目就会拥有了。

- Watch

这个也好理解就是观察，如果你 Watch 了某个项目，那么以后只要这个项目有任何更新，你都会第一时间收到关于这个项目的通知提醒。

- Gist

有些时候你没有项目可以开源，只是单纯的想分享一些代码片段，那这个时候 Gist 就派上用场了！

# Git使用

## 1、Git官方命令行控制台（Git Bash）本地仓库操作

**git init：初始化Git仓库，使当前目录转变为Git仓库**

可以通过右键该目录来打开控制台，直接定位到该目录下

也可以通过cd  目录路径定位；但是注意，git官方CMD为unix系统版本，路径分隔符为/,不能使用window系统的\

**git  status:查看当前分支的状态（经常使用，来查看仓库文件代码是否有变动）**



**默认初始化Git仓库后，生成一个主分支（master）**

**git branch: 获取该Git仓库所有分支**

**git branch  yh：创建一个分支，名为yh**

**git checkout yh：切换到yh分支进行操作**

**git checkout -b yh：为git branch  yh、git checkout yh两个命令行的组合，用于创建并切换到该分支**



**git branch -d yh：删除yh分支**

**git branch -D yh：强制删除yh分支，不会由于分支为合并，从而不能删除**

​		不能在该yh分支下，执行该代码（自己不能删除自己）；

**git merge   yh：将yh分支代码合并到当前分支中（一般为主分支master）**



**git add 文件名：当有新文件或产生文件修改时，通过git add来提交新文件或者修改的文件**

​		add提交只是将文件添加到缓存区中，并没有实际修改Git仓库，需要通过 Git commit来进行进一步确定

**git commit -m ‘     ’：确认提交，进行真正提交**

​		必须添加-m ‘ 提交信息’，并且信息不能为‘’（空）

**git  checkout 文件名：取消该文件的修改，并且还原为之前状态，需要在 git   add 之前；当add后，就无法施一公checkout来将文件修改还原**



**git tag v1.0：对于当前分支，添加一个标签，此刻所有代码在该标签下，都不会随分支的变化而变化（设置代码的一个版本，用于整体代码多版本管理）**

**git checkout v1.0：切换到v1.0标签的代码版本中**

## 2、git基于GitHub远程仓库操作

### 1、通过SSH使git本地仓库与GitHub平台建立连接授权

​	SSH：Secure Shell 安全的Shell，用于提供计算机之间加密的Shell操作

​    Shell：用于访问操作远程计算机的系统内核，通过shell程序提供的命令行或者图形化界面，来操作系统

GitBash提供SSH服务，用于和GitHub平台进行SSH通信，操作如下：

1、在GitBash窗口中输入   ssh-keygen -t rsa  :使用shh生成密钥，算法为rsa；此时会在Win系统**/c/Documents and Settings/username/.ssh** 目录下生成id_rsa 和 id_rsa.pub 两个文件

2、在GitHub平台中添加SSH key，将 id_rsa.pub 中的内容粘贴到其中

3、此时可以通过ssh -T git@github.com 来测试，是否能连接github服务器

### 2、Git本地仓库与GitHub远程仓库进行提交（Push），更新（Pull）

1、当第一次pull GitHub远程仓库的代码时（本地还没有该仓库）：

git clone git@github.com:GitHub账号名/仓库名.git    此时会自动将当前文件夹初始化为Git仓库

当本地仓库修改提交后，可以通过      git push origin master  将本地修改同步到远程仓库

2、当需要将自己的本地仓库代码同步到远程仓库时（即远程仓库没有该项目）：

- 在GitHub上新建一个远程仓库

- git remote add origin git@github.com: GitHub账号名/仓库名.git :将本地仓库与远程仓库建立联系
- git push origin master  ：将本地仓库代码同步到远程仓库中

git remote   -v    :查看当前本地仓库所有与之关联的远程仓库名，当只有一个远程仓库时，默认使用别名origin来代替，因此一般本地仓库提交时，都使用origin master

## 3、Git进阶操作

1、alias，使用别名来简化Git命令的使用

git config   -- global  alias.co   checkout   将checkout关键词别名设置为co

git config   -- global  alias.psm 'push origin master'   将push origin master 命令行  简化为别名 psm

PS：关键词不需要使用‘’，而对于组合的命令行需要进行‘ ‘

2、git log   获取仓库日志

3、设置项目修改时显示的用户信息（用户名、邮箱）

git config --global user.name "stormzhang"
git config --global user.email "stormzhang.dev@gmail.com"

global为全局参数，即修改所有项目中的用户信息，可以去掉global，从而使不同项目中的用户信息不一样

PS：如果用户信息中的邮箱地址无法与用户github中的邮箱地址匹配，则无法在远程仓库中显示对于该用户的提交进度。但是项目实际还有被修改了

4、git stash

用于暂存当前未commit（无论是否add）的代码

因为当切换分支时，之前未commit的代码会由于在该分支中发生冲突的改变

此时就需要使用 git stash，来将目前未写完的代码保存到暂存区，而切换分支后，不会丢失

git stash list  获取暂存记录

git stash apply  还原暂存代码

git stash drop  删除最近一条暂存记录

git stash pop  即还原暂存代码，又删除最近一条暂存记录

git stash clear  删除所有暂存记录

5、merge&rebase

git进行分支合并：将两个分支进行合并，有两种方式：

1、git merge 分支名     直接快速暴力合并，可以知道所有提交对于所在的分支，但是无法有效知道这些提交的顺序

2、git rebase 分支名   会先合并同一父级下的子分支节点，因此不会按照节点的提交顺序进行合并，

当合并的分支不需要时，可以直接使用merge；

不能再master分支上，使用rebase，会导致历史日志错乱

在同一个仓库下，切换不同分支，会导致仓库目录文件夹下的文件根据分支的切换而发生改变；

因此可以知道，不同分支是使用了不同的文件；对于修改的文件，可以选择切换分支来提交。

