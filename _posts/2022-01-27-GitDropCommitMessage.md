---
title: "[Git] Rebase 명령어로 커밋 메시지 삭제하기"
description: 커밋 메시지를 삭제하려고 할 때 충돌 나는 경우와 그렇지 않은 경우를 실습
categories:
 - Git
tags:
 - Git
 - Rebase
 - Commit
---

# 충돌 X

<hr>

## 커밋

- 서로 겹치지 않는 네 개의 파일을 만들어 각각 커밋한다.

```bash
touch test1.txt
git add .
git commit -m 'Add test1.txt'
touch test2.txt
git add .
git commit -m 'Add test2.txt'
touch test3.txt
git add .
git commit -m 'Add test3.txt'
touch test4.txt
git add .
git commit -m 'Add test4.txt'
```

<img width="600" alt="gitLog" src="https://user-images.githubusercontent.com/50051656/151368048-bd35afd8-b22b-4924-8c2c-9a17a360a815.png">

## 삭제

- test3.txt를 추가했던 커밋을 삭제하려고 한다.

```bash
git rebase -i HEAD~4
```

<img width="600" alt="dropCommit" src="https://user-images.githubusercontent.com/50051656/151368120-a183bcfa-759f-4263-8805-65fa79e0d785.png">

- 삭제하려는 커밋 해시값 옆에 drop으로 변경해주고 저장하고 파일을 나온다.

<img width="600" alt="gitLog2" src="https://user-images.githubusercontent.com/50051656/151368165-616beaf1-88cf-4b13-b2e0-bcfe5cca7589.png">

- 삭제하려는 커밋만 깔끔하게 삭제된 것을 확인할 수 있다.

# 충돌 O

<hr>

## 커밋

- test1.txt 파일을 생성한 뒤 빈 내용을 test1 -> test3으로 두 번을 걸쳐 변경 후 커밋을 하고 추가로 test2.txt 파일을 생성한다.

```bash
touch test1.txt
git add .
git commit -m 'Add test1.txt'
echo "test1" > test1.txt
git add .
git commit -m 'Update test1.txt'
echo "test3" > test1.txt
git add .
git commit -m 'Update test1.txt'
touch test2.txt
git add .
git commit -m 'Add test2.txt'
```

<img width="600" alt="gitLog3" src="https://user-images.githubusercontent.com/50051656/151368410-67c1d3b3-3083-492c-8645-86a03a143fb2.png">

## 삭제

```bash
git rebase -i HEAD~4
```

<img width="600" alt="dropCommit2" src="https://user-images.githubusercontent.com/50051656/151369007-bfcd1f6b-4360-4b46-8701-9f6c8f4d4179.png">

- 삭제하려는 커밋 해시값 옆에 drop으로 변경해주고 저장하고 파일을 나온다.

<img width="600" alt="conflict" src="https://user-images.githubusercontent.com/50051656/151369183-5a4847b0-60ae-49ba-855c-ae0b0fa67958.png">

- 이전과 다르게 CONFLICT 에러가 발생했다고 경고창이 뜨게 된다.

<img width="600" alt="fileContent" src="https://user-images.githubusercontent.com/50051656/151369277-ecfff0a0-5201-458e-9178-30c14ca0d439.png">

- 충돌이 일어난 파일에 들어가 수정 후에 새로운 커밋을 만든다.

```bash
git add .
git commit -m "solve conflict"
```

<img width="600" alt="gitLog4" src="https://user-images.githubusercontent.com/50051656/151369352-76e1cd64-1489-4145-baa5-bda87cb1c3ce.png">

- 충돌이 일어난 부분은 커밋 메시지가 새로 만들어져 들어가고 그 외에는 그대로 메시지가 유지되어있는 것을 확인할 수 있다.