---
layout: post
title: Register Gitlab Runner
categories: cs
tags: cs
---

#### Install Gitlab Runner

```sh
sudo curl --output /usr/local/bin/gitlab-runner "https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-darwin-arm64"
sudo chmod +x /usr/local/bin/gitlab-runner

cd ~
gitlab-runner install
gitlab-runner start
gitlab-runner register --url https://gitlab.com/ --registration-token {token}
```

> [https://docs.gitlab.com/runner/register/index.html](https://docs.gitlab.com/runner/register/index.html)
>
> [https://docs.gitlab.com/runner/install/osx.html](https://docs.gitlab.com/runner/install/osx.html)

#### Config File

```sh
# /Users/{username}/.gitlab-runner/config.toml

concurrent = 1
check_interval = 0
shutdown_timeout = 0

[session_server]
    session_timeout = 1800

[[runners]]
    name = {러너 이름}
    url = "https://gitlab.com/"
    id = {러너 ID}
    token = {토큰}
    token_obtained_at = 2023-12-08T12:37:00Z
    token_expires_at = 0001-01-01T00:00:00Z
    executor = "shell"
[runners.cache]
    MaxUploadedArchiveSize = 0
```

> [설정 파일 위치](https://docs.gitlab.com/runner/commands/#configuration-file)
> 
> UNIX 계열 운영체제에서 root 권한으로 러너를 실행했을 경우: /etc/gitlab-runner/config.toml
> 
> UNIX 계열 운영체제에서 사용자 권한으로 러너를 실행했을 경우: ~/.gitlab-runner/config.toml
> 
> 그 외 운영체제: ./config.toml

#### Verify Gitlab Runner

```sh
gitlab-runner list
gitlab-runner verify
```

![persistence_lifecycle](/assets/postImages/RegisterGitlabRunner/runnerActive.png)

#### Install Gitlab Runner In Docker

```sh
docker run -d --name gitlab-runner --restart always \
  -v {config mount 파일 경로}:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:latest
  
docker exec -it gitlab-runner bash

gitlab-runner register --url https://gitlab.com/ --registration-token {token}
```

> [https://docs.gitlab.com/runner/install/docker.html](https://docs.gitlab.com/runner/install/docker.html)