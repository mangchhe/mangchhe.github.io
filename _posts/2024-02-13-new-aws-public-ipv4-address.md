---
layout: post
title: New AWS Public IPv4 Address Chrage, Public IP Insights
categories: aws
tags: aws
---

최근에 진행 중인 사이드 프로젝트에서 인프라를 담당하고 있었다. 오늘도 어김없이 AWS 계정을 점검하던 중, 예상치 못한 `0.005 per In-use public IPv4 address per hour` 라는 과금 항목을 확인했다. 이유를 알아보니 AWS에서 IPv4에 대한 정책이 변경이 된 것이었다.

### AWS Public IPv4 정책

AWS에서 2024년 2월 1일부터 모든 공개 IPv4 주소에 대해 시간당 $0.005의 요금을 부과하기 시작했다. 이는 IPv4 주소의 부족 문제와 비용 상승 때문인데, 이 새로운 요금제는 모든 AWS 서비스와 지역에 적용된다.

<p align="center">
    <img src="/assets/postImages/NewAwsPublicIpv4Address/awsPublicIpv4Price.png" width="450" height="250">
</p>

AWS free tier 는 첫 12개월 동안 매달 750시간의 IPv4 주소 사용을 포함한다. 단, **EC2 전용만 프리티어 전용이라는 것을 명심해야 한다.**

현재 AWS는 IPv6로의 전환을 장려하고 있고 사용자가 자신의 공개 IPv4 주소 사용을 더 잘 관리하고 이해할 수 있도록 "Public IP Insights"라는 새로운 도입했다.

"Public IP Insights" 기능은 사용자가 자신의 공개 IPv4 주소 사용을 모니터링하고 분석하는 데 도움을 준다. 이 기능을 통해 사용자는 IPv4 주소 사용량을 보다 효과적으로 관리하고, 필요하지 않은 주소를 식별하여 비용을 절감할 수 있다. 이는 AWS의 공개 IPv4 주소 요금제 변화와 함께 제공되어, 고객들이 IPv4 주소 사용을 최적화하고, 불필요한 비용을 줄일 수 있도록 지원한다.

> [https://aws.amazon.com/ko/blogs/aws/new-aws-public-ipv4-address-charge-public-ip-insights/](https://aws.amazon.com/ko/blogs/aws/new-aws-public-ipv4-address-charge-public-ip-insights/)