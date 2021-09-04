---
title: 【Git】 자주 쓰는 명령어 모음
decription: 형상관리 도구인 Git을 이용하면서 자주 사용했던 명령어들을 정리해보자
categories:
 - Git
tags:
 - Git
 - tool
 - 형상관리
---

> 형상관리 도구인 Git을 이용하면서 자주 사용했던 명령어들을 정리해보자

> ## 목차

- 기본 명령어
  1. git clone [Repository URL]
  1. git add .
  1. git commit -m [Commit Message]
  1. git commit --amend
  1. git push
  1. git pull
  1. git merge 브랜치명
  1. git status
  1. git log
  1. git diff
- 브랜치 관련 명령어
  1. git branch 브랜치명
  1. git branch -D 브랜치명
  1. git branch -r
  1. git branch -a
  1. git branch -t
  1. git push origin 브랜치명
  1. git push origin --delete 브랜치명
  1. git checkout 브랜치명
  1. git fetch --prune
- 그 외 유용한 명령어
  1. git reflog
  1. git checkout 해시값 or HEAD~?
  1. git reset --hard
  1. git rebase
  1. git stash list
  1. git stash apply/drop [stash 이름]
  1. git stash push/pop [stash 이름]

> ## 기본 명령어

기본적으로 알고 있어야 되고 자주 사용했던 명령어들이다

> #### git clone [Repository URL]

원격 저장소(Remote Repository)에 있는 내용을 로컬에 불러온다

> #### git add .

Stage Area에 변경된 내용들을 저장한다 .은 현재 디렉토리에 있는 모든 항목에 대해서 저장, 원하는 파일만 저장하기 위해서는 . 대신해서 파일명을 적어주면 된다

**Stage Area** : Commit 하기 전 변경 사항에 대해서 저장하는 공간

> #### git commit -m [Commit Message]

Stage Area에 저장된 내용을 의미 있는 커밋 내용과 함께 내용을 저장한다

커밋하는 이유는 업데이트 작업 별로 커밋을 하게 되면 추후 버그가 발생하기 전으로 돌아가기 쉽고 변경했던 이력들에 대해서 추적하기 쉽다

> #### git commit --amend

바로 이전에 적은 커밋 내용에 대해서 수정하려고 할때 다음 명령어를 사용한다

> #### git push

commit 했던 내용들에 대해서 모두 원격 저장소(Remote Repository)로 최신화하게 된다

> #### git pull

원격 저장소(Remote Repository)에 변경사항을 감지하고 로컬 저장소로 그 내용들을 최신화하게 된다

프로젝트를 진행할 때 다른 팀원들과 동일한 브랜치를 사용하고 있다면 내용을 push하기 전에 pull을 하는 습관을 가지는 것이 좋다

> #### git merge

서로 다른 브랜치들이 각 내용들으 합치려고 할 때 사용하는 명령어이다

merge 종류 세가지 알아보기 -> [링크](https://mangchhe.github.io/git/2021/09/04/GitMerge/)

예를 들어 브랜치가 branch1, branch2가 있을때 branch1에 branch2의 데이터를 합치고자 할때 git checkout branch1로 branch1으로 이동한 후 git merge branch2를 하게 되면 branch2의 데이터가 branch1으로 합쳐지게 된다

``` bash
git checkout branch1 # base가 되는 브랜치로 이동
git merge branch2 # 분기된 브랜치 병합

git merge --squash <브랜치명> # 해당 브랜치 수정 내용 모두 가져오기
```

> #### git status

로컬 저장소에 Stage Area에 어떤 데이터들이 있는지 push 되지 않은 commit 내용들이 얼마나 있는지에 대한 정보를 알려준다

> #### git log

현재 있는 브랜치에 커밋 내용들을 보여준다.

``` bash
git log --onefile # 각 커밋을 한 라인으로 보기 쉽게 표현함 # pretty=oneline
git log --graph # 그래프로 표현
```

> #### git diff

파일들의 변경 상태를 비교할 수 있다.

``` bash
git diff HEAD HEAD^ # 이전 커밋과 비교
git diff <해시값> <해시값2> # 비교하고 싶은 커밋메시지(해시값)
git diff <브랜치명>, <브랜치명2> # 비교하고 싶은 브랜치
```

> ## 브랜치 관련 명령어

프로젝트를 진행하면서 브랜치를 나누고 생성, 삭제하면서 많이 사용했던 명령어이다

> #### git branch 브랜치명

해당 브랜치명을 가지는 브랜치를 생성하는 명령어이다

> #### git branch -D 브랜치명

해당 브랜치명을 가지는 브랜치를 삭제하는 명령어이다

> #### git branch -r

원격 저장소에 있는 브랜치들을 조회하는 명령어이다

> #### git branch -a

원격 저장소와 로컬에 있는 브랜치들을 조회하는 명령어이다

> #### git branch -t

원격 저장소에 있는 브랜치를 로컬로 가져오는 명령어이다.

> #### git push origin 브랜치명

로컬에 있는 브랜치를 원격 저장소에 추가시키는 명령어이다

> #### git push origin --delete 브랜치명

원격 저장소에 있는 브랜치를 삭제하는 명령어이다

> #### git checkout 브랜치명

해당 브랜치명을 가지는 브랜치로 이동하는 명령어이다

> #### git fetch --prune

리모트 저장소에 있는 브랜치들의 정보를 로컬에 최신화시킨다

예를 들어 git branch -r 명령어를 사용하였을 때 이미 다른 팀원들이 삭제했던 브랜치들이 남아 있는 경우가 있는데 이럴때 사용하게 되면 브랜치들이 최신화가 되는 것을 볼 수 있다

> ## 그 외 유용한 명령어

자주 쓰지는 않지만 꼭 필요로 했던 명령어이다

> #### git reflog

log 명령어와는 다르게 rebase, commit, reset, pull 등 상세한 정보들의 내용도 기록되어 있는 것들의 목록을 보여준다

> #### git checkout 해시값 or HEAD~?

이전에 커밋한 기록 중에 해당 시점의 커밋 상태로 이동하고 싶을 때 사용한다

각 커밋 기록은 해시 값을 가지고 있어서 그 해시값을 입력하여 그때 상태로 돌아갈 수 있고 HEAD~<숫자> 를 통해서도 이동할 수 있다

HEAD는 가장 최근 커밋을 가르키며 HEAD~<숫자>와 같이 숫자를 이용하여 숫자만큼 이전 커밋을 가르킬수있게된다 HEAD^도 있는데 HEAD~1와 동일하다

> #### git reset

Stage Area에 저장된 내용을 취소하고 싶을때 사용한다 git reset HEAD 또는 해시값을 이용하여 커밋 했던 내용도 되돌릴 수 있다

되돌릴 때 --hard라는 값을 줄 수 있는데 붙이지 않을 경우 기록들은 사라지지만 변경되었던 데이터들에 대해서는 그대로 작업 공간에 남게되고 붙일 경우에는 기록과 함께 작업했던 내용도 함께 사라지게된다

> #### git rebase

커밋 내용들을 다루기 위한 명령어이다

``` bash
git rebase <브랜치명> # 해당 브랜치로 커밋 메시지가 재배치된다.
git rebase -i HEAD~?
```

-i 옵션을 붙이게 되면 아래 사진과 같이 편집 상태로 들어가게 되고 원하는 명령어를 붙여서 커밋 내용들을 수정할 수 있다.

![gitRebase](/assets/gitRebase.PNG)

위 사진은 명령어 뒤에 HEAD~1을 붙였고 만약 HEAD~3와 같이 붙이게 된다면 이전 커밋 내용 세개가 나타나게 될 것이다

아래 commands를 보게 되면 여러가지 작업을 할 수 있고 위에 초록색 pick 부분을 지우고 삭제하고 싶다면 drop을 적고 커밋 내용을 수정하고 싶다면 reword를 적고 :wq를 하여 저장하고 나오면 하고자했던 처리를 하게 될 것이다

해당된 작업이 끝나고 내용을 완료하려고 한다면 `git rebase --continue` 취소하려고 한다면 `git rebase --abort`를 입력하면 된다

> #### git stash

working directory에서 작업하던 내용들을 임시로 저장할 수 있게 해주는 명령어이다

> #### git stash list

임시로 저장된 내용들에 대한 기록을 볼 수 있다

> #### git stash apply/drop [stash 이름]

저장된 내용에 대해서 다시 가져오거나(apply), 삭제(drop)시킬 수 있다

> #### git stash push/pop [stash 이름]

현재 working directory에 있는 내용을 임시 저장(push)하거나 가져올 수 있다(pop)

push/pop을 많이 사용했던 부분은 잘못된 브랜치에서 작업을 하고 있을 경우 변경된 파일들을 옮기기 위해서 사용했다

예를 들어 branch1에서 작업해야 되는데 branch2에서 작업했다면 branch2에서 `git stash push`를 한다음 `git checkout branch1`해서 브랜치를 변경한 후 `git stash pop`을 하게되면 했던 내용들이 손쉽게 변경되는 것을 확인할 수 있다
