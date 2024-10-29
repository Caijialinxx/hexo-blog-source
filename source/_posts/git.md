---
title: Git 中的后悔药
date: 2019-04-06 18:11:05
tags: [Notes, 工作]
---

前言：
- `git fsck` & `git stash apply <commit_id>` ：针对 stash 存储的误删
- `git checkout -- <file>` ：针对工作区的修改
- `git reset` ：针对暂存区的修改



## 💊fsck —— 针对 stash 存储的误删
在不同的公司，技术团队在代码仓库中的协同工作流程可能会有不同。我公司就是在 dev 分支上进行开发，合并更新的流程基本就是下面：
1. `git stash` 存储自己在 dev 上做的修改（新建的文件是不会被存储起来的）
2. `git pull` 拉取远程仓库中的 dev 的更新
3. `git stash pop` 弹出存储的修改（有冲突的话就解决冲突）
4. 然后就是添加文件并提交修改那些操作。

那么我在工作中，遇到最让我窒息、心痛如绞的事情就是：我亲手把自己 stash 的修改给 clear 了..........

神操作如下：
1. 啊，终于可以提交了，我先 `git stash` 存起来先
2. 那么接下来查查看我的存储仓记录 `git stash clear` （一失足成千古恨）
3. 嗯？为什么没有显示存储记录？
4. ？？？我执行了 `git stash clear` ？？？ CLEAR？？？？我不是要 `git stash list` 的吗？？？
![快叫120！](http://wx4.sinaimg.cn/bmiddle/006Cmetyly1fiqiy4pl49j30ji0ji76r.jpg)

5. 我不信，一定没有清空掉的， `git stash list` .........
    (Sorry, the list you checked is empty. Du... Du... Du...)
6. 我还是不信，可能我没有 stash 到，还在工作区里。 `git status` ........
    (Sorry, the list you checked is empty. Du... Du... Du...)
![我要冷静，冷静！](http://wx2.sinaimg.cn/bmiddle/006Cmetyly1fiqiy3lo8fj30hs0hjjsm.jpg)


![哇的一声哭出来](http://img.soogif.com/eRbY86K0GLTliVxr5IxQozeR4mMcZ59O.gif_s400x0)
![这不是真的](http://img.soogif.com/zwkRqLIh6wFXoH1oYpqIlWF09rAE8AJM.gif_s400x0)
![我的心好痛](http://img.soogif.com/IgA7kfbo8vght7ulLXx5DZc4Cq35pkCm.gif_s400x0)

不哭，一定有解决方法的！
![一定有解决方法的！](http://img.soogif.com/rFZAtRGF1OA0LzyYHYp57nBiL1HHrVDG.gif_s400x0)

掉坑里的果然不止我一个人，各路大佬介绍了自己的解决方法，我找到符合自己的方法：
1. **`git fsck`** 查询仓库中所有未被其他对象引用的对象，这密密麻麻地列出了一摞（我记得当时不是时间顺序排序的，但是今天一看好像又是时间顺序的）。
2. 于是我只能 **`git show <commit_id>`** 一个个打开来看。
3. 经过漫长的版本查找后，我终于找到了离上一次修改最近的记录！最后 **`git stash apply <commit_id>`** 。谢天谢地，回来了！

![千里寻子终得你](https://i.loli.net/2019/04/17/5cb6cece961fe.jpg)


## 💊checkout —— 针对工作区的修改
对于在工作区的修改，还没执行 `git add` 等操作，此时若是想放弃工作区的全部修改，只需要：
```
git checkout -- <需要撤销修改的文件名>
```

注意：这个不针对 Untracked 的文件哦～

## 💊reset —— 针对暂存区的修改
对于刚执行完 `git add` 把文件添加到暂存区的修改，此时若是想放弃暂存区某个文件的修改，只需要：
```
git reset HEAD <需要撤销修改的文件名>
```

而如果你已经执行了 `git commit` 将这些暂存区的文件提交，那你只能：
```
git reset --hard HEAD^                ## 将 HEAD 回退到上一个版本
git reset --hard HEAD@{<index>}       ## 将 HEAD 回退到第 index 个版本
git reset --hard <commit_id>          ## 将 HEAD 指向指定的 commit_id 版本

git log         ## 查看提交的历史
git reflog      ## 查看 HEAD 移动的历史记录，从而回到任意版本
```
注意：这个操作非常危险！⚠️如果你的工作区中还有已跟踪的修改文件未提交，执行这个操作将会丢失你的这些文件！
