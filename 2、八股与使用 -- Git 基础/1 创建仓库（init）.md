
>参考：
>`https://git-scm.com/docs/git-init`
>`https://git-scm.com/book/en/v2/Git-Basics-Getting-a-Git-Repository`
>`https://git-scm.com/book/en/v2/Git-Internals-Plumbing-and-Porcelain#ch10-git-internals`

创建 Git 仓库有 2 种方式：<font color=44cf57>将没有版本控制的普通目录转变为 Git 仓库目录</font>、<font color=44cf57>克隆（clone）已有的 Git 仓库</font>。

---
<font color="00E0FF">初始化普通目录为 Git 仓库目录</font>

）进入普通目录。
```bash
cd /home/user/my_project
```

）执行 `git init` 命令。
```bash
git init
```

`git init` <font color=44cf57>创建一个空的Git仓库</font> 或者 <font color=44cf57>重新初始化已有的Git仓库</font>。该命令默认做 2 件事情：
- 在特定目录（默认git仓库目录）下创建 `.git` 目录
- 创建一个没有任何 commit 的 branch

---
<font color="00E0FF">克隆（clone）已有的 Git 仓库目录</font>

）进入存放所有工程的目录。
```bash
cd workspace
```

）执行 `git clone` 命令。
```bash
git clone https://github.com/loftytopping/Aerosol_CDT_modelling
# Cloning into 'Aerosol_CDT_modelling'...
# remote: Enumerating objects: 207, done.
# remote: Counting objects: 100% (207/207), done.
# remote: Compressing objects: 100% (165/165), done.
# remote: Total 207 (delta 102), reused 128 (delta 42), pack-reused 0
# Receiving objects: 100% (207/207), 25.22 MiB | 417.00 KiB/s, done.
# Resolving deltas: 100% (102/102), done.
```
上述命令，会在 `workspace` 目录下创建一个 `Aerosol_CDT_modelling` 目录，其中包含了 `Aerosol_CDT_modelling` 所有文件及历史。

）（可选）变更本地仓库名称。
```bash
git clone https://github.com/loftytopping/Aerosol_CDT_modelling alias
```
上述命令，会在 `workspace` 目录下创建一个 `alias` 目录，其中包含了 `Aerosol_CDT_modelling` 所有文件及历史。

`git clone` 命令默认执行 3 件事情：
- 初始化 .git 仓库目录
- 拉取所有数据
- checkout 到最新版本

除了上述 `https` 协议，`git clone` 能够支持` git://` 、`user@server:path/to/repo.git` 两类 SSH 协议的方式。

---
# .git 目录介绍

下面是默认的初始 `.git` 目录结构。
```text
.git
  |----hooks(d,包含很多.sample文件)
  |----info(d)
  |     |----exclude
  |----objects(d)
  |        |----info(d)
  |        |----pack(d)
  |----refs(d)
  |      |----heads(d)
  |      |----tags(d)
  |----config(f)
  |----description 
  |----HEAD 
```

与 `git init` 命令相关的3个环境变量：
- `$GIT_DIR` -- 用来替换 `./.git`
- `$GIT_OBJECT_DIRECTORY` -- 保存 `sha1目录` 的目录，默认为 `$GIT_DIR/objects`

其中重要的是：HEAD（文件）、index（文件，默认还没有创建）、objects（目录）、refs（目录）。

## objects

>参考：https://git-scm.com/book/en/v2/Git-Internals-Git-Objects

Git 对象数据库（object database），以键值对的方式存放任何类型的值，通过键来获取对应的值。

---
`git hash-object` <font color=44cf57>向 Git 对象数据库中存入值，返回对应的哈希值，作为键。</font>
- -w：不仅仅计算给定内容的hash值，同时向 Git 对象数据库中写入内容
- --stdin：从标准输入中获取内容，如果不指定，需要指定文件名作为代替

）3 个例子。
```bash
echo 'test content' | git hash-object -w --stdin
# d670460b4b4aece5915caf5c68d12f560a9fe3e4
```

```bash
echo 'version 1' > test.txt
git hash-object -w test.txt
# 83baae61804e65cc73a7201a7252750c76066a30
```

```bash
echo 'version 2' > test.txt   # 修改了test.txt内容, 以验证版本控制
git hash-object -w test.txt
# 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a
```

）返回的哈希值一共40位，前2位作为文件夹名称，后38位作为对象文件名。
```bash
find .git/objects -type f
# .git/objects/1f/7a7a472abf3dd9643fd615f6da379c4acb3e3a
# .git/objects/83/baae61804e65cc73a7201a7252750c76066a30
# .git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4
```

---
`git cat-file` <font color=44cf57>通过hash键，从 Git 对象数据库中取值。</font> 
- -p：理解对象类型，然后显示对象内容
- -t：查看对象类型。包含 blob、tree
```bash
git cat-file -p d670460b4b4aece5915caf5c68d12f560a9fe3e4
# test content
```

```bash
git cat-file -t d670460b4b4aece5915caf5c68d12f560a9fe3e4
# blob
```

）现在可以删除本地的test.txt文件，然后从 Git 对象数据库中获取。
```bash
rm test.txt
```

```bash
git cat-file -p 83baae61804e65cc73a7201a7252750c76066a30 > test.txt
cat test.txt
# version 1
```

```bash
git cat-file -p 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a > test.txt
cat test.txt
# version 2
```

）



<font color="00E0FF">objects/pack</font>



<font color="00E0FF">objects/info</font>


---
<font color="00E0FF">refs</font>

```text
.git
  |----refs
         |----heads
         |----remotes
         |----tags
```

<font color="00E0FF">refs/heads</font>

该目录下的文件数量、文件名称都与该仓库中分支的数量、名称一致，每个文件中存储的是当前分支的 commit 节点 id。

举个例子：如果当前仓库有master、custom两个分支，那么该目录下也就会有master、custom 2个文件

---
<font color="00E0FF">HEAD</font>

指向当前 checkout 的分支。


--- 
<font color="00E0FF">index</font>

git add 后添加，存储暂存区内容。



-- --
<font color="00E0FF">logs</font>

```text
.git
  |----logs(d)
         |----refs(d)
         |      |----heads(d)
         |
         |----HEAD
```

git commit 后创建

<font color="00E0FF">logs/refs/heads</font>


<font color="00E0FF">logs/HEAD</font>



-- --

<font color="00E0FF">COMMIT_EDITMSG</font>

保存着最近一次的提交信息,Git系统不会用到这个文件，只是给用户一个参考。

-- --
description 

被 GitWeb 程序使用

---
config

当前 Git 仓库的配置文件。

--- 
info/exclude

keeps a global exclude file for ignored patterns that you don’t want to track in a .gitignore file.

---
hooks

https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks#_git_hooks

