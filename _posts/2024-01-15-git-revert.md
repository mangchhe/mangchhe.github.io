---
layout: post
title: Git Revert Command
categories: git
tags: git
---

```sh
git revert [--[no-]edit] [-n] [-m <parent-number>] [-s] [-S[<keyid>]] <commit>…​
git revert (--continue | --skip | --abort | --quit)
```

> Given one or more existing commits, revert the changes that the related patches introduce, and record some new commits that record them. This requires your working tree to be clean (no modifications from the HEAD commit).

```sh
touch test.txt
git add test.txt
git commit -m "add: test" # d6c473f7

touch test2.txt
git add test2.txt
git commit -m "add: test2" # decab75d

touch test3.txt
git add test3.txt
git commit -m "add: test3" # 9f3cbf60

git log --oneline
# 9f3cbf609 add: test3
# decab75d6 add: test2
# d6c473f71 add: test

git revert decab75d
# cebca5c04 Revert "add: test2"
# 9f3cbf609 add: test3
# decab75d6 add: test2
# d6c473f71 add: test
```

## References

- [https://git-scm.com/docs/git-revert](https://git-scm.com/docs/git-revert)