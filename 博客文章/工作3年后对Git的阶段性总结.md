不管是处于哪一级别的代码工程师， 不管是单枪匹马地干，还是跟团队一起协作，都不能绕过的一个工具就是代码版本管理工具（CVS，全称：Concurrent Version System）。以下是百度百科对代码管理工具的解释：

> 版本控制软件提供完备的[版本管理](https://baike.baidu.com/item/%E7%89%88%E6%9C%AC%E7%AE%A1%E7%90%86/2511538)功能，用于[存储](https://baike.baidu.com/item/%E5%AD%98%E5%82%A8/1582924)、[追踪](https://baike.baidu.com/item/%E8%BF%BD%E8%B8%AA/44928)目录（[文件夹](https://baike.baidu.com/item/%E6%96%87%E4%BB%B6%E5%A4%B9/23609397)）和[文件](https://baike.baidu.com/item/%E6%96%87%E4%BB%B6/24602116)的修改历史，是软件开发者的必备工具，是软件公司的基础设施。版本控制软件的最高目标，是支持[软件公司](https://baike.baidu.com/item/%E8%BD%AF%E4%BB%B6%E5%85%AC%E5%8F%B8)的配置管理活动，追踪多个版本的开发和维护活动，及时发布[软件](https://baike.baidu.com/item/%E8%BD%AF%E4%BB%B6/12053)。
> 
> [百度百科：版本控制软件](https://baike.baidu.com/item/%E7%89%88%E6%9C%AC%E6%8E%A7%E5%88%B6%E8%BD%AF%E4%BB%B6)

根据上面的描述可以知道，它跟我们如何写代码无关， 只是帮助我们存储写完的代码，它通常是我们完成工作的最后一个步骤。然而在实际工作中，程序员往往忽略了他的重要性， 这种情况在初级程序员当中尤其常见。其中的一个原因是在现阶段的学校教学中忽略对这一个方面有通识讲解，也缺少对应团队协作开发的场景（通常作业和作品都是需要单独完成上交，不允许抄袭也就没有一起协作的情况）。另外一个重要的原因是，通常开发人员只关心的是他如何完成目前的需求功能，而对于怎么储存他的代码，这似乎不是他们所应该承担的负责，而是由更高级人来负责。但事实上在目前需求瞬息万变的开发环境下，你的每一次提交都会影响着每个迭代的发版，因此系统地了解、学习代码管理工具，在团队中实施统一的版本管理规范，在软件开发过程中尤为重要。

### 常见的版本管理工具

目前常见的版本管理工具可以分为： 

- 集中式版本管理工具，代表为SVN。
- 分布式版本管理工具，代表为Git。

他们（SVN、Git）的区别主要有以下两个方面：

- 存储代码的地方不一样
- 存储代码的方式不一样


##### 存储代码的地方不一样

![[集中式版本管理工具.png]]
集中式版本管理工具（SVN）

![[分布式版本管理工具.png]]
分布式版本管理工具（Git）

由上面的图可以看出：

集中式版本管理工具把代码统一存储在一个服务器上，客户端都只能从这一个服务节点中获取、更新代码。

分布式版本管理工具是客户端机器可以有一个或者多个远程代码仓库，客户端可以分别从不同的远程代码仓库更新和拉取代码。

##### 存储代码的方式不一样
1. 对于文件修改先后的差异，Git是直接记录快照，而SVN则是记录比较后的差异。
	   ![[Pasted image 20220311005926.png]]
SVN记录文件差异方式

![[Pasted image 20220311010051.png]]
Git记录文件差异方式

>每次你提交 更新，或在 Git 中保存项目状态时，它主要对当时的全部文件制作一个快照并保存这个快照的索引。为了高效， 如果文件没有修改，Git 不再重新存储该文件，而是只保留一个链接指向之前存储的文件。Git 对待数据更像是 一个 快照流。

2. SVN的分支是一个目录，每个目录下存放着对应分支状态下的文件；git则是通过SHA-1算法计算出对应的文件内容；
3. git有工作区，暂存区，远程仓库三个概念，add将代码提交到暂存区，commit提交到本地版本库，push推送到远程版本库。svn是add 提交到暂存区，commit是提交到远端。
4. git可以离线工作，SVN则不可以。

##### Git的工作区、暂存区、git仓库
![[Pasted image 20220311011513.png]]

- 工作目录：当前处于修改的空间，也是我们写代码的地方，当我们把有修改代码的文件 add 之后，便会流转到 **暂存区**。
- 暂存区：即将放到仓库（本地）的空间，当我们 commit 后代码则会流转到 **本地仓库**
- 本地仓库：存放我们代码的地方，是跟远端仓库同步的空间

##### Git常用命令

- git add：把有修改的文件添加到 **暂存区**
- git commit：把 暂存区 的代码提交到 **本地仓库**
- git push：把 本地仓库 的分支代码推送到 **远程仓库**
- git fetch：更新 远程仓库 的分支代码信息
- git checkout：切换到对应的分支
- git merge：把另一个分支的代码合并对目前的分支
- git pull：在更新 远程仓库 的分支代码信息同时，马上合并该 远程仓库 的分支代码

![[Pasted image 20220311013321.png]]

其它不是很常用但是也很重要的命令有：
- git stash
- git rebase
- git cherry-pick
- git tags

git命令背后的操作

##### 常见工作流

同事A：
1. 拉取（fetch）**远端分支**代码信息
2. 切换（checkout）到对应分支进行开发
3. 暂存（add）代码
4. 提交（commit）代码到本地仓库
5. 推送（push）代码到远程仓库

#### Git目录

#### 新手不知道的操作

情况一：git fetch 与 git pull

rebase
修改最后一次提交

修改多个提交信息

重新排序提交

压缩提交

修改上一次提交背后的操作：
git commit --amend => git reset --soft HEAD~ && git commit -m "[new comment]"

git reset --hard 的后悔药：
git reflog

情况二：使用 fast-forward 模式 merge 分支代码

在 master 上同时新建了一个 feature A 和一个 hotfix B 分支，在hotfix B 分支上修复了一个问题后提交了一个 commit。

1. 如果让 master 分支 merge hotfix B 分支，此时的 merge 使用的是 fast-forward 模式，在分支图上不会产生多余的 merge commit。
2. feature A 正常开发了一个功能并提交了一个 commit，此时合并 master 则不是 fast- forward 模式，合并后将会产生一个新的 merge commit。


情况三：使用 rebase 变基源分支代码
1. 从 master 分支上新建了一个 feature A 分支，并且进行了 commit，此时 feature A 比 master 领先一个 commit；
2. 同事在 master 上新建了一个 hotfix B 分支修复了一个问题后，使用 fast-forward 合并到 master 上线，此时 master 与 feature A 都合自有一个自己独有的commit。
3. feature A 分支需要用到 hotfix B 修复的功能，此时如果直接 merge master，则会产生一个新的 merge commit。
4. 为了保持分支图整洁，feature A 分支应该使用 rebase 命令来把代码变基到 master 分支上，变基完后 feature A 跟原来一样，比 master 分支领先一个 commit；



