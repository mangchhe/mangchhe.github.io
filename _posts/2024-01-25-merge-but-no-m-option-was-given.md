---
layout: post
title: 'Error: commit {hash} is a merge but no -m option was given.'
categories: git
tags: git
---

```sh
error: commit 44a074a36610dfb85f733ff45356c18b15c8f3c6 is a merge but no -m option was given.
fatal: revert failed
```

revert를 하려는데 다음과 같은 에러를 만났다.

> [git revert -m](https://git-scm.com/docs/git-revert#Documentation/git-revert.txt--mparent-number)
>
> Usually you cannot revert a merge because you do not know which side of the merge should be considered the mainline. This option specifies the parent number (starting from 1) of the mainline and allows revert to reverse the change relative to the specified parent.

공식 문서에 다음과 같이 mainline을 알지 못하는 경우의 머지는 revert를 할 수 없다고 나와 있다. `-m` 옵션을 통해 mainline을 지정하여 문제를 해결할 수 있다.

로그를 그래프로 확인하면 현재 `44a074a` 커밋의 부모는 순서대로 `975dc09`(1), `7d4be38`(2)이다. 

```sh
$ git log --oneline --graph
*   44a074a (HEAD -> main) Merge branch 'feature/1004'
|\
| * 975dc09 (feature/1004) update(test.txt): add text
| * e0bd3c1 add: test.txt
|/
* 7d4be38 add: 카카오페이 온라인 결제 서비스 2.5배 성능 개선기
```

`git log` 또는 `git cat-file -p`를 통해 좀 더 자세히 알 수 있다.

```sh
$ git log
commit 44a074a36610dfb85f733ff45356c18b15c8f3c6
Merge: 7d4be38 975dc09
Author: joohyun.ha <mylifeforcoding@gmail.com>
Date:   Thu Jan 25 01:12:24 2024 +0900

    Merge branch 'feature/1004'

$ git cat-file -p 44a074a
tree c1e26b3dd1d51d4fdedf3a927d189715069544b9
parent 7d4be38d56ea58b72ab0faafc99795461ee9d654
parent 975dc092bf9a95e0831411100a90bf9a68f3037a
```

위 로그를 참고하여 mainline을 현재 main 브랜치의 parent를 지정하면 정상적으로 revert가 수행된 것을 알 수 있다.

```sh
git revert -m 1 44a074a
* b397ca0 (HEAD -> main) Revert "Merge branch 'feature/1004'"
*   44a074a Merge branch 'feature/1004'
|\
| * 975dc09 (feature/1004) update(test.txt): add text
| * e0bd3c1 add: test.txt
|/
* 7d4be38 add: 카카오페이 온라인 결제 서비스 2.5배 성능 개선기
```