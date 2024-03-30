
>参考文档：https://git-scm.com/book/en/v2/Getting-Started-What-is-Git%3F


---
<font color=fb463c>Git 与 其它VCS（Version Control System） 有什么不同？</font>Git 与其它 VCS 主要不同在于存储的数据内容。

<font color="00E0FF">其它VCS</font> 存储的是<font color=44cf57>文件、随时间对文件的修改</font>，所以也叫做基于delta（delta-based）的VCS。（如下图所示）

<img src="D:\NoteWithVersionControl\doc-git\1、八股与使用 -- Git 开始\PIC\Pasted image 20240326012948.png" width=75%/>

<font color="00E0FF">Git</font> 将每一个版本都认为是快照（snapshot），即每一个版本都包含的是文件而非对于文件的修改，同时存储每个版本的引用（可以理解为指针）。为了效率，如果某个版本中的某个文件较之前版本没有发生变化，那么该版本存储之前版本的链接。

<img src="D:\NoteWithVersionControl\doc-git\1、八股与使用 -- Git 开始\PIC\Pasted image 20240326013851.png" width=75%/>

---
<font color=fb463c>Git 操作有什么特点？</font>

Git 的几乎所有操作都是本地的，这意味着这些操作没有网络延迟，几乎是常量级别。

Git 因为客户端获取的是整个仓库及历史，因此诸如获取历史版本、比较版本之间的不同等操作，不需要向中心服务器请求，而只需要保存在本地数据库中的信息即可完成。

Git 几乎所有的操作都只是对 Git 数据库进行修改。

---
<font color=fb463c>什么是 Plumbing 命令、Porcelain 命令？</font>

>参考文档：https://git-scm.com/book/en/v2/Git-Internals-Plumbing-and-Porcelain#ch10-git-internals

Git 最初作为开发VCS的工具，而非对用户右好的VCS，有大量做底层工作的命令，这一类命令被设计成 UNIX 风格的链接或者从脚本中调用。 这一类命令称为 <font color="00E0FF">Plumbing 命令</font>。理解这一类命令能帮助更好地理解 Git 的工作原理，对于初学没有必要。

另外一类便于用户使用，也是普通用户接触最为广泛的命令即是 <font color="00E0FF">Porcelain 命令</font>，使用好这一类命令即可使用好 Git。

---
<font color=fb463c>如何理解 Git 是基于内容寻址（content-addressable）的文件系统？</font>

Git 的 .git/objects 目录保存 Git 完成功能的数据对象，所有的对象以键值的方式存储。可以向 Git 仓库中添加任何形式的值，然后通过对应的键来访问值。

具体而言，Git 在保存数据前会通过 SHA-1 算法，基于文件的内容（如果是文件）、目录的结构（如果是目录）获得文件、目录40位的哈希值，然后通过哈希值引用指向保存的数据。

一个 SHA-1 哈希值长下面这样。
```text
24b9da6552252987aa493b52f8696cd6d3b00373
```

---
<font color=fb463c>Git 的 3 块区域？</font>

Git 中有 3 块区域：<font color="00E0FF">工作目录（working tree、working directory）</font>、<font color="00E0FF">缓存区（staging area、index）</font>、<font color="00E0FF">数据库（.git directory ）</font>。

>关于 .git 目录，更加详细见 ...

<img src="D:\NoteWithVersionControl\doc-git\1、八股与使用 -- Git 开始\PIC\Pasted image 20240326015848.png" width=75%/>

<font color="00E0FF">工作目录</font> 是某个版本的快照中的所有内容，这些文件是从 .git目录中被压缩的数据库中取出，然后放到磁盘上等待修改。

<font color="00E0FF">缓存区</font> 保存的是下次待提交的内容，通常保存在 .git 目录中，注意缓存区中保存的是文件发生的改变。

<font color="00E0FF">.git 目录</font>保存的是元数据、对象数据库，clone 命令即是复制的该 .git 目录。


-- --
<font color=fb463c>Git 文件的 2 种、3 种状态（重要）</font>

Git 工作目录中文件有 2 种状态：untracked、tracked。

）untracked 表示该文件在最后一次提交（快照）中、缓存区中不存在

）tracked 的文件有 3 种状态：被修改（modified）、被缓存（staged）、被提交（committed）。
- 被修改：文件已经被修改，但是还没有提交到缓存区中。
- 被缓存：文件已经到了缓存区，将作为下一次commit的内容，即下一个版本快照中的内容。
- 被提交：文件已经存储到 Git 数据库中。

---
<font color=fb463c>使用 Git 的流程？</font>

通过按照下面的基本流程使用 Git：
- 修改工作目录中的文件
- 选择性地将文件发生的改变保存到缓存区
- 提交缓存区内容到 Git 数据库，当前版本的快照会永久地保存在 Git 数据库。





