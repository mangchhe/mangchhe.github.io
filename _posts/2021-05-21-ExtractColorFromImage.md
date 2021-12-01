---
title: "[영상 처리] 파이썬으로 이미지 속에 원하는 색상 추출하기"
decription: 취미로 삼아 매크로를 만들곤 했는데 필요에 따라 이미지 서치 등 작업을 하기 위해서 이미지에 대해서 전처리 작업이 필요했었다. 그때마다 많이 사용했던 영상 처리 기술을 포스팅 해보려고 한다. 이번 포스터는 이미지에 원하는 색상을 추출하는 방법에 대해서 알아보자
categories:
 - ImageProcess
tags:
 - Python
 - Image
 - OpenCV
 - Numpy
---

> 취미로 삼아 매크로를 만들곤 했는데 필요에 따라 이미지 서치 등 작업을 하기 위해서 이미지에 대해서 전처리 작업이 필요했었다. 그때마다 많이 사용했던 영상 처리 기술을 포스팅 해보려고 한다. 이번 포스터는 이미지에 원하는 색상을 추출하는 방법에 대해서 알아보자

# 이미지 준비

<hr>

![extractImageColor_square](/assets/postImages/ExtractColorFromImage/extractImageColor_square.PNG)

테스트하기 위해서 파란색 사각형이 여러 개 그려진 사진을 준비하였다.

# 그림판을 통해 RGB값 구하기

<hr>

![extractImageColor_square2](/assets/postImages/ExtractColorFromImage/extractImageColor_square2.PNG)

찾고자 하는 이미지를 그림판에 가져와 스포이드를 통해 찾으려는 색상 위치에 가져다대고 색 편집을 클릭한다.

![extractImageColor_square3](/assets/postImages/ExtractColorFromImage/extractImageColor_square3.PNG)

색 편집을 누르게 되면 위에 RGB값이 나타나는 것을 확인할 수 있다. 이제 이 RGB값을 통해서 이미지에서 색 추출을 해보도록 하자

# 패키지 설치

<hr>

``` bash
pip install opencv-python
pip install numpy
```

# 코드 구현

<hr>

내가 준비한 사진은 사각형이 모두 같은 색상인 거 같지만 하나의 사각형에는 다른 색상을 주었고, 다른 색상을 가지는 사각형을 추출하는 방법에 대해서 여러 코드를 보고 배워보자

## 첫 번째 방법

``` python
import cv2
import numpy as np

img = cv2.imread('square.png')

h, w, _ = img.shape

# img = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)

for i in range(h):
    for j in range(w):
        if not all(img[i][j] == [255, 0, 0]) and not all(img[i][j] == [255, 255, 255]):
            img[i][j] = [0, 255, 255]

cv2.imwrite('trans_square.png', img)
```

사각형 이외에의 부분 흰색(255, 255, 255)과 파란색(0, 255, 0)이 아닌 부분에 대해서 노란색(255, 255, 0)으로 칠하는 코드이다. 이때 주의해야 할 점은 왜 `all(img[i][j] == [255, 0, 0])`, `img[i][j] = [0, 255, 255]` 부분이 파란색과 노란색인데 RGB 값이 다를 것이다. 이유는 `cv.imread()` 하여 이미지를 가져오게 되면 **RGB 값이 아닌 BGR 값으로 불러오기 때문에 순서가 RGB -> BGR로 변화된 것**이다. RGB값을 이용하고 싶다면 주석되어 있는 소스 코드를 추가해주도록 하자

## 두 번째 방법

``` python
import cv2
import numpy as np

img = cv2.imread('square.png')

h, w, _ = img.shape

for i in range(h):
    for j in range(w):
        if all(img[i][j] == [254, 0, 0]):
            img[i][j] = [0, 255, 255]

cv2.imwrite('trans_square.png', img)
```

찾고자 하는 색상에 RGB값을 그대로 찾는 방법이다. 크게 로직 부분은 변화된 것은 없고 조건문만 조금 변경 되었다.

## 세 번째 방법

``` python
import cv2
import numpy as np

img = cv2.imread('square.png')

pos = np.where((img[:,:,0] == 254) & (img[:,:,1] == 0) & (img[:,:,2] == 0))
for h, w in zip(pos[0], pos[1]):
    img[h][w] = [0, 255, 255]

cv2.imwrite('trans_square.png', img)
```

np.where()를 이용하여 찾고자 하는 위치를 먼저 찾고 색상을 입히는 작업을 진행한 코드이다.

# 결과

<hr>

![extractImageColor_square4](/assets/postImages/ExtractColorFromImage/extractImageColor_square4.png)

위 세 개의 소스 코드를 이용하면 다른 사각형과 다른 색상을 가지는 사각형이 노란색으로 변화되어 파일이 저장되는 것을 볼 수 있다.

# 반복문 vs np.where()

<hr>

예제들을 보게 되면 반복문과 np.where을 사용하는 두 가지 방법이 있다. 그럼 어떤 방법을 이용하는 것이 좋을까?

내가 처음 색상 추출에 접했을 때 1, 2번째 방법과 같이 반복문을 이용하여 색상을 추출하였는데 대부분 매크로에 적용하는 것이었는데 단순히 시간 복잡도를 계산하면 O(w * h)가 되어 시시각각 변화하는 이미지에 적용하기 속도가 너무 느린 감이 있었다.

배열은 numpy를 사용하고 있지만, 반복문을 이용하는 너무 구시대적 방법을 이용했던 것 같아 numpy를 이용한 방법이 어떤 것이 있을까 찾아보다가 where이라는 메소드를 찾게 되었고 이 메소드를 이용하면 속도를 크게 향상 시킬 수 있다는 것을 알게 되었다.

![extractImageColor_square5](/assets/postImages/ExtractColorFromImage/extractImageColor_square5.PNG)

현재 예제를 통해 반복문과 where를 이용했을 때를 5번을 반복하여 걸린 시간을 확인했다.

# 결론

<hr>

이미지 속에 색상을 추출할 때 np.where을 이용해라
