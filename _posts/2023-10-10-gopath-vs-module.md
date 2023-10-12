---
layout: post
title: GOPATH vs go.mod
categories: go
tags: go
---

`GOPATH` 와 `go.mod`에 대한 구분은 Go 개발자들이 프로젝트의 공간과 의존성에 대한 구조화와 관리에 중요한 영향을 미친다.

## Single Workspace - GOPATH

Go는 초기에 파일 시스템 내에 커스텀 프로젝트와 의존성 위치에 대한 공간을 제한했다. 이는 `GOPATH` 환경 변수로 정의하고 제한하고 있다. 이말은 고는 오직 해당 환경 변수가 가르키는 디렉터리 아래에서만 binary, source code를 찾는다는 것을 의미한다.

디폴트 `GOPATH` 변수는 사용자 홈 디렉터리 경로 바로 아래에서 `/go` 디렉토리를 가르킨다.

`GOPATH`는 커스텀 경로를 설정할 수 있고, 한 사용자가 하나 이상의 GOPATH를 정의할 수 있다.

`GOPATH` 디렉터리 아래에는 주요 세 디렉터리가 있다.

### src

나의 프로젝트와 설치된 의존성에 대한 소스 코드들이 존재한다.

`go get github.com/user/repo` 명령어를 실행할 때 Go tool은 지정된 위치에서 모듈을 가져온다. 그리고 `GOPATH` 아래에 resource's URL의 이름을 따서 src 디렉터리에 놓여진다.

예를 들어 "someuser" 가 소유한 repo 에서 library를 다운 받게 된다면, `/home/user/go/src/github.com/someuser/library` 에 library 소스 코드가 위치하게 된다.

### pkg

컴파일된 패키지 객체(파일)들이 존재한다. 패키지가 빌드되면 그 결과 파일은 `pkg` 디렉터리에 놓여진다. 

컴파일된 패키지 파일들은 재컴파일이 필요 없이 다른 패키지들을 직접 가져올 수 있기 때문에 컴파일 시간을 줄이는 데 도움을 준다.

### bin

애플리케이션을 실행할 수 있는 바이너리 파일들이 존재한다. 실행가능한 프로그램을 빌드할 때 그 결과로 바이너리 파일은 `bin` 디렉터리에 놓여진다.

```
/home/user/go/         <--- This is your GOPATH
├── bin/
├── pkg/
│   └── linux_amd64/
│       └── github.com/
│           └── someuser/
│               └── somelib.a    <--- Compiled dependency package
└── src/
    ├── github.com/
    │   └── someuser/
    │       └── somelib/         <--- Dependency's source code
    │           └── somelib.go
    └── myapp/
        └── main.go
```

`GOPATH`에 정의된 single workspace structure는 모든 go 코드와 의존성들이 하나의 공통 공간을 공유한다는 것을 의미한다.

해당 방식은 Go 1.11 에 Go modules이 도입되면서 발전됐다.

Go module 방식으로 파일 시스템 아래에 프로젝트를 저장하기 위해 유연성과 의존성을 위한 적절한 versioning 시스템의 부족함을 다루었다.

## Modular Approach - the `go.mod` File

Go 1.11을 시작으로 모듈 옵션은 `GOPATH`를 이용한 workspace 정의를 대체할 수 있게 되었다. 이것은 `go.mod`, `go.sum` 파일의 존재로 구분된다.

### Packages in Go

Go 패키지는 코드 배포의 가장 작은 단위이다. 하나 이상의 `.go` 파일들이 모여 정의된다. 모든 파일은 같은 패키지 이름에 정의되어야만 한다.

패키지는 코드들이 정리되고 재사용되는 것을 허락한다. 그것은 한 유닛에 관련된 코드들을 캡슐화하는 방법을 제공한다. 이는 다른 패키지에서 가져와 사용할 수 있다. 예를 들어 Go 표준 라이브러리로 `fmt`, `os`, `net`과 같은 많은 패키지를 포함한다.

`main`은 single special package name 이고 이 패키지는 프로젝트의 entry point인 `main()` 함수를 포함한다. 모든 실행 가능한 프로젝트는 `main()` 함수를 포함한다.

### Modules in Go

모듈은 single unit을 위해 함께 versioning 해야 하는 연관된 go 패키지들의 집합이다. 모듈들은 정확한 의존성 요구 조건들을 기록하고 재현가능한 빌드를 만든다.

go 모듈은 `go.mod` 파일에 의해 정의되고 모듈 디렉터리 계층에서 최상위에 위치한다. 이 파일은 모듈 내에 모든 패키지의 import path를 정의한다. 그것은 모듈의 의존성에 대해서 열거하고 다른 모듈들의 요구되는 버전들을 포함한다.

모듈들은 패키지들을 함께 release 하거나 versioning 하도록 하고, 의존성 버전 정보를 명시적으로 정의하고 쉽게 관리할 수 있게 한다.

### Naming Convertions for Modules

고에서 모듈 이름은 시스템 전반적으로 사용되기 때문에 가능한 구체적으로 작성해야한다. 특히 다른 개발자와 함께 모듈 배포를 계획한다면 모듈 이름이 구체적이어야 한다.

모듈 이름은 `go.mod` 파일에 작성되고 모듈 manifest 역할을 하며 모듈 최상위 디렉터리에 위치한다.

##### Module Path

- 모듈 경로는 모듈의 전역적으로 unique 한 식별자 역할을 한다.
- 일반적으로 모듈 경로는 인터넷 도메인 이름 뒤에 오며 모듈에서 패키지를 가져올 때 사용된다.

##### Module Name

- 모듈 경로의 마지막 구성 요소이고 짧고 간결하고 Go naming conventions를 따른다.
- 모듈 이름은 _ 없고 소문자 또는 대문자를 섞어 사용하는 것을 추천한다.

##### Versioning

- 모듈 이름은 버전 정보를 포함하지 않는다. `go,mod` 파일에 모듈의 버전 식별 정보가 `v1.2.3`와 같이 관리된다.

### The go.sum File

`go get github.com/user/repo` 명령어를 통해 모듈을 가져오고 이것은 Git, Mercurial, Subversion 과 같은 버전 관리 시스템에서 Go 코드를 호스팅할 수 있게 해준다.

Go는 외부 저장소에서 의존성을 다운받을 수 있는 유연하고 직접적인 접근 방식을 가지고 있다. 그래서 의존성의 무결성을 보장하기 위해 local file(`go.sum`)에 각 의존성에 대한 checksum을 저장하는 것을 채택한다. 이것을 통해 정확히 일치하는 의존성 설치를 보장한다.

checksum file(`go.sum`)은 Go 툴에 의해 생성되며 local과 동일한 환경을 확인하기 위해 외부 환경에 보내야 한다.

### File Tree for a modular Go Project

```
/home/user/projects/
└── myapp/                        # Your project
    ├── custom/                   # Custom packages
       └── ...                    # .go files defining functionality
    ├── go.mod                    # go.mod file defining dependencies & versions
    ├── go.sum                    # go.sum file that verifies dependency integrity
    └── main.go                   # Entrypoint
```

프로젝트는 `go.mod`, `go.sum`를 포함한다.

- go.mod : 프로젝트의 의존성의 구체적인 버전을 담고 있다.
- go.sum : 추가한 모듈의 각 의존성의 구체적인 내용에 대한 checksum을 제공한다.
- 의존성은 Go module cache에 저장되어 있고 시스템 내의 모든 프로젝트에서 공유한다.

### References

- [https://www.freecodecamp.org/news/golang-environment-gopath-vs-go-mod/](https://www.freecodecamp.org/news/golang-environment-gopath-vs-go-mod/)