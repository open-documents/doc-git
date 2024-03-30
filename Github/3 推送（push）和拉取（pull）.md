
# push

将本地分支推送到指定远程仓库的指定分支。

语法：`git push <remote-name> <local-branch>:<remote-branch>`
- `<remote-name>`：将要推送到的远程仓库名称。
- `<local-branch>`：将要被推送的本地分支名称。
- `<remote-branch>`：将要被推送到的远程分支名称。

## 可能出现的问题




# pull

）从指定远程仓库拉取指定分支。

语法：`git pull <remote-name> <remote-branch>`
- `<remote-name>`：远程仓库名称。
- `<remote-branch>`：远程分支名称。

## 可能出现的问题

）不相关的历史

错误如下：
```bash
fatal: refusing to merge unrelated histories
```

原因：本地仓库和远程仓库没有关联，即本地仓库不是通过 `clone` 远程仓库创建的。

解决方法：加上 `--allow-unrelated-histories` 参数
```bash
$ git pull <remote-name> <remote-branch> --allow-unrelated-histories
```

）