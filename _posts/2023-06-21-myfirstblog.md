---
title: Running jekyll server is not working on my Apple M2 device.
date: 2023-06-21 23:05:00 -0300
categories: [Jekyll]
tags: [jekyll, local server, apple m2]     # TAG names should always be lowercase
---

You may want to preview the site content before publishing, so just run it by:

```shell
$ bundle exec jekyll s
```

However, this command didn't work on the Apple M2 devices.

Here is a workaround I found:

```shell
$ rm -rf vendor
$ arch -arch x86_64 bundle install
$ arch -arch x86_64 bundle exec jekyll serve
```
