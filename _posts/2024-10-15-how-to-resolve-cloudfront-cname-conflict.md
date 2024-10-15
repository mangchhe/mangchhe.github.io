---
layout: post
title: How to Resolve CloudFront CNAME Conflict
categories: devops
tags: devops
---

CloudFront에서 CNAME 별칭을 설정할 때, 이미 다른 배포에 동일한 CNAME이 설정되어 있다면 `CNAMEAlreadyExists` 오류가 발생할 수 있다.

나 역시 인프라를 이관하는 과정에서 CloudFront를 생성 과정에서 `One or more of the CNAMEs you provided are already associated with a different resource.` 라는 에러를 만나게 되었다.

> [https://repost.aws/en/knowledge-center/resolve-cnamealreadyexists-error](https://repost.aws/en/knowledge-center/resolve-cnamealreadyexists-error)

일반적으로 해당 배포가 본인 계정이라고 하면 해당 가이드를 따라하여 타계정의 배포를 이동시킬 수 있다.

```sh
_.example.com.         900   IN   TXT     "dexample123456.cloudfront.net"
_cname.example.com.    900   IN   TXT     "dexample123456.cloudfront.net"
_*.example.com.        900   IN   TXT     "dexample123456.cloudfront.net"

$ aws cloudfront associate-alias --target-distribution-id YourTargeDistributiontID --alias your_cname.example.com
```

하지만 기존 배포가 본인 계정이 아니라고 하면 상황은 다르다. 보통 아래와 같이 기존 배포가 활성화되어 있다는 에러를 만나게 될 가능성이 높다.

`An error occurred (IllegalUpdate) when calling the AssociateAlias operation: Alias move is not allowed since the source distribution is enabled.`

`list-conflicting-aliases` 명령어로 충돌하는 배포를 확인했을 때, 해당 배포가 다른 계정에 있다면 직접 비활성화할 방법이 없으므로, AWS Support에 하루라도 빨리 문의하여 CNAME을 이동할 수 있도록 요청하는 것이 좋다. 나 역시 이 방법으로 문제를 해결할 수 있었다.

```sh
$ aws cloudfront list-conflicting-aliases --distribution-id YourDistributionID --alias YourCNAME

"ConflictingAliasesList": {
    "MaxItems": 100,
    "Quantity": 1,
    "Items": [
        {
            "Alias": "example.com",
            "DistributionId": "*******4ABC123D",
            "AccountId": "******123456"
        }
    ]
}
```

> After you deactivate the source distribution, follow the steps in the Use the AssociateAlias API to move your CNAME section.
> 
> If you don't have access to the account that contains the source distribution or you can't deactivate the source distribution, then contact AWS Support.
