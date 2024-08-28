---
title: Git checkout a remote branch
date: 2014-02-20 19:02:00 -0300
categories: [Git]
tags: [git, checkout, remote branch]     # TAG names should always be lowercase
---

One of the first Git commands you've learned ws certainly "git checkout":

```shell
$ git checkout development
```

In this simplest form, it allows you to switch(and even create) local branches -something you need countless times in your day-to-day work.

However, git checkout's power is not limited to local branches: it can also be used to create a new branch from a remote one.

## Collaborating with Branches

Remember that branches are the main way of collaboration in Git. Let's say that one of your colleagues wants you to collaborate on(or review) a piece of code:

1. She will push the corresponding branch to your common remote server.

2. In order to see this newly published branch, you will have to perform a simple "git fetch" for the remote.

3. Using the "git checkout" command, you can then create a local version of this branch - and start collaborating.

## Checking out for remote branches

The syntax for making git checkout "remote-ready" is rather easy: simply add the "--track" flag.

```shell
$ git checkout --track origin/branch-a
```

Based on the remote branch "origin/branch-a", we now have a local branch named "branch-a".

Note that, by default, Git uses the same name for the local branch. Being a good convention, there''s rarely the need to change this.