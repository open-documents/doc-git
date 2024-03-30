
>参考文档 1：https://git-scm.com/book/en/v2/Git-Basics-Recording-Changes-to-the-Repository



# 提交（commit）

>参考文档：https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell

下面介绍什么是提交？

当做一次提交时，Git 创建一个存储在 Git 数据库的提交对象，提交对象包含的内容有：指向快照对象的 Hash 值（通过该 Hash 值可以访问快照对象）、提交者的信息（名称、邮箱）、键入的提交信息、<font color="00E0FF">一个或多个提交对象</font>。

---
下面用情境来帮助理解。

现有2个文件（file1、file2）、1个包含1个文件（file3）的目录（dir）。
```
repository
    |---- file1
    |---- file2
    |---- dir
           |----file3
```

执行 `git add *`、`git commit`，执行第1次提交。

此时 git 对象数据库有6（或当file1、file2、file3内容相同时，4）个对象。

）file1 对象、file2 对象、file3 对象。（如果 file1、file2、file3内容相同，则只有一个对象）

）提交对象。
```bash
git cat-file -p aefdcacbfc916ab41afb0d0994cbc7550b79f46f
# tree 007eab280e9421d093304733498474f4e386583c
# author open-docs <13439022108@163.com> 1711740742 +0800
# committer open-docs <13439022108@163.com> 1711740742 +0800
# 
#         new file:   dir/file3
#         new file:   file1
#         new file:   file2
```
）2 个 tree 对象：根目录、dir目录
```bash
git cat-file -p f85838974eeb7155f764eca9004070f04626c67e                  # 根目录 tree 对象
# 040000 tree 0b55db39f55c1baaf6c6e334c5f37a00e182991d    dir
# 100644 blob ce013625030ba8dba906f756967f9e9ca394464a    file1
# 100644 blob 5ad28e22767f979da2c198dc6c1003b25964e3da    file2

git cat-file -p 0b55db39f55c1baaf6c6e334c5f37a00e182991d                  # dir目录 tree对象
# 100644 blob 91ca0fad0eed0693c32237b7cb5da41f7fdf3464    file3
```

git 对象数据库内容如下。（橙色为提交对象，粉红为tree目录对象，淡蓝色为blob文件对象）
<img src="D:\NoteWithVersionControl\doc-git\2、八股与使用 -- Git 基础\PIC\Pasted image 20240330040656.png" width=75%/>

）修改 file1 内容，执行第2次提交。
）修改 file1 内容，执行第3次提交。

此时，3次提交之间的关系如下，需要注意的是：
- 尽管3次提交对象中的tree对象都指向根目录，但是因为根目录中的file1文件发送了变化，所以根目录tree对象哈希值发生了变化，即 f85838 -> 3993b0 -> df6993。
- 除了第1次提交，新提交都有一个指针指向前一次提交对象。

<img src="D:\NoteWithVersionControl\doc-git\2、八股与使用 -- Git 基础\PIC\Pasted image 20240330042132.png" width=75%/>

每一次新的提交（指简单的 `git commit`），分支指针（默认master）都向前移动，指向最新的提交对象。（关于什么是分支，见[[1 分支、创建分支（branch）]]）

<img src="D:\NoteWithVersionControl\doc-git\2、八股与使用 -- Git 基础\PIC\Pasted image 20240330043419.png" width=75%/>

# rm

>参考文档1：https://git-scm.com/book/en/v2/Git-Basics-Recording-Changes-to-the-Repository（Removing Files 部分）

先说结论，再用情境说明。

rm（右键删除）：删除 working directory 中的文件。

git rm：删除 staging area 中的文件（个人理解：添加删除修改到缓存区）。

---
准备环境。

）新建内容为 "hello1" 名称为 file1 的文件，添加到缓存区。
```bash
echo "hello1" > file1
git add file1
```
）新建内容为 "bala1" 名称为 file2 的文件，不添加到缓存区。
```bash
echo "bala1" > file2
```

执行 git status。
```bash
git status
# On branch master
# No commits yet
# Changes to be committed:
#   (use "git rm --cached <file>..." to unstage)
#         new file:   file1
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#         file2
```
现在的 3 个区域如下。
![[Pasted image 20240327222103.png]]

---
）执行 `rm file2`。
```bash
rm file2

git status
# On branch master
# No commits yet
# Changes to be committed:
#   (use "git rm --cached <file>..." to unstage)
#         new file:   file1
```
可以看到，对于只存在working directory 的文件（即untracked 文件），rm（右键删除）即可。现在的 3 个区域如下。

![[Pasted image 20240327222404.png]]


）执行 `rm file1`。
```bash
rm file1

git status
# On branch master
# No commits yet
# Changes to be committed:
#   (use "git rm --cached <file>..." to unstage)
#         new file:   file1
# Changes not staged for commit:
#   (use "git add/rm <file>..." to update what will be committed)
#   (use "git restore <file>..." to discard changes in working directory)
#         deleted:    file1
```
可以看到，rm 只是删除了 working directory 中的文件，没有删除 staging area 中的文件，这样会导致 git commit 依然会提交在working directory 删除了的文件。

现在的 3 个区域如下。
![[Pasted image 20240327222613.png]]

）执行 `git rm file1`。
```bash
git rm file1

git status
# On branch master
# No commits yet
# nothing to commit (create/copy files and use "git add" to track)
```
可以看到，git 
