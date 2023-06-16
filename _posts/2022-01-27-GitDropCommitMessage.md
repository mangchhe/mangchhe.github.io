---
title: "Remove git commit message using rebase command"
categories: git
tags: git
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

![gitLog](/assets/postImages/GitDropCommitMessage/gitLog.png){: width="600" }

## 삭제

- test3.txt를 추가했던 커밋을 삭제하려고 한다.

```bash
git rebase -i HEAD~4
```

![dropCommit](/assets/postImages/GitDropCommitMessage/dropCommit.png){: width="600" }

- 삭제하려는 커밋 해시값 옆에 drop으로 변경해주고 저장하고 파일을 나온다.

![gitLog2](/assets/postImages/GitDropCommitMessage/gitLog2.png){: width="600" }

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

![gitLog3](/assets/postImages/GitDropCommitMessage/gitLog3.png){: width="600" }

## 삭제

- test1.txt 파일 내용을 test1으로 변경했던 커밋을 삭제하려고 한다.

```bash
git rebase -i HEAD~4
```

![dropCommit2](/assets/postImages/GitDropCommitMessage/dropCommit2.png){: width="600" }

- 삭제하려는 커밋 해시값 옆에 drop으로 변경해주고 저장하고 파일을 나온다.

![conflict](/assets/postImages/GitDropCommitMessage/conflict.png){: width="600" }

- 이전과 다르게 CONFLICT 에러가 발생했다고 경고창이 뜨게 된다.

![fileContent](/assets/postImages/GitDropCommitMessage/fileContent.png){: width="600" }

- 충돌이 일어난 파일에 들어가 수정 후에 새로운 커밋을 만든다.

```bash
git add .
git commit -m "solve conflict"
```

![gitLog4](/assets/postImages/GitDropCommitMessage/gitLog4.png){: width="600" }

- 충돌이 일어난 부분은 커밋 메시지가 새로 만들어져 들어가고 그 외에는 그대로 메시지가 유지되어있는 것을 확인할 수 있다.