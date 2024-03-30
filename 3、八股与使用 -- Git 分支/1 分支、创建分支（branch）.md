
>参考文档 1：https://git-scm.com/docs/git-branch
>参考文档 2：https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell

下面介绍什么是分支？Git 中分支是一个指向一次提交的指针。

下面通过情境来理解。沿用 [[3 修改状态（add、commit、rm）]] 的情境。
<img src="D:\NoteWithVersionControl\doc-git\3、八股与使用 -- Git 分支\PIC\Pasted image 20240330043647.png" width=55%/>

默认分支为master。

---
创建分支

使用 `git branck <branch-name>` 创建一个新的分支，会创建一个指向当前提交对象的指针。需要注意的是，git branch 只是创建新的分支，并不切换到新的分支。
```bash
git branch testing
```

<img src="D:\NoteWithVersionControl\doc-git\3、八股与使用 -- Git 分支\PIC\Pasted image 20240330044126.png" width=55%/>

---
切换分支

使用 `git checkout <branch-name>` 切换当前分支到指定分支。
```bash
git checkout testing
```

此时 HEAD 指向 testing 指针。

<img src="D:\NoteWithVersionControl\doc-git\3、八股与使用 -- Git 分支\PIC\Pasted image 20240330044601.png" width=55%/>

切换分支有什么影响？

切换分支，会 1）让 HEAD 指针指向切换分支的引用；2）使用切换的分支指针指向的提交对象指向的快照对象更新缓存区；3）使用缓存区更新工作目录。

切换分支，新的提交时，会让当前分支指向新的提交对象，而其它分支保持不动。

通过情境来帮助理解。修改 file1，做一次新的提交。
```bash
echo "not hello world!" > file1
git add file1 
git commit
```

此时提交历史如下。
<img src="D:\NoteWithVersionControl\doc-git\3、八股与使用 -- Git 分支\PIC\Pasted image 20240330045241.png" width=66%/>

切换回分支，修改file1，做一次新的提交。

```bash
git checkout master
echo "i am to play!" > file1
git add file1
git commit
```

此时提交历史如下，可以看到提交历史发生了分叉。
<img src="D:\NoteWithVersionControl\doc-git\3、八股与使用 -- Git 分支\PIC\Pasted image 20240330050147.png" width=60%/>

使用 `git log --oneline --decorate --graph --all` 可以查看完整的提交日志。
```bash
git log --oneline --decorate --graph --all
# * 4829432 (HEAD -> master)      modified:   file1
# | * 74e870e (testing)   modified:   file1
# |/
# * a168ab6       modified:   file1
# * 444ae25       modified:   file1
# * eff8a09       new file:   dir/file3   new file:   file1       new file:   file2
```











）查询分支（git branch --list）

查询分支，当前分支以高亮显示。

常用参数：
- -r -- 显示远程分支
- -a -- 显示所有分支（即本地、远程分支）