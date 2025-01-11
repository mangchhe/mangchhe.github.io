---
layout: post
title: Pre-Signed URL
categories: cs
tags: cs, devops
---

> [미리 서명된 URL을 통해 객체 공유](https://docs.aws.amazon.com/ko_kr/AmazonS3/latest/userguide/PresignedUrlUploadObject.html)
>
> 미리 서명된 URL을 사용하여 다른 사람이 Amazon S3 버킷에 객체를 업로드하도록 허용할 수 있습니다.

Pre-Signed URL을 사용하면 클라이언트에서 서버를 거치지 않고 S3로 파일을 직접 업로드할 수 있어 서버의 부하를 줄일 수 있다.

![sequence](/assets/postImages/PreSignedUrl/sequence.png)

이 방식의 핵심은 클라이언트가 Pre-Signed URL을 요청하면 서버에서 임의의 식별 키(fileKey)를 생성하여 데이터베이스에 PENDING(임시) 상태로 저장하고, Pre-Signed URL을 반환하는 것이다. 이후 클라이언트가 S3에 파일을 업로드하고 다시 서버로 확인 요청을 보내면, 서버는 해당 파일의 상태를 CONFIRMED(확정)로 업데이트한다.

추가로, 업로드되지 않은 PENDING 상태의 파일들은 배치 작업을 통해 주기적으로 정리한다. 이를 통해 S3와 데이터베이스 간의 데이터 무결성을 유지하고, 불필요한 데이터를 제거함으로써 효율적으로 파일을 관리할 수 있다. 이러한 설계는 대량의 파일 업로드 처리 시 서버 자원의 소모를 줄이고, **데이터 관리의 안정성과 효율성을 동시에 확보할 수 있는 방법**이다.