# 快速入门

[git - 简明指南](https://www.runoob.com/manual/git-guide/)

### 创建远程仓库

① 在GitHub创建一个仓库，复制 SSH 地址

```
git@github.com:STAR0903/biz-reviews-apis.git
```

② 配置本地项目并推送到远程

```
# 进入你的项目目录
cd biz-reviews-apis

# 初始化 Git 仓库（如果尚未初始化）
git init

# 添加所有文件并提交
git add .
git commit -m "Initial commit"

# 关联远程仓库（使用上面复制的 SSH 地址）
git remote add origin git@github.com:STAR0903/biz-reviews-apis.git

# 将当前分支重命名为 main（GitHub 默认主分支为 main）
git branch -M main

# 首次推送并设置上游跟踪
git push -u origin main
```

### 提交

```
# 提出更改（把它们添加到暂存区）
# 所有
git add .
# 某个文件/文件夹
git add 文件位置
# 实际提交改动
git commit -m "代码提交信息"
# 提交到远端仓库
git push origin master
```

### 撤销

##### 撤销一次 Git commit

① 本地 commit 尚未推送到远程（推荐保留代码）

```bash
# 撤销最近一次 commit，但保留修改内容（文件仍处于暂存状态）
git reset --soft HEAD~1
```

② 本地 commit 尚未推送，且想彻底丢弃更改

```bash
# 撤销 commit 并永久删除所有相关修改（不可恢复！）
git reset --hard HEAD~1
```

③ commit 已经 push 到远程仓库（团队协作场景）

```bash
# 创建一个新的“反向 commit”来撤销更改（安全、不改写历史）
git revert HEAD

# 推送撤销操作到远程
git push
```

### 子模块

##### 添加一个子模块到当前项目

```bash
# 将远程仓库作为子模块添加到本地指定目录（如 vendor/tool）
git submodule add git@github.com:xxx/shared-utils.git vendor/shared-utils
```

执行后会：

- 在 `vendor/shared-utils/` 创建子仓库目录
- 自动生成 `.gitmodules` 配置文件
- 主仓库记录子模块的 commit ID（快照）

##### 更新子模块

更新项目内子模块到最新版本

```bash
git submodule update
```

更新子模块为远程项目的最新版本

```bash
git submodule update --remote
```

# 版本控制

版本控制（Revision control）是一种在开发的过程中用于管理我们对文件、目录或工程等内容的修改历史，方便查看更改历史记录，备份以便恢复以前的版本的软件工程技术。

* 实现跨区域多人协同开发
* 追踪和记载一个或者多个文件的历史记录
* 组织和保护你的源代码和文档
* 统计工作量
* 并行开发、提高开发效率
* 跟踪记录整个软件的开发过程
* 减轻开发人员的负担，节省时间，同时降低人为错误

简单说就是用于管理多人协同开发项目的技术。

没有进行版本控制或者版本控制本身缺乏正确的流程管理，在软件开发过程中将会引入很多问题，如软件代码的一致性、软件内容的冗余、软件过程的事物性、软件开发过程中的并发性、软件源代码的安全性，以及软件的整合等问题。

### 常见工具

主流的版本控制器有如下这些：

* **Git**
* **SVN**（Subversion）
* **CVS**（Concurrent Versions System）
* **VSS**（Micorosoft Visual SourceSafe）
* **TFS**（Team Foundation Server）
* Visual Studio Online

版本控制产品非常的多（Perforce、Rational ClearCase、RCS（GNU Revision Control System）、Serena Dimention、SVK、BitKeeper、Monotone、Bazaar、Mercurial、SourceGear Vault），现在影响力最大且使用最广泛的是Git与SVN。

### 分类

##### 本地版本控制

记录文件每次的更新，可以对每个版本做一个快照，或是记录补丁文件，适合个人用，如RCS。

![图片](https://mmbiz.qpic.cn/mmbiz_png/uJDAUKrGC7Ksu8UlITwMlbX3kMGtZ9p0Dg3fHrbPqbNEOMO9GTjFhVaukMZWx54icS7eS2x8A7BEu0VB9ibwEhzQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**2、集中版本控制  SVN**

所有的版本数据都保存在服务器上，协同开发者从服务器上同步更新或上传自己的修改。

![图片](https://mmbiz.qpic.cn/mmbiz_png/uJDAUKrGC7Ksu8UlITwMlbX3kMGtZ9p00V4uLaibxtZI9RLpq7tkSdlWiaF92AVeZ0ib9DicqBkS2poo5u8sEU2mCQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

所有的版本数据都存在服务器上，用户的本地只有自己以前所同步的版本，如果不连网的话，用户就看不到历史版本，也无法切换版本验证问题，或在不同分支工作。而且，所有数据都保存在单一的服务器上，服务器损坏，就会丢失所有的数据，当然可以通过定期备份解决。代表产品：SVN、CVS、VSS。

**3、分布式版本控制  Git**

安全隐患：每个人都拥有全部的代码。

所有版本信息仓库全部同步到本地的每个用户，这样就可以在本地查看所有版本历史，可以离线在本地提交，只需在连网时push到相应的服务器或其他用户那里。由于每个用户那里保存的都是所有的版本数据，只要有一个用户的设备没有问题就可以恢复所有的数据，但这增加了本地存储空间的占用。

不会因为服务器损坏或者网络问题，造成不能工作的情况！

**Git是目前世界上最先进的分布式版本控制系统。**

![图片](https://mmbiz.qpic.cn/mmbiz_png/uJDAUKrGC7Ksu8UlITwMlbX3kMGtZ9p0ev8Q7qXjsTfeSwFexdA4tGjFAiaVEKQzAHdGcINXILKflI2cfk9BiawQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

##### Git与SVN的主要区别

SVN是集中式版本控制系统，版本库是集中放在中央服务器的，而工作的时候，用的都是自己的电脑，所以首先要从中央服务器得到最新的版本，然后工作，完成工作后，需要把自己做完的活推送到中央服务器。集中式版本控制系统是必须联网才能工作，对网络带宽要求较高。

Git是分布式版本控制系统，没有中央服务器，每个人的电脑就是一个完整的版本库，工作的时候不需要联网了，因为版本都在自己电脑上。协同的方法是这样的：比如说自己在电脑上改了文件A，其他人也在电脑上改了文件A，这时，你们两之间只需把各自的修改推送给对方，就可以互相看到对方的修改了。Git可以直接看到更新了哪些代码和文件！

# 基础理论

### 三个区域

Git本地有三个工作区域：工作目录（Working Directory）、暂存区(Stage/Index)、资源库(Repository或Git Directory)。如果在加上远程的git仓库(Remote Directory)就可以分为四个工作区域。文件在这四个区域之间的转换关系如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/uJDAUKrGC7Ksu8UlITwMlbX3kMGtZ9p0NJ4L9OPI9ia1MmibpvDd6cSddBdvrlbdEtyEOrh4CKnWVibyfCHa3lzXw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

* Workspace：工作区，就是你平时存放项目代码的地方
* Index / Stage：暂存区，用于临时存放你的改动，事实上它只是一个文件，保存即将提交到文件列表信息
* Repository：仓库区（或本地仓库），就是安全存放数据的位置，这里面有你提交到所有版本的数据。其中HEAD指向最新放入仓库的版本
* Remote：远程仓库，托管代码的服务器，可以简单的认为是你项目组中的一台电脑用于远程数据交换

本地的三个区域确切的说应该是git仓库中HEAD指向的版本：

![图片](https://mmbiz.qpic.cn/mmbiz_png/uJDAUKrGC7Ksu8UlITwMlbX3kMGtZ9p0icz6X2aibIgUWzHxtwX8kicPCKpDrsiaPzZk04OlI2bzlydzicBuXTJvLEQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

* Directory：使用Git管理的一个目录，也就是一个仓库，包含我们的工作空间和Git的管理空间。
* WorkSpace：需要通过Git进行版本控制的目录和文件，这些目录和文件组成了工作空间。
* .git：存放Git管理信息的目录，初始化仓库的时候自动创建。
* Index/Stage：暂存区，或者叫待提交更新区，在提交进入repo之前，我们可以把所有的更新放在暂存区。
* Local Repo：本地仓库，一个存放在本地的版本库；HEAD会指向当前的开发分支（branch）。
* Stash：隐藏，是一个工作状态保存栈，用于保存/恢复WorkSpace中的临时状态。

### 文件的四种状态

版本控制就是对文件的版本控制，要对文件进行修改、提交等操作，首先要知道文件当前在什么状态，不然可能会提交了现在还不想提交的文件，或者要提交的文件没提交上。

* Untracked: 未跟踪，此文件在文件夹中，但并没有加入到git库，不参与版本控制。通过 git add 状态变为 Staged。
* Unmodify: 文件已经入库，未修改，即版本库中的文件快照内容与文件夹中完全一致。如果它被修改，变为 Modified。如果使用 git rm 移出版本库，则成为Untracked文件。
* Modified: 文件已修改，仅仅是修改，并没有进行其他的操作。通过 git add可进入暂存 staged 状态，使用 git checkout 则丢弃修改过内容（从库中取出文件, 覆盖当前修改），返回到unmodify状态。
* Staged: 暂存状态。 执行 git commit 则将修改同步到库中，这时库中的文件和本地文件又变为一致，文件为Unmodify状态。执行 git reset HEAD filename 取消暂存，文件状态为Modified。

# [子模块的管理和使用](https://www.jianshu.com/p/9000cd49822c)

当你在一个Git 项目上工作时，你需要在其中使用另外一个Git 项目。也许它是一个第三方开发的Git 库或者是你独立开发和并在多个父项目中使用的。这个情况下一个常见的问题产生了：你想将两个项目单独处理但是又需要在其中一个中使用另外一个。

在Git 中你可以用子模块 `submodule`来管理这些项目，`submodule`允许你将一个Git 仓库当作另外一个Git 仓库的子目录。这允许你克隆另外一个仓库到你的项目中并且保持你的提交相对独立。

# 实践经验

如使用GitHub作为远程仓库，采用 `git clone [url]` 的方式初始化本地仓库时发现报错：

```
git clone https://github.com/[user_name]/[repository_name].git
```

```
fatal: unable to access 'https://github.com/STAR0903/algorithm.git/': OpenSSL SSL_read: SSL_ERROR_SYSCALL, errno 0
```

根据排查，发现HTTPS协议可能被拦截，改用SSH克隆：

```
git clone git@github.com:[user_name]/[repository_name].git
```

# 参考笔记

[狂神公众号](https://mp.weixin.qq.com/s/Bf7uVhGiu47uOELjmC5uXQ)

[Git submodule 子模块的管理和使用 - 简书](https://www.jianshu.com/p/9000cd49822c)

[5分钟搞懂Monorepo_git_俞凡_InfoQ写作社区](https://xie.infoq.cn/article/4f870ba6a7c8e0fd825295c92)

[Gen Transaction | GORM - The fantastic ORM library for Golang, aims to be developer friendly.](https://gorm.io/zh_CN/gen/transaction.html)
