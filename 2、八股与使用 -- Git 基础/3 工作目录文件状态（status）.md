
>参考文档 1：https://git-scm.com/docs/git-status
>参考文档 2：https://git-scm.com/book/en/v2/Git-Basics-Recording-Changes-to-the-Repository

在创建了 Git 仓库后，会对工作目录中的文件进行增、删、改的操作，工作目录中的文件的状态会随之发生改变。

工作目录中的文件有 2 种状态：tracked、untracked。

）untracked 表示该文件在最后一次提交（快照）中、当前缓存区中都不存在。

）tracked 的文件有 4 种状态：
- 没有被修改（unmodified）：工作目录中的文件相对于最后一次提交（快照）没有修改。
- 被修改（modified）：工作目录中的文件已经被修改，但是还没有提交到缓存区中。
- 被缓存（staged）：工作目录中的文件发生的变化已经拷贝到了缓存区，将作为下一次commit的内容，即下一个版本快照中的内容。
- 被提交（committed）：工作区目录的文件发生的变化已经提交到了 Git 数据库中。

<img src="D:\NoteWithVersionControl\doc-git\2、八股与使用 -- Git 基础\PIC\Pasted image 20240327114004.png" width=80%/>

---
使用 git status 查看工作目录文件的状态。

）工作目录与最后一次提交（快照）完全一致。
```bash
git status
# On branch master
# nothing to commit, working tree clean
```

---
下面介绍

）工作目录存在 untracked 的文件。

```bash
echo 'My Project' > README

git status
# On branch master
# No commits yet
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#         README
# nothing added to commit but untracked files present (use "git add" to track)
```

）追踪（track） untracked 文件。（working directory -> staging area）

```bash
git add README

git status
# On branch master
# No commits yet
# Changes to be committed:
#   (use "git rm --cached <file>..." to unstage)
#         new file:   README
```

）提交缓存区内容。（staging area -> .git directory）
```bash
git commit 
```

会使用之前配置的文本编辑器显示下面的内容。
```text
# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# On branch master
# Changes to be committed:
#   new file:   README                  1(原始文本不包含, 为了说明清楚, 标注为第1行) 
#
```
将打算提交的缓存区中的内容前的注释取消掉（如第 1 行），然后关闭文本编辑器，缓存区中的内容即可选择性地提交到 .git 仓库。

此时，工作目录与最后一次提交（快照）保持完全一致。
```bash
git status
# On branch master
# nothing to commit, working tree clean
```
---
下面介绍

）工作目录存在被修改的文件。

```bash
echo 'hello world' > README       # 修改工作目录中的文件

git status
# On branch master
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#   (use "git restore <file>..." to discard changes in working directory)
#         modified:   README
# no changes added to commit (use "git add" and/or "git commit -a")
```

）缓存修改。（working directory -> staging area）

```bash
git add 

git status
# On branch master
# Changes to be committed:
#   (use "git restore --staged <file>..." to unstage)
#         modified:   README
```

---

下面介绍工作目录、缓存区存在相同的文件，继续修改工作目录中的文件。

```bash
echo 'hello 1' > README   # 修改工作目录中的 README
```

```bash
git status
# On branch master
# Changes to be committed:
#   (use "git restore --staged <file>..." to unstage)
#         modified:   README
# 
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#   (use "git restore <file>..." to discard changes in working directory)
#         modified:   README
```

---

# Short Status

git status 显示比较复杂，使用 -s 或 --short 选项来简略显示。

```
```


