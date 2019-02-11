---
title: 关于git stash的使用小结
date: 2018-08-04 11:14:24
tags:
- git
categories:
- tutorial
keywords:
description:

---



以前使用svn作为版本控制时，每个branch都在本地有一份代码，这样可以同时打开好多个分支。在一个分支加feature，可以很方便的另一个分支上修bug。在切换为git之后，默认所有branch都在本地只有一份copy，有的时候需要临时切换下分支，但不想提交还未完工的代码，会使用到git stash来暂存下代码。然后bug修改之后，再使用git stash pop将暂存的代码恢复过来。



但是在使用了几次git stach之后，经常会遇到一些问题：忘记pop或者pop到了错误的分支导致冲突。本文结合再v2ex上的一些回答，建议最好不要使用git stash，直接通过commit然后最后再通过git reset来回到原来的commit点，完成之后重新commit。



<!--more-->

## 使用git stash

stash暂时将你做的修改保存起来，然后恢复到未修改的状态。这样可以去做其他事，比如你已经修改了一些东西，但是现在还不能提交，但是要去修改其他功能

>The git stash command takes your uncommitted changes (both staged and unstaged), saves them away for later use, and then reverts them from your working copy

- git stash

  将当前工作保存

- git stash show -p stash@{1}

  显示哪些文件被修改。如果不指定某个stash，则显示最近的一次stach的修改情况

- git stash list

  查看有哪些stash，分别位于哪个分支上

- git stash pop

  将暂存的修改恢复到本地中并删掉该stash，注意在git stash pop之前先确认下stash是在哪个分支的,使用git stash list来查看

- git stash apply

  与git stash pop的区别是，apply不会将stash删除

- git stash clear

  删除所有的stash entry， 不可恢复

- git stash drop <stash>

  删除某一个stash entry，如果不给出具体stash，则删除最新的一个

## 使用git commit

假设遇到需要暂时保存的情况可以选择将修改记录直接commit，commit message可以写WIP commit（work in progress）。这样在完成其他工作之后，通过git reset  --mixed 可以恢复到commit时的状态，然后完成feature，重新填写commit message

```bash
//you are adding a feature, then a bug come in ,you have to fix bug
git add .
git commit -m "WIP commit: i will be back"
...
//after the bug, xxxxxxxxxxx is the WIP commit SHA-1
git reset --mixed xxxxxxxxxxx
//THEN you finish the feature, and remend the commit message
git commit -m "i am done"
```

## git worktree

如果需要同时checkout多个branch到本地，可以使用git worktree，这个命令与git clone多份代码的区别是，使用git worktree可以使得多个worktree之间有联系，比如可以直接将其他worktree中merge过来。

git worktree可以支持不同的worktree attach到同一个repo上,方便同时查看两个分支的代码 .

> A git repository can support multiple working trees, allowing you to check out more than one branch at a time. With git worktree add a new working tree is associated with the repository. This new working tree is called a "**linked working tree**" as opposed to the "**main working tree**" prepared by "git init" or "git clone". A repository has one main working tree (if it’s not a bare repository) and zero or more linked working trees. When you are done with a linked working tree, remove it with git worktree remove.

- git worktree add ../new_directory feature_branch

  添加worktree

- git worktree list

  列出所有worktree

- git worktree move <worktree> <new-path>

  移动worktree到新路径

- git worktree remove [-f] <worktree>

  移除worktree，只有clean的worktree才可以被删除掉

## git workflow 

网上推崇的workflow，具体参见这个[blog](https://nvie.com/posts/a-successful-git-branching-model/)

{% asset_img git-model.png %}

