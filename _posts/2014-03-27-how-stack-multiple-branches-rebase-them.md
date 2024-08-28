---
title: How to stack multiple git branches and rebase them
date: 2014-03-27 19:02:00 -0300
categories: [Git]
tags: [git, rebase, gitflow]     # TAG names should always be lowercase
---

It took me quite some time to start feeling comfortable with git, and although I still know I have a lot to learn I finally feel confident not to screw up.

I see some people constantly struggle with:

- How to rebase your branch on top of master in the most efficient way.
- How to stack up branches on top of each other and keep them all updated using rebase.

Note: All the tips are assuming you use a rebase workflow and not a merge workflow. I recommend the use of the rebase workflow, however, that's a discussion for another post :)

## 1. How to rebase your branch on top of master in the most efficient way.

a lot of people doing:

- git checkout master
- git pull
- git checkout branch_to_rebase
- git rebase master

You can optimize this flow by running the rebase directly against your remote master branch:

- git fetch origin - it will retrieve the latest meta-data info from your remote, but unlike git pull it will not transfer any file.
- git rebase origin/master

## 2. How to stack up branches on top of each other and keep them all updated using rebase

If you have only one branch, the previous way is enough to make sure your branch is always updated with the latest changes in master.

The problem starts when you have multiple branches depending on each other.

Let's imagine the following situation:
- You create a new branch to start working on feature A.
- You finish all the work and open a Pull Request for your team to review.
- Since you don't want to be blocked, you start working on feature B that depends on feature A, so you create a new branch-B on top of branch-A.
- In the meanwhile, you rebase and update branch-A to fix all the PR comments.
- You go back to branch-B and you want to update it with the latest changes in branch-A.
- You try "git rebase branch-A" and git complains about a lot of nonsense conflicts.

So, what happened there?

Basically, when you updated branch-A you might have changed some of the commit hashes from that branch, but branch-B isn't aware of them yet. Thus, when you rebase the branch-B on top pf branch-A, git will think the old commits from branch-A are new and part of branch-B, so it might create conflicts with themself. I like to call them: "repeated commits".

This is the number 1 criticized aspect of the superior rebase workflow - the fact that it messes with git history and some tools, including git itself, get confused by that. But honestly, if you understand how git works, that will never be a problem, quite the opposite, it will make your git history as clear as never.

There is more than one way to fix this, but there is a neat one-line way of solving this:
- From branch B run "git rebase -i branch-A" this will make the rebase interactive and editable.
- Drop all the old/repeated commits from branch-A present on branch-B. You have to make sure only commits from branch-B will be rebased.

That's it! No more nonsense conflicts. You might still have conflicts, but those you really need to fix :)

Thank you for reading!
