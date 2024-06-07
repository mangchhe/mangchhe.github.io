---
layout: post
title: Git Stash Commits
categories: git
tags: git
---

Git에서 rebase와 stash를 사용하여 커밋을 합치면 이력을 깔끔하게 정리할 수 있다.

해당 방법은 리모트 브랜치의 커밋 해시값을 변경하기 때문에 주로 로컬 또는 개인 브랜치에서 사용하는 것을 권장한다. 만약 공동으로 사용하는 브랜치에서 사용해야 한다면 변경 사항을 병합한 후 팀원들이 브랜치를 새로 받아 싱크를 맞춰야 한다.

#### 초기 커밋 로그

```sh
$ git log

commit 013f24839c25e0dc249fae5d76290bf364ec6cf1 (HEAD -> main)
Author: joohyun.ha <mylifeforcoding@gmail.com>
Date:   Sat Jun 8 02:24:51 2024 +0900

    test4

commit 444fcbdccb5fa776487cb87832b1c322ae6fa4b5
Author: joohyun.ha <mylifeforcoding@gmail.com>
Date:   Sat Jun 8 02:24:44 2024 +0900

    test3

commit 98ee7d08852a0e143d09f3dd609c697e6c4e33b4
Author: joohyun.ha <mylifeforcoding@gmail.com>
Date:   Sat Jun 8 02:24:36 2024 +0900

    test2

commit 5f2fcbaaa1710125a062d63f3dbb9943cf4eb1b4
Author: joohyun.ha <mylifeforcoding@gmail.com>
Date:   Sat Jun 8 02:24:30 2024 +0900

    test
```

#### 리베이스

합칠 커밋 범위만큼 HEAD 기준으로 가져와 리베이스를 시작한다.

```sh
$ git rebase -i HEAD~4
```

가장 오래된 커밋은 그대로 두고 합칠 커밋들을 `pick`을 `s` or `squah`로 변경한다.

```sh
pick 5f2fcba test
squash 98ee7d0 test2
squash 444fcbd test3
squash 013f248 test4
```

만약 첫 번째 커밋 `pick`으로 안 하고 합치려고 하는 경우 `error: cannot 'squash' without a previous commit` 에러가 발생하니 항상 첫 번째 커밋이 `pick`으로 설정되어 있어야 한다.

#### 최종 커밋

변경 내용을 저장하고 리베이스를 마무리한다.

```sh
$ git rebase --continue
```

리베이스가 완료되면 새로운 커밋이 생성된 것을 확인할 수 있다.

```sh
$ git log

commit e36e89c48d2d9c3aabb932a2ce75365e001f173c (HEAD -> main)
Author: joohyun.ha <mylifeforcoding@gmail.com>
Date:   Sat Jun 8 02:24:30 2024 +0900

    test

    test2

    test3

    test4
```