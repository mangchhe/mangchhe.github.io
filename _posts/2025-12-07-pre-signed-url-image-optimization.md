---
layout: post
title: Optimize presigned URL image uploads with Lambda
categories: cs
tags: cs aws s3 lambda
---

> 원본 이미지를 그대로 S3에 올려서 쓰다가 **S3 비용 + 응답 속도**가 부담돼서 개선한 기록.

---

#### 문제 정리

- 프론트에서 **Presigned URL**로 이미지 업로드
- 업로드된 이미지를 그대로 노출 (JPG, PNG, 10MB 넘어가는 것도 있음)
- 그러다 보니:
  - **S3 저장 용량**이 커지고
  - **CloudFront/S3 전송 비용**도 커지고
  - 리스트 화면 같은 데서 **응답 속도도 느려짐**

**업로드 경험은 그대로 유지하면서,
저장/전송은 더 가볍게** 만드는 게 목표였다.

---

#### 구조/흐름

![preSignedUrlImageOptimization](/assets/postImages/PreSignedUrlImageOptimization/pre-signed-url-image-optimization.png)

이미지 업로드 플로우는 그대로 두고,
**S3 이벤트 + Lambda + Sharp**로 뒤에서 자동으로 리사이즈/포맷 변환을 돌렸다.

- 클라이언트
  - 기존처럼 `users/...` 경로로 Presigned URL 받아서 업로드
- S3
  - `users/` prefix에 객체 생성 → **S3 ObjectCreated 이벤트 발생**
- Lambda (`ImagesVariantsLambda`)
  - 이벤트를 받아서 S3에서 원본을 가져온 뒤,
  - `sharp`로 **회전 보정 + 가로 최대 `MAX_WIDTH` + WebP 변환**
  - 결과를 `images/variants/...` 아래에 저장
 - 서비스
  - 클라이언트에서 이미지를 쓸 때는  
    `images/variants/...webp` 경로를 사용

---

#### Lambda 구현

핵심 아이디어만 정리하면:

- **SOURCE_PREFIX** (`users/`)에 올라오는 것만 처리
- **DEST_PREFIX** (`images/variants/`)에 WebP 한 장 생성
- 동일 키에 대한 **중복 변환 방지(headObject로 체크)**
- EXIF 회전 보정, 가로 최대 폭 제한, WebP 품질 조절

```js
const AWS = require('aws-sdk');
const s3 = new AWS.S3({ signatureVersion: 'v4' });
const sharp = require('sharp');

const env = () => ({
  BUCKET_NAME: process.env.BUCKET_NAME,
  SOURCE_PREFIX: (process.env.SOURCE_PREFIX || '').replace(/^\/+|\/+$/g, '') + '/',
  DEST_PREFIX: (process.env.DEST_PREFIX || '').replace(/^\/+|\/+$/g, '') + '/',
  WEBP_QUALITY: parseInt(process.env.WEBP_QUALITY || '80', 10),
  MAX_WIDTH: parseInt(process.env.MAX_WIDTH || '1280', 10),
});

const stripExtension = (key) => key.replace(/\.[^.\/]+$/, '');

exports.handler = async (event) => {
  const cfg = env();

  for (const rec of (event.Records || [])) {
    const bucket = rec.s3.bucket.name;
    const key = decodeURIComponent(rec.s3.object.key.replace(/\+/g, ' '));

    // 1) prefix/bucket 검증
    if (!key.startsWith(cfg.SOURCE_PREFIX) || bucket !== cfg.BUCKET_NAME) {
      console.log('skip:', bucket, key);
      continue;
    }

    console.log(`processing s3://${bucket}/${key}`);

    // 2) 원본 이미지 가져오기
    const obj = await s3.getObject({ Bucket: bucket, Key: key }).promise();
    const inputBuf = obj.Body;

    const baseNoExt = stripExtension(key);
    const outKey = `${cfg.DEST_PREFIX}${baseNoExt}.webp`;

    // 3) 이미 변환된 파일이 있으면 skip (idempotent)
    try {
      await s3.headObject({ Bucket: bucket, Key: outKey }).promise();
      console.log(`exists, skip: ${outKey}`);
      continue;
    } catch (_) {}

    // 4) sharp로 리사이즈 + webp 변환
    const baseImage = sharp(inputBuf, { failOn: 'none' }).rotate(); // EXIF 회전 보정
    const buf = await baseImage
      .resize({ width: cfg.MAX_WIDTH, withoutEnlargement: true })
      .webp({ quality: cfg.WEBP_QUALITY })
      .toBuffer();

    // 5) 결과 업로드
    await s3.putObject({
      Bucket: bucket,
      Key: outKey,
      Body: buf,
      ContentType: 'image/webp',
      CacheControl: 'public, max-age=31536000, immutable',
      Metadata: { source: key, q: String(cfg.WEBP_QUALITY) },
    }).promise();

    console.log(`saved -> ${outKey} (${buf.length}B)`);
  }

  return { ok: true };
};
```

---

#### 환경 설정

- **핵심 환경 변수**

  - `BUCKET_NAME` : 업로드/결과를 저장하는 S3 버킷
  - `SOURCE_PREFIX` : 업로드 경로 (예: `users/`)
  - `DEST_PREFIX` : 변환된 이미지 저장 경로 (예: `images/variants/`)
  - `WEBP_QUALITY` : WebP 품질 (0~100)
  - `MAX_WIDTH` : 리사이즈 시 최대 가로 폭 (예: 1280)

- **Terraform 모듈 사용 예시 (요약)**

```hcl
module "images_variants_lambda" {
  source = "../../modules/lambda"

  app_name      = var.app_name
  environment   = var.environment
  function_name = "ImagesVariantsLambda"
  handler       = "index.handler"
  runtime       = "nodejs18.x"
  filename      = "images_variants.zip"
  memory_size   = 1536
  timeout       = 60

  environment_variables = {
    BUCKET_NAME   = module.api_s3_bucket.bucket_name
    SOURCE_PREFIX = "users/"
    DEST_PREFIX   = "images/variants/"
    WEBP_QUALITY  = "80"
    MAX_WIDTH     = "1280"
  }

  iam_policy_statements = [
    {
      Effect   = "Allow"
      Action   = ["s3:GetObject","s3:PutObject","s3:HeadObject"]
      Resource = ["${module.api_s3_bucket.bucket_arn}/*"]
    },
  ]
}
```

S3 쪽에서는 `ObjectCreated` 이벤트를 `ImagesVariantsLambda`로 연결해주면 된다.

---

#### 기존 이미지 마이그레이션

새로 올라오는 이미지뿐 아니라, 운영 중이던 기존 데이터에도 똑같이 최적화가 적용되어야 했다.
그래서 기존 `users/` 경로 아래에 쌓여 있던 이미지들을 한 번에 처리하는 스크립트를 따로 만들어 돌렸다.

- S3에 이미 쌓여 있던 `users/` 경로의 객체 목록을 쭉 가져오고
- 각 객체에 대해 Lambda가 평소에 받는 것과 같은 형태로 **가짜 ObjectCreated 이벤트**를 만들어서
- 같은 Lambda를 그대로 재사용해 한 번에 WebP 변환을 돌렸다.

별도의 마이그레이션 전용 Lambda를 새로 만들기보다는, 운영에 쓰는 Lambda 로직을 그대로 재사용하는 쪽을 선택했다.
그때는 AWS CLI로 `users/` 아래 객체들을 전부 읽어 오고, 각 키마다 Lambda를 호출하는 방식으로 한 번에 돌렸다.

---

#### 결과

11월부터 WebP 리사이즈본을 따로 두는 구조를 적용한 이후, 9·10월 대비 저장·전송 비용이 눈에 띄게 줄었다.
일부 이미지는 용량이 최대 99%까지 줄었고, 그 과정에서도 클라이언트 업로드 로직은 그대로 유지할 수 있었다.

![preSignedUrlImageOptimizationResult](/assets/postImages/PreSignedUrlImageOptimization/pre-signed-url-image-optimization-result.png)
