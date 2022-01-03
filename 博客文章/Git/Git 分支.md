## Git 基本概念与分支

### 版本控制系统

在这之前，其实版本控制系统的类型有：

- 本地版本控制系统

  <img src="/Users/kuntang/Library/Application Support/typora-user-images/image-20191220171311504.png" alt="image-20191220171311504" style="zoom:50%;" />

- 集中化版本控制系统

  <img src="/Users/kuntang/Library/Application Support/typora-user-images/image-20191220171346750.png" alt="image-20191220171346750" style="zoom:50%;" />

- 分布式版本控制系统

  <img src="/Users/kuntang/Library/Application Support/typora-user-images/image-20191220171403574.png" alt="image-20191220171403574" style="zoom:50%;" />

### Git 基础 

#### 直接记录快照，而非差异比较

Git 和其它版本控制系统(包括 Subversion 和近似工具)的主要差别在于 Git 对待数据的方法。概念上来区分， 其它大部分系统以文件变更列表的方式存储信息。这类系统(CVS、Subversion、Perforce、Bazaar 等等)将 它们保存的信息看作是一组基本文件和每个文件随时间逐步累积的差异。 

<img src="/Users/kuntang/Library/Application Support/typora-user-images/image-20191220171715970.png" alt="image-20191220171715970" style="zoom:50%;" />

Git 不按照以上方式对待或保存数据。反之，Git 更像是把数据看作是对小型文件系统的一组快照。每次你提交 更新，或在 Git 中保存项目状态时，它主要对当时的全部文件制作一个快照并保存这个快照的索引。**为了高效， 如果文件没有修改，Git 不再重新存储该文件，而是只保留一个链接指向之前存储的文件。**Git 对待数据更像是 一个 ***快照流***。 

<img src="/Users/kuntang/Library/Application Support/typora-user-images/image-20191220171824579.png" alt="image-20191220171824579" style="zoom:50%;" />

### Git 保证完整性 

Git 中所有数据在存储前都计算 ***校验和*** ，然后以 ***校验和*** 来引用。这意味着不可能在 Git 不知情时更改任何文件内容或目录内容。 

Git 用以计算校验和的机制叫做 SHA-1 散列(hash，哈希)。这是一个由 40 个十六进制字符(0-9 和 a-f)组成 字符串，基于 Git 中文件的内容或目录结构计算出来。SHA-1 哈希看起来是这样: 

```
24b9da6552252987aa493b52f8696cd6d3b00373
```

Git 中使用这种哈希值的情况很多，你将经常看到这种哈希值。实际上，Git 数据库中保存的信息都是以文件内 容的哈希值来索引，而不是文件名。 

### Git 一般只添加数据 

你执行的 Git 操作，几乎只往 Git 数据库中增加数据。**很难让 Git 执行任何不可逆操作，或者让它以任何方式清 除数据**。 



### 三个状态

Git 有三种状态，你的文件可能处 于其中之一:已提交(committed)、已修改(modified)和已暂存(staged)。 

- 已提交：表示数据已经安全的 保存在本地数据库中 。
- 已修改：表示修改了文件，但还没保存到数据库中。 
- 已暂存：表示对一个已修改文件的当前版 本做了标记，使之包含在下次提交的快照中。 

由此引入 Git 项目的三个工作区域的概念:Git 仓库、工作目录以及暂存区域。 

<img src="/Users/kuntang/Library/Application Support/typora-user-images/image-20191220172653452.png" alt="image-20191220172653452" style="zoom:50%;" />

- Git 仓库目录：Git 用来保存项目的元数据和对象数据库的地方。这是 Git 中最重要的部分，从其它计算机克隆 仓库时，拷贝的就是这里的数据。 
- 工作目录：它是对项目的某个版本独立提取出来的内容。这些从 Git 仓库的压缩数据库中提取出来的文件，放在磁盘 上供你使用或修改。 
- 暂存区域：它是一个文件，保存了下次将提交的文件列表信息，一般在 Git 仓库目录中。有时候也被称作`‘索 引’'，不过一般说法还是叫暂存区域。 

基本的 Git 工作流程如下: 

1. 在工作目录中修改文件。
2. 暂存文件，将文件的快照放入暂存区域。
3. 提交更新，找到暂存区域的文件，将快照永久性存储到 Git 仓库目录。 



## 分支

正如之前说的，Git 保存的不是文件的变化或者差异，而是一系列不同时刻的文件快照。 所以在进行提交操作时，Git 会保存一个提交对象(commit object)。 该提交对象会包含一个指向暂存内容快照的指针，还包含了作者的姓 名和邮箱、提交时输入的信息以及指向它的父对象的指针。 特别的是，**首次提交产生的提交对象没有父对象**，普通提交操作产生的提交对象有一个父对象，而由多个分支合并产生的提交对象有多个父对象。

![image-20191220165041898](/Users/kuntang/Library/Application Support/typora-user-images/image-20191220165041898.png)



举个例子，假设现在有一个工作目录，里面包含了三个将要被暂存和提交的文件。暂存操作会为每一个文件计算校验和(使用 SHA-1 哈希算法)，然后会把当前版本的 ***文件快照*** 保存到 Git 仓库中(Git 使用 blob 对象来保存它们)，最终将校验和加入到暂存区域等待提交: 

```bash
git add README test.rb LICENSE
git commit -m 'The initial commit of my project'
```

当使用git commit进行提交操作时，**Git会先计算每一个子目录(本例中只有项目根目录)的*校验和*，然后把 Git 仓库中这些 *校验和* 保存为树对象**。随后，Git 便会创建一个 ***提交对象*** ，它除了包含上面提到的那些信息外， 还包含**指向这个 *树对象* (项目根目录)的指针**。如此一来，Git 就可以在需要的时候重现此次保存的快照。 

<img src="/Users/kuntang/Library/Application Support/typora-user-images/image-20191220165843722.png" alt="image-20191220165843722" style="zoom:50%;" />

当做些修改后再次提交，那么这次产生的提交对象会包含一个指向上次提交对象(父对象)的指针。 

<img src="/Users/kuntang/Library/Application Support/typora-user-images/image-20191220170148667.png" alt="image-20191220170148667" style="zoom:50%;" />

**Git 的分支，其实本质上仅仅是指向提交对象的可变指针。**

**Git 的默认分支名字是 master。**在多次提交操作之后，你其实已经有一个指向最后那个提交对象的 master 分支。它会在每次的提交操作中自动向**前移动**。 

> NOTE: Git 的 “master” 分支并不是一个特殊分支。**它就跟其它分支完全没有区别**。之所以几乎每一个仓库都有master分支，是因为**git init命令默认创建它**，并且大多数人都懒得去改动它。 

<img src="/Users/kuntang/Library/Application Support/typora-user-images/image-20191220170409823.png" alt="image-20191220170409823" style="zoom:50%;" />

#### 分支创建

使用`git branch`命令新建一个分支：

```bash
git branch testing
```

这个操作会在当前提交对象上创建一个指针：

<img src="/Users/kuntang/Library/Application Support/typora-user-images/image-20191223171028751.png" alt="image-20191223171028751" style="zoom:50%;" />

#### 特殊的HEAD分支

Git有一个名为 HEAD 的特殊指针 ，它代表着当前处于本地的哪一个分支上。下图HEAD指向master分支上。

<img src="/Users/kuntang/Library/Application Support/typora-user-images/image-20191223171226771.png" alt="image-20191223171226771" style="zoom:50%;" />

你可以简单地使用git log命令查看各个分支当前所指的对象。提供这一功能的参数是--decorate。 

```bash
git log --oneline --decorate
```

#### 分支切换

要切换到一个已存在的分支，你需要使用git checkout命令。我们现在切换到新创建的testing分支去: 

```bash
git checkout testing
```

这样 HEAD 就指向 testing 分支了。 

<img src="/Users/kuntang/Library/Application Support/typora-user-images/image-20191223171359310.png" alt="image-20191223171359310" style="zoom:50%;" />

#### 分支合并与快进

使用git merge命令可以合并或快进分支：

```bash
git merge [branch A]
// 把分支A的代码合并到当前所在分支，或者当前所在分支快进到分支A
```

假设我们的分支如下图：

<img src="/Users/kuntang/Library/Application Support/typora-user-images/image-20191223172506946.png" alt="image-20191223172506946" style="zoom:50%;" />

切换到master分支，使用`git merge hotfix`后，分支图如下：

<img src="/Users/kuntang/Library/Application Support/typora-user-images/image-20191223172744981.png" alt="image-20191223172744981" style="zoom:50%;" />

假如现在的分支图如下：

<img src="/Users/kuntang/Library/Application Support/typora-user-images/image-20191223172849983.png" alt="image-20191223172849983" style="zoom:50%;" />

当前分支在master上，使用`git merge iss53`合并iss53代码到master分支后，分支图如下：

<img src="/Users/kuntang/Library/Application Support/typora-user-images/image-20191223173220102.png" alt="image-20191223173220102" style="zoom:50%;" />

#### 删除分支

使用`git branch -d`删除分支：

```bash
git branch -d iss53
```

### 分支开发工作流

因为 Git 使用简单的三方合并，所以就算在一段较长的时间内，反复把一个分支合并入另一个分支，也不是什么 难事。也就是说，在整个项目开发周期的不同阶段，你可以同时拥有多个开放的分支;你可以定期地把某些特性 分支合并入其他分支中。 

#### 长期分支 

比如 master 分支上保留完全稳定的代码——有可能仅 仅是已经发布或即将发布的代码。 

还有一些名为 develop 或者 next 的平行分支，被用来做后续开发或者 测试稳定性——这些分支不必保持绝对稳定，但是一旦达到稳定状态，它们就可以被合并入 master 分支了。 

事实上我们刚才讨论的，**是随着你的提交而不断右移的指针。稳定分支的指针总是在提交历史中落后一大截，而 前沿分支的指针往往比较靠前**。 

<img src="/Users/kuntang/Library/Application Support/typora-user-images/image-20191223173654963.png" alt="image-20191223173654963" style="zoom:50%;" />

<img src="/Users/kuntang/Library/Application Support/typora-user-images/image-20191223173710442.png" alt="image-20191223173710442" style="zoom:50%;" />

updates(建议更新)分支，它可能因包含一些不成熟的内容而不能进入 next 或者 master 分支。这么做的目的是使你的分支具有不同级别的稳定性。当它们具有一定程度的稳定性后，再把它们合并入具 有更高级别稳定性的分支中。再次强调一下，使用多个长期分支的方法并非必要，但是这么做通常很有帮助，尤 其是当你在一个非常庞大或者复杂的项目中工作时。 

#### 特性分支 

考虑这样一个例子，你在 master 分支上工作到 C1，这时为了解决一个问题而新建 iss91 分支，在 iss91 分 支上工作到 C4，然而对于那个问题你又有了新的想法，于是你再新建一个 iss91v2 分支试图用另一种方法解决 那个问题，接着你回到 master 分支工作了一会儿，你又冒出了一个不太确定的想法，你便在 C10 的时候新建 一个 dumbidea 分支，并在上面做些实验。你的提交历史看起来像下面这个样子: 

<img src="/Users/kuntang/Library/Application Support/typora-user-images/image-20191223214814884.png" alt="image-20191223214814884" style="zoom:50%;" />

现在，我们假设两件事情:你决定使用第二个方案来解决那个问题，即使用在 iss91v2 分支中方案;另外，你 将 dumbidea 分支拿给你的同事看过之后，结果发现这是个惊人之举。这时你可以抛弃 iss91 分支(即丢弃 C5 和 C6 提交)，然后把另外两个分支合并入主干分支。最终你的提交历史看起来像下面这个样子: 

<img src="/Users/kuntang/Library/Application Support/typora-user-images/image-20191223214837124.png" alt="image-20191223214837124" style="zoom:50%;" />

#### 远程分支 

可以通过git ls-remote (remote)来显 式地获得远程引用的完整列表，或者通过git remote show (remote)获得远程分支的更多信息 。

远程跟踪分支是远程分支状态的引用。它们是你不能移动的本地引用，当你做任何网络通信操作时，它们会自动移动。它们以 (remote)/(branch) 形式命名 。

> 远程仓库名字 “origin” 与分支名字 “master” 一样，在 Git 中并没有任何特别的含义一 样。 

