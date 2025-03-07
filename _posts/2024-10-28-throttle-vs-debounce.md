---
layout: post
title: Throttle vs Debounce
categories: cs
tags: cs
---

Throttle과 Debounce는 연속적인 실행을 제한하기 위해 (성능 최적화) 이벤트 호출 빈도를 조절하는 두 가지 주요 기법이다. 이 둘은 동작 방식이 다르므로, 상황에 맞는 적절한 방법을 선택해서 사용해야 한다.

##### 스로틀 (Throttle)

정해진 시간 간격마다 한 번씩만 이벤트를 처리한다. 즉, 일정 시간 동안 여러 번 발생한 이벤트가 있어도 한 번만 실행되도록 제한한다.

##### 디바운스 (Debounce)

연속된 이벤트가 발생했을 때, 마지막 이벤트를 처리한다. 즉, 연속해서 이벤트가 발생하는 동안 실행을 지연시키다가, 마지막 이벤트 이후에 일정 시간이 지나면 실행되도록 제한한다.

둘 다 연속적인 실행을 제한하는 공통 목적을 가지지만, **Throttle은 주기적으로 호출**을 허용하는 반면 **Debounce는 연속적인 이벤트 중 마지막 이벤트 이후 일정 시간 후에만 실행**된다.