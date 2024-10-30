---
layout: post
title: Cross-Account S3 Bucket Data Migration Guide
categories: devops
tags: devops
---

`source` → `target` 버킷으로 데이터를 이동하는 방법.

<p align="center">
    <img src="/assets/postImages/S3BucketDataMigrationGuide/diagram.png" width="350">
</p>

##### Target IAM Policy

Target 버킷이 있는 계정에서 해당 정책을 생성한 후 특정 IAM 계정에 해당 정책 권한을 부여한다.

- target 버킷에 대한 읽기 권한 부여
- source 버킷에 대한 쓰기 권한 부여

```yml
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket",
                "s3:GetObject"
            ],
            "Resource": [
                "arn:aws:s3:::{target-bucket-name}",
                "arn:aws:s3:::{target-bucket-name}/*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket",
                "s3:PutObject",
                "s3:PutObjectAcl"
            ],
            "Resource": [
                "arn:aws:s3:::{source-bucket-name}",
                "arn:aws:s3:::{source-bucket-name}/*"
            ]
        }
    ]
}
```

##### Source S3 Bucket Policy

Source 버킷이 있는 계정에서 해당 정책을 버킷에 부여한다.

- target 유저가 source 버킷에 대한 읽기 권한 부여

```yml
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "DelegateS3Access",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::{target-account-id}:user/{target-iam-username}"
            },
            "Action": [
                "s3:ListBucket",
                "s3:GetObject"
            ],
            "Resource": [
                "arn:aws:s3:::{source-bucket-name}/*",
                "arn:aws:s3:::{source-bucket-name}"
            ]
        }
    ]
}
```

##### Command

```sh
$ aws --profile {target-profile} s3 sync s3://{source-bucket-name} s3://{target-bucket-name}
```