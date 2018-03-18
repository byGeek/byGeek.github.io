---
title: 如何删除github敏感文件
date: 2018-02-24 16:35:47
tags:
- github
- bfg
---

### 前言

由于当初建立博客的失误，我将一些不可名状的东西上传到了github中，今天才发现，吓的我赶紧找方法如何删除。google了一番之后终于找到了一个repo-cleaner工具：[BFG](https://rtyley.github.io/bfg-repo-cleaner/)。 下面就简单记录下如何删除已经commit到服务器上的文件，同时删除commit记录。

<!--more-->

### 步骤

1. 在你的local working directory中将你要删除的文件删除，正常commit并push到remote，因为BFG工具会使用最新的一次commit。

   > The BFG treats you like a reformed alcoholic: you've made some mistakes in the past, but now you've cleaned up your act. Thus the BFG assumes that your latest commit is a *good* one, with none of the dirty files you want removing from your history still in it. This assumption by the BFG protects your work, and gives you peace of mind knowing that the BFG is *only* changing your repo history, not meddling with the *current* files of your project.

2. 下载BFG工具jar包，[链接](http://repo1.maven.org/maven2/com/madgag/bfg/1.13.0/bfg-1.13.0.jar) 。（当然要先装java runtime）:)

3. 下载git repo的bare repository：`git clone --mirror git://example.com/some-repo.git`

   bare repository并不能看到实际的repo文件，但是却包含了repo的全部数据。下载完之后，最好备份一下。

4. 在目录下执行：`java -jar bfg.jar --delete-files filename some-repo.git`。

   - 后面这个参数为刚刚下载到本地的repo目录，不是完整的git地址。
   - filename为要删除的文件，注意，该操作会将根目录下及根目录下的子目录的相同filename的文件都删除
   - 如果想删除多个文件，filename参数可以写成`{file1,file2}`。

5. 进入到刚下好的repo目录中，执行：

   `git reflog expire --expire=now --all && git gc --prune=now --aggressive`

6. 将修改的repo push到remote：`git push`。

7. push成功之后查看remote中是否文件已删除，并且与该文件相关的commit的历史记录也删除了。



### 参考链接

- [bfg-repo-cleaner](https://rtyley.github.io/bfg-repo-cleaner/)
- [Removing sensitive data from a repository](https://help.github.com/articles/removing-sensitive-data-from-a-repository/)



下篇博客预告，总结下博客迁移的步骤吧，今天又踩了以前踩过的坑:)