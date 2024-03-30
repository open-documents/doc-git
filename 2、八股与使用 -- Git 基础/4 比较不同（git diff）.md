
>参考文档 1：https://git-scm.com/book/en/v2/Git-Basics-Recording-Changes-to-the-Repository（Viewing Your Staged and Unstaged Changes 部分）

>参考文档 2：


# git diff、--staged 选项

git diff 将缓存区中的文件与工作目录中的文件做比较，返回修改的部分。

---
下面介绍情境。

先做如下操作：

1）工作目录中，新建内容为 "hello1" 的文件 file1，新建内容为 "bala1" 的文件 file2
```bash
echo 'hello1' > file1
echo 'bala1' > file2
```
2）将 file1、file2 添加到缓存区。
```bash
git add *
```
3）提交缓存区。
```bash
git commit
```

做完上述操作后，3 个区域如下。
![[Pasted image 20240327211426.png]]

再做如下操作：

1）修改 file1 的内容为 "hello2"，将其缓存到缓存区。
```bash
echo "hello2" > file1
git add file1
```
2）修改 file2 的内容为 "bala2"，不将其缓存到缓存区。
```bash
echo "bala2" > file2
```
3）新建内容为 "haha1" 的文件 file3。
```bash
echo "haha1" > file3
```

做完上述操作后，3 个区域如下。
![[Pasted image 20240327220053.png]]

---
执行 git diff。
```bash
git diff

# diff --git a/file2 b/file2
# index 2ec6404..5097547 100644
# --- a/file2
# +++ b/file2
# @@ -1 +1 @@
# -bala1
# +bala2
```

可以看到：
- git diff 显示 unstaged 的文件与缓存区中同名文件之间的不同。
- untracked 的文件不在显示之列。

执行 git diff --staged。
```bash
git diff --staged
# diff --git a/file1 b/file1
# index 15b8f2a..14be0d4 100644
# --- a/file1
# +++ b/file1
# @@ -1 +1 @@
# -hello1
# +hello2
```

可以看到：
- `git diff --staged` 显示缓存区与最后一次提交（快照）之间的不同。简单理解，如果提交会造成的变化。