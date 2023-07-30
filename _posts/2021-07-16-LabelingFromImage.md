---
layout: post
title: "Label object with python"
categories: imageprocess
tags: imageprocess
---

# 상황

![label_situation](/assets/postImages/LabelingFromImage/label_situation.JPG)

다양한 객체들이 있는 화면에서 초록색을 가지는 객체의 이미지만 뽑아내고 싶거나 객체 갯수 등을 알고싶을 때 어떻게 해야 될까?

이때 영상처리 기법 **라벨링(Labeling)** 을 사용하면 위 내용들을 한번에 해결할 수 있다.

# 라벨링(Labeling)이란?

이미지 내에서 주위 같은 픽셀값들을 가지는 픽셀들을 그룹화하여 그룹별로 번호를 매긴 것이다.

# 라벨링 과정 설명

1. image grayscale
2. image 이진화(binarization)
3. grassfire algorithm

## image grayscale

컬러 이미지를 흑백 이미지로 변환해야 한다. 이유는 간단하다. RGB라고 한다면 색상이 RGB(255(R) * 255(G) * 255(B))개로 너무 다양해서 시간이 오래걸리고 다양한만큼 비슷한 색상을 걸러내기 위해서 if(최저색상 < 색상 < 최대색상)과 같은 조건문이 필요로 한다.
쉽게 말해서 **컬러 이미지 -> 흑백 이미지 지 변환 작업을 해야하고 하는 이유는 컬러 이미지를 이용하면 시간이 오래 걸리고 비효율적이기 때문이다.**

``` python
# 이미지 읽어올 때 흑백으로 불러오기
src = cv2.imread('파일명.jpg', cv2.IMREAD_GRAYSCALE)

# 컬러 이미지(RGB)를 흑백 이이미지(GRAY)로 변환
gray = cv2.cvtColor(src, cv2.COLOR_BGR2GRAY)
```

## image 이진화(binarization)

0 ~ 255의 픽셀값을 가지는 흑백 이미지를 255(백)과 0(흑) 픽셀값만 가지는 이미지로 변환하는 것을 말한다.
임계값을 정하여 임계값 보다 낮은 값을 가지면 흑, 임계값 보다 높은 값을 가지면 백으로 만든다.
이진화를 하는 이유는 추출하고자하는 객체에 대해서 효율적으로 처리하기 위해서 하는 전처리 작업이라고 생각하면 된다.

``` python
# param
# grayscale image
# 임계값보다 작다면 0
# 임계값보다 크다면 255
# cv2.THRESH_OTSU는 임계값을 자동으로 구하는 알고리즘
_, src_bin = cv2.threshold(src, 0, 255, cv2.THRESH_OTSU)
```

## grassfire algorithm

영어를 그대로 해석하면 잔디 + 불 알고리즘이다. 알고리즘이 잔디에 불을 붙인 형태로 로직이 처리되기 때문에 붙여진 이름이다. 그러면 이 알고리즘이 무엇인가?

![label_explanation](/assets/postImages/LabelingFromImage/label_explanation.JPG)

시작 지점으로부터 사방으로 퍼져가며 객체를 찾는 방식이다. 위 그림에 대략적으로 흘러가는 알고리즘에 대해서 작성하였다. 이해가 안간다면 다음 [백준 링크](https://www.acmicpc.net/problem/21938)에서 문제를 풀어보도록 하자.

|-|grassfire|image|
|:-:|:-:|:-:|
|시작 지점|불을 집힌 곳|255(백) 픽셀값을 가진 곳|
|이동|좌우상하로 잔디가 있는 곳에 불이 번짐|좌우상하 255 픽셀값을 가지는 곳으로 확장|

``` python
# return
# cnt : 객체 총 갯수 - 1(배경 제외)
# labels : 각 객체 번호
# stats : x, y, w ,h
# centroids : 각 객체 중심 좌표
cnt, labels, stats, centroids = cv2.connectedComponentsWithStats(src_bin)
```

# 라벨링 결과

![label_result](/assets/postImages/LabelingFromImage/label_result.JPG)

``` python
import cv2

src = cv2.imread('map.jpg', cv2.IMREAD_GRAYSCALE)

_, src_bin = cv2.threshold(src, 0, 255, cv2.THRESH_OTSU)

cnt, labels, stats, centroids = cv2.connectedComponentsWithStats(src_bin)

dst = cv2.cvtColor(src, cv2.COLOR_GRAY2BGR)

for i in range(1, cnt):
    (x, y, w, h, area) = stats[i]

    cv2.putText(dst, str(i), (x + w // 2 - 5, y + h // 2 + 5), cv2.FONT_HERSHEY_PLAIN, 1, (0, 0, 255), 1, cv2.LINE_AA)
    cv2.rectangle(dst, (x, y, w, h), (0, 0, 255))

cv2.imshow('src', src)
cv2.imshow('src_bin', src_bin)
cv2.imshow('dst', dst)
cv2.waitKey(0)
```

# 라벨링 응용

원하는 색상만 객체 라벨링을 이용해보자

색상 추출과 팽창의 개념을 모른다면 → [링크](https://mangchhe.github.io/imageprocess/2021/06/07/ExtractDarkColorFromImage/)

![label_result2](/assets/postImages/LabelingFromImage/label_result2.JPG)

``` python
import cv2
import numpy as np

src = cv2.imread('map_2.jpg')

# 소스 추가(응용)
lower_green = (79, 209, 146)
upper_green = (79, 209, 146)
hsv = cv2.inRange(src, lower_green, upper_green)
hsv_dilate = cv2.dilate(hsv, np.ones((3, 3), np.uint8), iterations=3)

cnt, labels, stats, centroids = cv2.connectedComponentsWithStats(hsv_dilate)

dst = cv2.cvtColor(hsv_dilate, cv2.COLOR_GRAY2BGR)

for i in range(1, cnt):
    (x, y, w, h, area) = stats[i]

    cv2.putText(dst, str(i), (x + w // 2 - 5, y + h // 2 + 5), cv2.FONT_HERSHEY_PLAIN, 1, (0, 0, 255), 1, cv2.LINE_AA)
    cv2.rectangle(dst, (x, y, w, h), (0, 0, 255))

cv2.imshow('src', src)
cv2.imshow('hsv', hsv)
cv2.imshow('hsv_dilate', hsv_dilate)
cv2.imshow('dst', dst)
cv2.waitKey(0)
```
