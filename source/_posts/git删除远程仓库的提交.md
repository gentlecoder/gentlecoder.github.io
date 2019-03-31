---
title: git删除远程仓库的提交
date: 2019-03-28 22:06:11
tags:
  - git
---

* git reset commitId [--soft]（注：不要带–hard）到上个版本
* git stash 暂存修改
* git push --force 强制push，远程的最新的一次commit被删除
* git stash pop 释放暂存的修改，开始修改代码
* git add . -> git commit -m "massage" -> git push