
>参考文档：`https://git-scm.com/book/en/v2/Git-Tools-Reset-Demystified#_git_reset`

在介绍 reset、checkout 命令的底层之前，

# HEAD、Index、Working Directory

Git 有 3 个区域：HEAD（之前叫 Git 数据库）、Index（缓存区，也叫 Staging Area）、Working Directory（工作目录，也叫 Working Tree）。

---
HEAD

HEAD，即为 `.git/HEAD`，指向当前所在分支的引用，进一步指向该分支最后一次提交（快照）。

下面是 `.git/HEAD` 的内容。 
```text
ref: refs/heads/master
```
下面是 `.git/refs/heads/master` 的内容，是最后一次提交对象的哈希值。
```test
e65b36b7e0f6cf132853819fac213c34f59101d8
```
最后一次提交对象中保存着对应的快照对象的哈希值。（下面 1、2 命令等价）
```bash
git cat-file -p HEAD                                      # 1
git cat-file -p e65b36b7e0f6cf132853819fac213c34f59101d8  # 2
# tree cc9809c563c0a65d4eae3f56b319d30d53043117
# author open-docs <13439022108@163.com> 1711559480 +0800
# committer open-docs <13439022108@163.com> 1711559723 +0800
# 
# Changes to be committed:
# new file:   file1
# new file:   file2
```
通过快照对象的哈希值能够获取快照对象。（下面 1、2 命令等价）
```bash
git ls-tree -r HEAD                                       # 1
git ls-tree cc9809c563c0a65d4eae3f56b319d30d53043117      # 2
# 100644 blob ce013625030ba8dba906f756967f9e9ca394464a    file1
# 100644 blob 5320b231d56588f569e2a2795347af9a566fc512    file2
```

因此可以简单地理解，<font color=44cf57>HEAD 指向某一分支最后一次提交的快照。</font>

---
Index

Index，缓存区，功能不做介绍。

通过 `git ls-files -s` 能够查看缓存区中的内容。
```bash
git ls-files -s
# 100644 ce013625030ba8dba906f756967f9e9ca394464a 0       file1
# 100644 5320b231d56588f569e2a2795347af9a566fc512 0       file2
```

---
Working Directory

Working Directory，工作目录，容易理解不做过多介绍。


# 3 个区域内文件的关系（Workflow）

3 个区域内文件、目录的转化关系如下图所示。

<img src="D:\NoteWithVersionControl\doc-git\7、八股与使用 -- Git tools\PIC\Pasted image 20240328021017.png" width=75%/>

下面通过一个案例来理解。

）初始有一个没有Git控制的目录，初始化为 Git 仓库，其中包含一个文件 file.txt（版本记为 v1，用蓝色表示）。

HEAD 指向分支 master 的引用（即 HEAD 中的内容为 ref: refs/heads/master）

因为该分支没有提交，所以该引用没有指向任何提交对象。（即 refs/heads/master 中的内容为空）

<img src="D:\NoteWithVersionControl\doc-git\7、八股与使用 -- Git tools\PIC\Pasted image 20240328021327.png" width=50%/>

）依次执行 git add、git commit，执行第一次提交。

<center class = "half"><img src="D:\NoteWithVersionControl\doc-git\7、八股与使用 -- Git tools\PIC\Pasted image 20240328021802.png" width=50%/><img src="D:\NoteWithVersionControl\doc-git\7、八股与使用 -- Git tools\PIC\Pasted image 20240328021950.png" width=48.7%/></center>

此时 `refs/heads/master` 中提交对象指向的快照对象的hash值为 eb43bf8。

）修改 file.txt（版本记为 v2，用红色表示），再依次执行 git add、git commit。
<figure class="third"> <img src="D:\NoteWithVersionControl\doc-git\7、八股与使用 -- Git tools\PIC\Pasted image 20240328022536.png" width=32%/> <img src="D:\NoteWithVersionControl\doc-git\7、八股与使用 -- Git tools\PIC\Pasted image 20240328022702.png" width=32%/> <img src="D:\NoteWithVersionControl\doc-git\7、八股与使用 -- Git tools\PIC\Pasted image 20240328022731.png" width=32%/> </figure>
此时 `refs/heads/master` 中提交对象指向的快照对象的hash值为 9e5e6a4（发生了变化）。

）再次修改 file.txt（版本记为 v3，用黄色表示），再依次执行 git add、git commit。

<img src="D:\NoteWithVersionControl\doc-git\7、八股与使用 -- Git tools\PIC\Pasted image 20240328022923.png" width=60%/>

此时 `refs/heads/master` 中提交对象指向的快照对象的hash值为 38eb946（再次发生了变化）。

需要注意的是，HEAD 中的内容一直是 ref: refs/heads/master，变换的是 refs/heads/master 中的内容（即提交对象的哈希值）。


# reset

下面介绍 reset 命令的流程，分为 3 步：
- 移动 HEAD（更准确地说是修改分支引用的指向，仍然处于 master 分支）（--soft 选项，完成这一步停止）
- 反向更新 Index（--mixed 选项，完成这一步停止，也是默认项）
- 反向更新 Working Directory（--hard 选项，完成所有 3 步）

需要注意的是，--hard 是非常危险的选项，因为前 2 步可以简单地反向操作恢复。如果 Working Directory 中的内容没有在 Git 数据库中保存，--hard 选项会让 Working Directory 中原有的内容丢失。

---
修改分支引用的指向，快照对象发生变化。

执行 `git reset 9e5e6a4`，refs/heads/master 的内容不再是指向哈希值38eb946快照对象的提交对象哈希值，而变为指向 哈希值9e5e6a4快照对象的提交对象哈希值。  

<img src="D:\NoteWithVersionControl\doc-git\7、八股与使用 -- Git tools\PIC\Pasted image 20240328023840.png" width=60%/>

---

反向更新 Index

<img src="D:\NoteWithVersionControl\doc-git\7、八股与使用 -- Git tools\PIC\Pasted image 20240328024354.png" width=60%/>

---
反向更新 Working Directory。 

<img src="D:\NoteWithVersionControl\doc-git\7、八股与使用 -- Git tools\PIC\Pasted image 20240328024800.png" width=60%/>


# reset 传递路径参数

当向 git reset 传递路径（文件）时，reset 会跳过第 1 步，然后将第 2、3 步的作用范围限制在给定的文件内。

---
）下面来解释 `git reset HEAD <file>` 为什么能撤销添加文件到缓存区。

`git reset HEAD <file>` 等价于 `git reset --mixed HEAD <file>`、`git reset <file>`，结果如下。

<img src="D:\NoteWithVersionControl\doc-git\7、八股与使用 -- Git tools\PIC\Pasted image 20240328025503.png" width=60%/>

---
）git reset 也有只更新 Index 的作用。

执行 `git reset eb43bf file.txt`

<img src="D:\NoteWithVersionControl\doc-git\7、八股与使用 -- Git tools\PIC\Pasted image 20240328025955.png" width=60%/>

---
）git reset 也有压缩提交历史的作用。

假设有如下的提交历史，如果 9e5e6a4 只是中间提交，可以删除的话，可以利用 git reset 完成压缩的效果。
<img src="D:\NoteWithVersionControl\doc-git\7、八股与使用 -- Git tools\PIC\Pasted image 20240328030203.png" width=60%/>

执行 `git reset --soft HEAD~2`，3 个区域、master 指向变化如下。
<img src="D:\NoteWithVersionControl\doc-git\7、八股与使用 -- Git tools\PIC\Pasted image 20240328030342.png" width=60%/>
执行提交。
<img src="D:\NoteWithVersionControl\doc-git\7、八股与使用 -- Git tools\PIC\Pasted image 20240328030537.png" width=60%/>

# checkout 与 reset 的区别

checkout 与 reset 在传递路径（文件）时没有区别，主要区别在于没有传递路径（文件）。下面介绍在没有传递路径（文件）时，checkout 与 reset 的区别。

没有传递路径（文件）时，`git checkout [branch]` 功能有点类似 `git reset --hard [branch]`，主要有 2 个方面的区别。

）

First, unlike `reset --hard`, `checkout` is working-directory safe; it will check to make sure it’s not blowing away files that have changes to them. Actually, it’s a bit smarter than that — it tries to do a trivial merge in the working directory, so all of the files you _haven’t_ changed will be updated. `reset --hard`, on the other hand, will simply replace everything across the board without checking.


）checkout 更新 HEAD，而 reset 不更新 HEAD

通过一个例子进行说明，现有2个分支（master、develop），目前在 develop 分支。

`git reset master`：不更改 HEAD 内容（即目前所在分支仍然为 develop），仅更改 `refs/heads/develop` 的内容为  `refs/heads/master` 的内容。

`git checkout master`：更改 HEAD 内容从 `ref: refs/heads/develop` 为 `ref: refs/heads/master`（所在分支发生变化），不更改 `refs/heads/develop`、 `refs/heads/master` 的内容。

<img src="D:\NoteWithVersionControl\doc-git\7、八股与使用 -- Git tools\PIC\Pasted image 20240328031015.png" width=60%/>

