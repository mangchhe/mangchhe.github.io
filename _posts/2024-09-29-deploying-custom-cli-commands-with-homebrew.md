---
layout: post
title: Deploying Custom CLI Commands with Homebrew
categories: tool
tags: tool
---

> Sample
> 
> [https://github.com/mangchhe/homebrew-plot](https://github.com/mangchhe/homebrew-plot)

**작업 순서**

1.	GitHub Repository 생성
2.	CLI 스크립트 작성 및 압축
3.	GitHub Release 생성 및 업로드
4.	Homebrew Formula 파일 생성 후 커밋
5.	Tap을 이용해 설치

**GitHub Repository 생성**

`homebrew-<repository name>` 형식으로 생성

**스크립트 제작 및 압축**

```sh
tar -czvf <command_name>-0.0.1.tar.gz <command_name>.sh
```

**Formula 파일 생성**

```sh
class <command_name> < Formula
  desc "설명"
  homepage "https://github.com/<your-username>/<repository name>"
  url "https://github.com/<your-username>/<repository name>/releases/download/v1.0.0/<command_name>-0.0.1.tar.gz"
  sha256 "tar.gz 파일의 SHA256 해시값"

  depends_on "awscli"

  def install
    bin.install "<command_name>.sh" => <command_name>
  end

  test do
    system "#{bin}/<command_name>", "--help"
  end
end
```

SHA256 값 얻기:

```sh
shasum -a 256 <command_name>-0.0.1.tar.gz
```

**tap을 이용해 repository를 추가 및 설치**

```sh
brew tap <username>/<repository_name>
brew install <file_name>
```