---
title: Git学习指南
date: 2026-10-21 20:09:05
tags:
- Git
category:
- 版本控制
index_img: /imgs/Git.png
excerpt: 摘要：Git是一种分布式版本控制系统，用于记录文件内容变化，便于查阅特定版本。它由 Linus Torvalds 于 2005 年创建，最初为管理 Linux 内核开发。Git 支持初始化仓库、追踪文件、提交更新、克隆远程仓库等基础操作，并提供分支管理、合并、变基、打标签等功能，帮助开发者高效协作与维护代码历史。
---

## Git

> 版本控制是一种记录一个或若干文件内容变化，以便将来查阅特定版本修订情况的系统。

### Git历史

Git 诞生于一个极富纷争大举创新的年代，Linux 内核开源项目有着为数众多的参与者。 绝大多数的 Linux 内核维护工作都花在了提交补丁和保存归档的繁琐事务上（1991－2002年间）。 到 2002 年，整个项目组开始启用一个专有的分布式版本控制系统 BitKeeper 来管理和维护代码。到了 2005 年，开发 BitKeeper 的商业公司同 Linux 内核开源社区的合作关系结束，他们收回了 Linux 内核社区免费使用 BitKeeper 的权力。 这就迫使 Linux 开源社区（特别是 Linux 的缔造者 Linus Torvalds）基于使用 BitKeeper 时的经验教训，开发出自己的版本系统 Git。

![image](/imgs/Git/2106185-20260930152108917-1984883504.png)


### 初次运行Git前的配置

#### 配置用户信息

```shell
git config --global user.name "John Doe"
git config --global user.email "johndoe@example.com"
```

#### 查看配置信息

```shell
 git config --list
```

### Git基础

#### 初始化仓库

```shell
git init
```

#### 追踪文件

```shell
git add .
```

#### 提交更新

```shell
git commit -m "initial"
```

#### 克隆仓库

```shell
git clone <url>
```

![image](/imgs/Git/2106185-20260930152143514-351515968.png)


#### 查看文件状态

```shell
git status
```

##### 状态简览

```shell
git status -s
```

- 未跟踪的文件：`??`
- 暂存区文件：`A`
- 修改过的文件：`M`

> [!caution]
>
> 输出有两栏，左栏表明暂存区状态，右栏表示的是工作区的状态。

#### 忽略文件

> 一般我们总会有些文件无需纳入 Git 的管理，也不希望它们总出现在未跟踪文件列表。 通常都是些自动生成的文件，比如日志文件，或者编译过程中创建的临时文件等。 在这种情况下，我们可以创建一个名为 .gitignore的文件，列出要忽略的文件的模式。

文件 `.gitignore` 的格式规范如下：

• 所有空行或者以 # 开头的行都会被 Git 忽略。

• 可以使用标准的 glob 模式匹配，它会递归地应用在整个工作区中。

• 匹配模式可以以（/）开头防止递归。

• 匹配模式可以以（/）结尾指定目录。

• 要忽略指定模式以外的文件或目录，可以在模式前加上叹号（!）取反。

#### 查看修改的部分

```sh
git diff
```

##### 查看已暂存的要添加到下次提交的内容

```sh
git diff --staged
```

#### 跳过使用暂存区域直接提交

```sh
 git commit -a -m 'added new benchmarks'
```

#### 从暂存区移除文件但是不想让Git继续跟踪

```shell
git rm --cached 文件名
```

#### 查看提交历史

```shell
git log
```

#### 取消暂存文件

```shell
git reset HEAD <file>
```

#### 撤销对文件的修改

> 改命令会使得对文件的任何修改都会消失——Git会用最近提交的版本覆盖它。**危险！！！**

```shell
git checkout -- <file>
```

### 远程仓库

#### 查看远程仓库

```shell
git remote -v
```

#### 添加远程仓库

```shell
git remote add <shortname> <url>
```

#### 从远程仓库抓取与拉取

1. 访问远程仓库数据拉取还没有的数据

   > 只会将数据下载到本地仓库，不会自动合并或修改当前工作

```shell
git fetch <remote>
```

2. 自动抓取后并合并远程分支到当前分支

```shell
git pull
```

#### 推送到远程仓库

```shell
git push <remote> <branch>
```

#### 查看远程仓库

```shell
git remote show <remote>
```

#### 远程仓库的重命名与移除

```shell
# 重命名
git remote rename <oldname> <newname>
# 移除
git remote rm <shortname>
```

### 打标签

> Git可以给仓库历史中的某一个提交打上标签，以示重要。

#### 列出标签

```shell
git tag -l "v1.0"
```

#### 创建标签

##### 备注标签

```shell
git tag -a v1.0 -m "version 1.0"
# 查看标签信息
git show v1.0
```

##### 轻量标签

```shell
git tag v1.0-lw
```

##### 后期打标签

```shell
git tag -a v1.0 部分校验和
```

#### 推送标签

```shell
# 推送所有标签
git push origin --tags
# 推送指定标签
git push origin v1.0
```

#### 删除标签

1. 方式1

```shell
# 该命令不会从任何远程仓库中移除标签
git tag -d <tagname>
# 更新仓库
git push <remote> :refs/tags/<tagname>
```

2. 方式2

```shell
git push origin --delete <tagname>
```

#### 检出标签

```shell
git checkout -b version2 v1.0
```

### Git别名

```shell
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
```

### Git分支

#### 查看所有分支

```shell
git branch
```

#### 分支创建

```shell
git branch <分支名>
```

#### 分支切换

```shell
git checkout <分支名>
```

### 分支的新建与合并

看一个简单的分支新建与分支合并的例子，实际工作中你可能会用到类似的工作流。 你将经历如下步骤：

1. 开发某个网站。
2. 为实现某个新的用户需求，创建一个分支。
3. 在这个分支上开展工作。

正在此时，你突然接到一个电话说有个很严重的问题需要紧急修补。 你将按照如下方式来处理：

1. 切换到你的线上分支（production branch）。
2. 为这个紧急任务新建一个分支，并在其中修复它。
3. 在测试通过之后，切换回线上分支，然后合并这个修补分支，最后将改动推送到线上分支。
4. 切换回你最初工作的分支上，继续工作。

#### 新建分支

![image](/imgs/Git/2106185-20260930152207777-1966223671.png)


##### 新建分支并同时切换到新分支

```shell
git checkout -b iss53
```

![image](/imgs/Git/2106185-20260930152221790-22712818.png)


假设你已经把你的修改全部提交了，这时你可以切换回 master分支：

```shell
git checkout master
```

假如又出现一个紧急问题，你要修复这个紧急问题。 我们来建立一个 hotfix 分支，在该分支上工作直到问题解决：

```shell
git checkout -b hotfix
```

![image](/imgs/Git/2106185-20260930152238410-1949753025.png)


然后将 hotfix 分支合并回你的 master 分支来部署到线上：

```shell
git checkout master
git merge hotfix
```

![image](/imgs/Git/2106185-20260930152251981-1126425027.png)


关于这个紧急问题的解决方案发布之后，你准备回到被打断之前时的工作中。 然而，你应该先删除 hotfix 分支，因为你已经不再需要它了 —— master 分支已经指向了同一个位置。 你可以使用带 -d 选项的 git branch 命令来删除分支：

```shell
 git branch -d hotfix
 git checkout iss53
```

![image](/imgs/Git/2106185-20260930152313222-808623057.png)

假设你已经修正了 #53 问题，并且打算将你的工作合并入 master 分支。

```shell
 git checkout master
 git merge iss53
 git branch -d iss53
```

![image](/imgs/Git/2106185-20260930152328267-1062265529.png)

![image](/imgs/Git/2106185-20260930152344269-1594861819.png)

#### 遇到冲突时的合并

Git 会在有冲突的文件中加入标准的冲突解决标记，这样你可以打开这些包含冲突的文件然后手动解决冲突。 出现冲突的文件会包含一些特殊区段，看起来像下面这个样子：

```shell
<<<<<<< HEAD:index.html
<div id="footer">contact : email.support@github.com</div>
=======
<div id="footer">
 please contact us at support@github.com
</div>
>>>>>>> iss53:index.html
```

这表示 HEAD 所指示的版本（也就是你的 master 分支所在的位置，因为你在运行 merge 命令的时候已经检出到了这个分支）在这个区段的上半部分（ `======` 的上半部分），而 iss53 分支所指示的版本在 `======` 的下半部分。 为了解决冲突，你必须选择使用由 `======` 分割的两部分中的一个，或者你也可以自行合并这些内容。 例如，你可以通过把这段内容换成下面的样子来解决冲突：

```shell
<div id="footer">
please contact us at email.support@github.com
</div>
```

上述的冲突解决方案仅保留了其中一个分支的修改，并且 `<<<<<<<`, `=======`, 和 `>>>>>>>`这些行被完全删除了。

如果你想使用图形化工具来解决冲突，你可以运行 git mergetool，该命令会为你启动一个合适的可视化合并工具。

### 分支开发工作流

#### 长期分支

因为 Git 使用简单的三方合并，所以就算在一段较长的时间内，反复把一个分支合并入另一个分支，也不是什么难事。 也就是说，在整个项目开发周期的不同阶段，你可以同时拥有多个开放的分支；你可以定期地把某些主题分支合并入其他分支中。

许多使用 Git 的开发者都喜欢使用这种方式来工作，比如只在 master 分支上保留完全稳定的代码——有可能仅仅是已经发布或即将发布的代码。 他们还有一些名为 develop 或者 next 的平行分支，被用来做后续开发或者测试稳定性——这些分支不必保持绝对稳定，但是一旦达到稳定状态，它们就可以被合并入 master 分支了。 这样，在确保这些已完成的主题分支（短期分支，比如之前的 iss53 分支）能够通过所有测试，并且不会引入更多bug 之后，就可以合并入主干分支中，等待下一次的发布。

![image](/imgs/Git/2106185-20260930152413685-2109239885.png)


#### 主题分支

主题分支对任何规模的项目都适用。 主题分支是一种短期分支，它被用来实现单一特性或其相关工作。

考虑这样一个例子，你在 master 分支上工作到 C1，这时为了解决一个问题而新建 iss91 分支，在 iss91 分支上工作到 C4，然而对于那个问题你又有了新的想法，于是你再新建一个 iss91v2 分支试图用另一种方法解决那个问题，接着你回到 master 分支工作了一会儿，你又冒出了一个不太确定的想法，你便在 C10 的时候新建一个 dumbidea 分支，并在上面做些实验。 你的提交历史看起来像下面这个样子：

![image](/imgs/Git/2106185-20260930152430089-1358530792.png)


### 变基

在 `Git` 中整合来自不同分支的修改主要有两种方法：`merge` 以及 `rebase`。

#### 变基基本操作

一个开发任务分叉到两个不同分支，又各自提交了更新。

![image](/imgs/Git/2106185-20260930152447588-1011025774.png)


还有一种方法：你可以提取在 C4 中引入的补丁和修改，然后在 C3 的基础上应用一次。 在 Git 中，这种操作就叫做`变基（rebase）`。即：

```shell
git checkout experiment
git rebase master
```

它的原理是首先找到这两个分支（即当前分支 experiment、变基操作的目标基底分支 master） 的最近共同祖先 C2，然后对比当前分支相对于该祖先的历次提交，提取相应的修改并存为临时文件， 然后将当前分支指向目标基底 C3, 最后以此将之前另存为临时文件的修改依序应用。

![image](/imgs/Git/2106185-20260930152026584-225490404.png)

现在回到 master 分支，进行一次快进合并:

```shell
git checkout master
git merge experiment
```

![image](/imgs/Git/2106185-20260930152043205-1799550800.png)

变基会使得提交历史更加整洁。 你在查看一个经过变基的分支的历史记录时会发现，尽管实际的开发工作是并行的， 但它们看上去就像是串行的一样，提交历史是一条直线没有分叉。
