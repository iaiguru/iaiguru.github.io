---
title: Undo a git rebase.
date: 2014-02-18 19:02:00 -0300
categories: [Git]
tags: [git, rebase]     # TAG names should always be lowercase
---

The easiest way would be to find the head commit of the branch as it was immediately before the rebase started using [reflog](https://git-scm.com/docs/git-reflog).

```shell
$ git reflog
```

Then reset the current branch to the old commit. Suppose the commit was HEAD@{10} in the `ref log`.

```shell
$ git reset --hard HEAD@{10}
```

In windows, you may need to quote the reference:

```shell
$ git reset --hard "HEAD@{10}"
```





