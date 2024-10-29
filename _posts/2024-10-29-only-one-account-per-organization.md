---
layout: post
title: 'AWS Free Tier: Only One Account per Organization'
categories: devops
tags: devops
---

> [https://aws.amazon.com/ko/free/free-tier-faqs/](https://aws.amazon.com/ko/free/free-tier-faqs/)
>
> AWS Organizations의 조직에 연결되어 있는 경우, 조직 내 하나의 계정만 프리 티어 오퍼의 혜택을 누릴 수 있습니다.

AWS 계정에서 예상치 못한 비용이 청구되어 설정 오류라고 생각했지만, 원인은 AWS Organizations을 사용한 것이었다. 

AWS Organizations에 연결된 경우 조직 내 한 계정만 프리 티어 혜택을 받을 수 있기 때문에, 나머지 계정에서 사용된 리소스는 과금 대상이 되었다..

오늘도 돈을 내며 배우는 즐거운 인프라의 세계 🥹

<p align="center">
    <img src="/assets/postImages/OnlyOneAccountPerOrganization/charge.png" width="350">
</p>

> [Free Tier in AWS Organizations Sub-Account](https://repost.aws/questions/QUBTMYiaJwS-utI527pmLDkw/free-tier-in-aws-organizations-sub-account)