---
title: git subtree的使用
date: 2019-04-28 23:46:59
tags:
  - git
---

- `git subtree add --prefix=<子目录名> <子仓库名> <分支> --squash`

```sh
git subtree push --prefix=dist origin gh-pages
```

意思就是把指定的dist文件提交到gh-pages分支上，前提是dist在原分支中存在。

