---
title: "Extract colors with morphology expansion operation and InRange() function with python"
categories: imageprocess
tags: imageprocess
---

# 상황

<hr>

이전 포스터에서는 파란 사각형의 면 색깔이 모두 같아서 추출하는 데에 큰 문제가 없었다. 하지만 **만약에 면의 색깔이 다 같지 않다면 어떻게 될까?**

![extractImageDarkColor1](/assets/postImages/ExtractDarkColorFromImage/extractImageDarkColor1.PNG)

왼쪽 사진 사각형을 보면 여러 가지 색상이 섞여 있는 것을 알 수 있다. 여기서 파란색(0, 255, 0)을 그대로 추출하게 된다면 오른쪽 사진처럼 사각형 전체가 아닌 해당하는 일부분만 추출이 될 것이다.

# 해결 방법

<hr>

내가 사용하는 방법에는 두 가지가 있다.

1. 모폴로지 변환 중 팽창(dilation) 방법을 이용하는 방법
2. `cv2.inRange()` 함수를 이용하는 방법

이 두가지 방식은 상황에 맞게 사용해야한다.

## 팽창을 이용하는 방법

우선 팽창이 무엇인지 모른다면 팽창이 무엇인지부터 알아보자

![extractImageDarkColor2](/assets/postImages/ExtractDarkColorFromImage/extractImageDarkColor2.PNG)

위 사진에서 회색 바탕은 이미지이며 각 사각형은 픽셀, 빨간색으로 둘러쌓인 곳이 연산을 수행할 범위이고 노란색이 현재 연산의 대상이라고 생각하면 된다. 현재 마스크의 크기는 3x3으로 지정했으며 두 가지의 상황이 나타나있다.

왼쪽에 있는 노란색 테두리 사각형은 주위가 모두 검은색이다. 이럴 경우에는 해당 사각형을 검은색(0)으로 즉, 물체를 확장하지 않는다. 오른쪽 노란색 테두리의 경우 빨간색(Mask) 칸 안에 흰색(1)이 하나라도 존재하게 되면 같이 흰색으로 변경된다 즉, 물체를 확장한다.

정리하면 특정 픽셀 기준으로 마스크의 크기 안에 흰색(1)이 하나라도 존재할 경우 그 특정 픽셀로 흰색 그 외에는 검은색(0)으로 변경된다.

![extractImageDarkColor3](/assets/postImages/ExtractDarkColorFromImage/extractImageDarkColor3.PNG)

팽창을 하면 위 사진과 같이 흰색 부분이 확장된 것을 확인할 수 있고 Mask의 크기와 연산을 반복해서 확장될 영역의 크기를 결정할 수 있다.

![extractImageDarkColor4](/assets/postImages/ExtractDarkColorFromImage/extractImageDarkColor4.png)

위 사진은 기존의 이미지와 팽창 연산한 이미지의 비트 and 연산을 통해 사각형을 추출한 것을 볼 수 있다.

``` python
import cv2
import numpy as np
# 이미지 불러오기
img = cv2.imread('square.png')
# 이미지 크기 얻기
h, w, o = img.shape
# 검은색 도화지 생성
black_img = np.zeros((h, w, o), dtype=np.uint8)
# 알고 있는 색상 위치 추출
pos = np.where((img[:,:,0] == 255) & (img[:,:,1] == 0) & (img[:,:,2] == 0))
# 해당 위치 흰색으로 색칠
for h, w in zip(pos[0], pos[1]):
    black_img[h][w] = [255, 255, 255]
# 팽창 연산 수행 (9, 9) : Mask 크기 지정, iterations = ? : 팽창 반복 횟수 지정
result = cv2.dilate(black_img, np.ones((9, 9), np.uint8), iterations=14)
# 기존 이미지와 비트 연산
result = cv2.bitwise_and(img, result)
# 결과 저장
cv2.imwrite('trans_square.png', result)
```

## cv2.inRange()를 이용하는 방법

`inRange(image, lower_color, upper_color)` 함수를 통해 lower_color와 upper_color 사이에 있는 색상 값들이 존재하는 픽셀을 1로 만들고 그 외에는 0으로 만든 이미지를 반환한다.

inRange를 이용하기 위해서는 색상값을 BGR 값이 아닌 HSV를 이용하기 때문에 변환을 해주어야 한다.

관련 내용 : [링크](https://docs.opencv.org/3.4/da/d97/tutorial_threshold_inRange.html)

![extractImageDarkColor5](/assets/postImages/ExtractDarkColorFromImage/extractImageDarkColor5.PNG)

결과적으로 inRange()를 이용해서도 사각형 추출이 잘되는 것을 알 수 있다.

``` python
import cv2
import numpy as np
# 이미지 불러오기
img = cv2.imread('square.png')
# 이미지 크기 얻기
h, w, o = img.shape
# BGR -> HSV 색상으로 변경
hsv = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)
# 추출할 색상 영역
lower_blue = (110, 50, 50)
upper_blue = (130, 255, 255)
# 이미지에서 파랑색 영역 추출
hsv = cv2.inRange(hsv, lower_blue, upper_blue)
# 기존 이미지 씌우기
result = cv2.bitwise_and(img, img, mask=hsv)
# 결과 저장
cv2.imwrite('trans_square.png', result)
```

# 팽창, cv2.inRange() 차이점

<hr>

팽창은 추출하고자하는 영역에 대해서 매끄럽게 추출하지 못하지만 inRange()의 경우에는 뽑고자 하는 색상에 대해서 정확히 알고 있다면 매끄럽게 추출할 수 있다.

만약 뽑고자하는 영역에 대해서 색상이 너무 다양하다면 범위 내에 색상을 얻는 것은 사실상 불가능하며 범위를 너무 넓혀버리면 추출하고자하는 영역이 아닌 다른 영역까지 다 추출하게 될것이다. 하지만 팽창같은경우는 유일한 색상 값이 있다면 아무리 주위 색상이 다양하더라도 그 색상을 기준으로 팽창을 하면 원하는 영역에 대해서 추출을 할 수 있을 것이다.
