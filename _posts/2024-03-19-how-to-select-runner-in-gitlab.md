---
layout: post
title: How to select Runner in Gitlab?
categories: etc
tags: etc
---

> [Runner Tags](https://docs.gitlab.com/runner/#tags)
> 
> When you register a runner, you can add tags to it.
> 
> When a CI/CD job runs, it knows which runner to use by looking at the assigned tags. Tags are the only way to filter the list of available runners for a job.

러너를 등록할 때 러너에 태그를 추가할 수 있다. 워크플로우에서 job을 실행시킬 때 할당된 태그를 확인하여 어떤 러너를 사용할 지 알 수 있다. 태그는 job에 사용 가능한 러너 목록을 필터링하는 유일한 방법이다.

> [CI/CD YAML syntax tags](https://docs.gitlab.com/ee/ci/yaml/index.html#tags)
>
> Use tags to select a specific runner from the list of all runners that are available for the project.

태그를 사용하여 프로젝트에서 사용할 수 있는 모든 러너 목록 중 특정 러너를 선택할 수 있다. 

```sh
job:
  tags:
    - {runner tag}
```

태그 설정한 러너가 태그 설정된 job에서만 선택되어 동작할 수 있게 하려면 러너 설정에서 `Indicates whether this runner can pick jobs without tags` 옵션을 해제해야 한다.

![runUnstaggedJobs](/assets/postImages/HowToSelectRunnerInGitlab/runUnstaggedJobs.PNG)