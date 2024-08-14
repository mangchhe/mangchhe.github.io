---
layout: post
title: AWS Policy AWSCompromisedKeyQuarantineV2
categories: aws
tags: aws
---

AWS의 엑세스 키와 비밀키 노출이 되면 메일이 날라오게 되는데 다음과 같은 내용이 담겨있다.

> Your security is important to us and this exposure of your account’s IAM credentials poses a security risk to your AWS account, could lead to excessive charges from unauthorized activity, and violates the AWS Customer Agreement or other agreements with us governing your use of our Services.
> 
> To protect your account from excessive charges and unauthorized activity, **we have applied the “AWSCompromisedKeyQuarantineV2” AWS Managed Policy (“Quarantine Policy”) to the IAM User listed above.** The Quarantine Policy applied to the **User protects your account by denying access to high risk actions like iam:CreateAccessKey and ec2:RunInstances.**

IAM credentials의 노출이 AWS 계정에 보안적 이슈를 일으킬 수 있고 그로 인해 비정상적인 과금이 나올 수 있다. 그리하여 `AWSCompromisedKeyQuarantineV2` Policy를 credentials가 노출된 IAM 유저에 적용시킨다는 것이고 이에 따라 IAM 엑세스 키를 생성하거나 EC2 인스턴스를 실행하는 것과 같은 고위험 리스크를 가진 액션들을 제한한다는 것이다.

> [AWSCompromisedKeyQuarantineV2](https://docs.aws.amazon.com/ko_kr/aws-managed-policy/latest/reference/AWSCompromisedKeyQuarantineV2.html)
>
>  IAM 사용자의 자격 증명이 침해되거나 공개적으로 노출된 경우 AWS 팀에서 적용하는 특정 작업에 대한 액세스를 거부한다.

메일에는 대응 절차에 대한 안내가 포함되어 있으며, 다음 지침을 따라 조치를 취하시면 된다.

1.	노출된 키를 교체해야 하므로, 새로운 키를 생성하고 애플리케이션에 적용한 후 기존 키를 비활성화한다.
2.	CloudTrail을 사용하여 원치 않는 활동 로그(액세스 키, 정책, 역할, 비밀번호)를 확인하고, 문제가 되는 항목을 즉시 삭제한다.
3.	불필요한 AWS 리소스가 사용되고 있는지 확인한 후 이를 삭제한다.
4.	위 2~3번 항목에 해당하는 문제가 있었다면, 청구 조정 신청을 진행한다.