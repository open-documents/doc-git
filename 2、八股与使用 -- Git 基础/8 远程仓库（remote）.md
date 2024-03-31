
>参考文档：https://git-scm.com/book/en/v2/Git-Basics-Working-with-Remotes

远程仓库，保存在网络中其它位置（可以是其它服务器上，也可以是本地上）的 Git 仓库。通过远程仓库，能够协作开发。下面以 GitHub 作为远程仓库服务器为例展开介绍。

---
）执行 `git clone <remote-repository>`，克隆远程仓库到本地目录中，初始化本地仓库。 
```bash
git clone https://github.com/open-documents/doc-git.git
```

git clone 除了获取远程仓库中的所有数据外，另外做了 2 件事：
- 自动在本地仓库中添加名为 origin 的远程仓库，相当于执行了 `git add origin <remote-repository>` 命令。
- 自动将本地仓库中创建的默认分支（master）绑定到远程仓库的默认分支（master）。

）使用 `git push <remote> <branch>` 推送（push）当前分支内容到远程仓库。其中，remote 使用添加的远程仓库的 shortname 代替。 

```bash
git push doc-git master 
```

此时，会弹出认证窗口，认证当前用户是否拥有向远程仓库的 push 权限。

在认证成功后，本地仓库的 master 分支内容会推送到远程仓库的指定分支（因为第一次推送，远程仓库默认创建同名分支master）。
```bash
# Enumerating objects: 83, done.
# Counting objects: 100% (83/83), done.
# Delta compression using up to 12 threads
# Compressing objects: 100% (82/82), done.
# Writing objects: 100% (83/83), 2.45 MiB | 1003.00 KiB/s, done.
# Total 83 (delta 0), reused 0 (delta 0), pack-reused 0
# To https://github.com/open-documents/doc-git
#  * [new branch]      master -> master
```

---
情境2：其它协作者提交自身commit到远程仓库中，此时远程仓库与本地仓库中内容不一致。

使用 `git fetch <remote-repository>` 从远程仓库中获取所有数据。需要注意的是：
- 如果没有指定 `<remote-repository>`，则 `remote-repository` 默认为origin。
- `git fetch` 只是下载远程仓库数据到本地仓库中，并不会自动合并远程仓库当前分支与本地当前分支。

使用 `git pull <remote-repository>` 从远程仓库中获取所有数据，并自动将远程仓库当前分支合并到本地仓库当前分支，前提是本地当前分支已经绑定（track）一个远程仓库分支。


---
在本地仓库中，往往需要与远程仓库互动，如 git push、git pull、git fetch 等，需要指定远程仓库，而远程仓库可以是 url 的形式，通过取别名的方式，可以用较短的名称来代替远程仓库。

）使用 `git remote add <shortname> <url>` 添加远程仓库。此后，shortname 代表远程仓库的 url。 

```bash
git remote add doc-git https://github.com/open-documents/doc-git
```

）使用 `git remote rename <shortname-old> <shortname-new>` 来更新远程仓库名称。这会更新所有远程分支名称，即将 `doc-git/master` 更新为 `doc-git-remote/master`
```bash
git remote rename doc-git doc-git-remote
# Renaming remote references: 100% (1/1), done.
```

）使用 `git remote remove <shortname>` 或 `git remote rm <shortname>` 来移除远程仓库（执行完下面的例子，记得重新添加远程仓库）。
```bash
git remote rm doc-git-remote
```

---
）使用 `git remote show <remote>` 查看更多远程仓库信息。其中，remote 可以用 `git remote add` 时指定的 shortname 代替。
```bash
git remote show doc-git
# * remote doc-git
#   Fetch URL: https://github.com/open-documents/doc-git
#   Push  URL: https://github.com/open-documents/doc-git
#   HEAD branch: master
#   Remote branch:
#     master tracked
#   Local ref configured for 'git push':
#     master pushes to master (up to date)
```

--- 
