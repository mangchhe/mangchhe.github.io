---
layout: post
title: Git Stash Command
categories: git
tags: git
---

## SYNOPSIS

```bash
git stash list [<log-options>]
git stash show [-u | --include-untracked | --only-untracked] [<diff-options>] [<stash>]
git stash drop [-q | --quiet] [<stash>]
git stash pop [--index] [-q | --quiet] [<stash>]
git stash apply [--index] [-q | --quiet] [<stash>]
git stash branch <branchname> [<stash>]
git stash [push [-p | --patch] [-S | --staged] [-k | --[no-]keep-index] [-q | --quiet]
	     [-u | --include-untracked] [-a | --all] [(-m | --message) <message>]
	     [--pathspec-from-file=<file> [--pathspec-file-nul]]
	     [--] [<pathspec>…​]]
git stash save [-p | --patch] [-S | --staged] [-k | --[no-]keep-index] [-q | --quiet]
	     [-u | --include-untracked] [-a | --all] [<message>]
git stash clear
git stash create [<message>]
git stash store [(-m | --message) <message>] [-q | --quiet] <commit>
```

## DESCRIPTION

`git stash`를 사용하면 작업 디렉터리의 현재 상태를 저장한다. 해당 커맨드는 로컬 수정사항을 저장하고 `HEAD` commit 으로 되돌아간다.

앞서 말한 커맨드로 stash 된 수정사항 리스트를 `git stash list`로 확인할 수 있다. `git stash show`로 수정사항을 검사할 수 있고 `git stash apply`로 수정사항을 복원할 수 있다. 
인자 없이 `git stash`를 사용하면 `git stash push`와 동일하다. stash 리스트는 기본적으로 `WIP on branchname` 으로 표시된다. 예로 `stash@{0}: WIP on develop`와 같다.

생성한 가장 최근 stash는 `refs/stash`에 저장된다. 더 오래된 stash들은 해당 참조에서 `reflog`로 찾을 수 있다. 그리고 일반적인 `reflog` 문법을 통해 이름을 지정할 수 있다. (`stash@{0}` 는 가장 최근 stash이고 `stash@{1}`은 이전 stash이며 `stash@{2.hours.ago}`도 가능하다.)
stash들은 index를 구체적으로 지정함으로써 참조할 수 있다. (integer `n`은 `stash@{n}`와 동일하다.)

## COMMANDSS

**push**

로컬 수정사항을 새로운 stash entry에 저장한다. 그리고 HEAD로 되돌린다. push는 생략할 수 있다.

**save**

`git stash push`로 인해서 deprecated 된 커맨드이다.

**list**

현재 가지고 있는 stash entry의 리스트를 보여준다.

```bash
stash@{0}: WIP on submit: 6ebd0e2... Update git-stash documentation
stash@{1}: On master: 9cc0589... Add git-stash
```

**show**

stash entry를 만들었을 때와 만들기 전 상태의 차이점을 보여준다.

**pop**

stash 목록에서 가장 최근 stash를 제거하고 현재 작업 트리에 적용한다. 역연산으로 `git stash push`가 사용된다.

**apply**

`pop`과 유사하다. 하지만 그건 stash 목록에서 제거하지 않는다. 그리고 무조건 최신 stash를 꺼내오는 `pop`과 달리 `<stash>`를 지정하여 원하는 stash를 지정해서 적용할 수 있다.


### Reference

- [Git Stash 공식 문서](https://git-scm.com/docs/git-stash)