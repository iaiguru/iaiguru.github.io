---
title: Use multiple SSH keys for different GitHub accounts.
date: 2021-11-20 18:07:00 -0300
categories: [GitHub]
tags: [github, ssh, multiple accounts]     # TAG names should always be lowercase
---

## Create different public key

Follow GitHub document and create a ssh key.

Start the ssh-agent in the background.

```
$ eval "$(ssh-agent -s)"
```

Then, add the key as following.

```
$ ssh-add ~/.ssh/ssh-key-file
```

You can check your saved keys

```
$ ssh-add -l
```

## Modify the ssh config

Create the `~/.ssh/config` file.

Then add as following

```
#jstfun account
Host github.com-jstfun
HostName github.com
User git
IdentityFile ~/.ssh/jstfun
```

## Clone your repo and modify your ssh repo URL

Tell the GitHub to not fetch the repo from `github.com` but from `github.com-jstfun`. 

There is no impact on the github URL but tell our computer that the host for the repo is connected to the `jstfun` ssh key. A `git clone` command should be like this:

```
before: git clone git@github.com:jstfun/project.git
after:  git clone git@github.com-jstfun:jstfun/project.git 
```

If you already have fetched the repo on your local machine, you can change the remote URL properly.

## Check later

Simply, set the config as following:

```
Host github.com
HostName github.com
User your_user_account_github
PreferredAuthentications publickey
IdentityFile ~/.ssh/your_user_account_github_rsa
IdentitiesOnly yes
```