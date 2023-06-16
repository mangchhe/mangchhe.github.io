---
title: "Practice tdd, refactor - woowahan tech seminar"
categories: seminar
tags: seminar
---

# 의식적인 연습의 7가지 원칙

<hr>

1. 효과적인 훈련 기법이 수립된 기술 연마
1. 개인의 **컴포트 존을 벗어난 지점에서 진행, 자신의 현재 능력을 살짝 넘어가는 작업을 지속적으로 시도**
1. 명확하고 구체적인 목표를 가지고 진행
1. 신중하고 계획적이다. 개인이 온전히 집중하고 '의식적'으로 행동할 것을 요구
1. **피드백과 피드백에 따른 행동 변경을 수반**
1. 효과적인 심적 표상을 만들어내는 한편으로 심적 표상에 의존
1. **기존에 습득한 기술의 특정 부분을 집중적으로 개선함으로써 발전시키고, 수정하는 과정을 수반**

> 컴포트 존 (Comfort Zone)
>
> 편안함을 느끼는 영역을 말함. 예를 들어 내가 핸드폰을 가지고 침대 안에서 귤을 까먹는 상태

# 의식적인 연습으로 TDD, 리팩토링 연습하는 과정

<hr>

## 단위 테스트 연습 (1단계)

- 내가 사용하는 API 사용법을 익히기 위한 학습 테스트
- Input과 Output이 명확한 클래스 메소드 (Util 성격을 띠는 - 날짜 변환 등)

## TDD 연습 (2단계)

![tddProcess](/assets/postImages/TddOkkyReview/tddProcess.png){: width="600"}

1. 실패하는 테스트를 만든다.
2. 컴파일을 성공하게 만들고 프로덕션 코드를 만든다.
3. 리팩토링한다.

- 회사 프로젝트에 연습하지 말고 장난감 프로젝트를 활용해 연습
- 웹, 모바일 UI나 DB에 의존관계를 가지지 않는 요구사항을 연습

### 문자열 덧셈 계산기 요구사항

- 쉼표(,) 또는 콜론(:)을 구분자로 가지는 문자열을 전달하는 경우 (Input)
- 구분자를 기준으로 분리한 각 숫자의 합을 반환 (Output)

![requirements](/assets/postImages/TddOkkyReview/requirements.png)

### 실패하는 코드 작성

```java
public class StringCalculatorTest {
    @Test
    public void null_또는_빈값() {
        assertThat(StringCalculator.splitAndSum(null)).isEqualsTo(0);
        assertThat(StringCalculator.splitAndSum("")).isEqualsTo(0);
    }

    @Test
    public void 값_하나() {
        assertThat(StringCalculator.splitAndSum("1")).isEqualsTo(1);
    }

    @Test
    public void 쉽표_구분자() {
        assertThat(StringCalculator.splitAndSum("1,2")).isEqualsTo(3);
    }

    @Test
    public void 쉼표_콜론_구분자() {
        assertThat(StringCalculator.splitAndSum("1,2:3")).isEqualsTo(6);
    }
}
```

### 프로덕션 코드 작성

```java
public class StringCalculator {
    public static int splitAndSum(String text){
        result = 0;
        if(text == null || text.isEmpty()) {
            result = 0;
        } else {
            String[] values = text.split(",|:");
            for (String value : values) {
                result += Integer.parseInt(value);
            }
        }
        return result;
    }
}
```

## 리팩토링 연습 (3단계)

### 한 메서드에 오직 한 단계의 들여쓰기(indent)

- depth가 2인 부분을 따로 메소드로 분리

```java
public class StringCalculator {
    public static int splitAndSum(String text){
        result = 0;
        if(text == null || text.isEmpty()) {
            result = 0;
        } else {
            String[] values = text.split(",|:");
            // for (String value : values) {
            //     result += Integer.parseInt(value);
            // }
            result = sum(values);
        }
        return result;
    }

    private static int sum(String[] values) {
        int result = 0;
        for (String value: values) {
            result += Integer.parseInt(value);
        }
        return result;
    }
}
```

### else 예약어를 쓰지 않는다.

- else를 없앰으로써 indent도 줄이고 더 간결해졌다.

```java
public class StringCalculator {
    public static int splitAndSum(String text){
        if(text == null || text.isEmpty()) {
            return 0;
        } 
        String[] values = text.split(",|:");
        return sum(values);
    }

    private static int sum(String[] values) {
        int result = 0;
        for (String value: values) {
            result += Integer.parseInt(value);
        }
        return result;
    }
}
```

### 메소드가 한 가지 일만 하도록 구현하기

- sum 메서드는 두 가지 일을 한다. (메소드 분리의 신호)
    - 값을 int로 캐스팅
    - 값들의 합을 구함
- 쪼갠 메서드를 지역변수를 사용해야 하나?
    - 둘 중 어떤 게 나은지 고려해봐야 함

```java
public class StringCalculator {
    public static int splitAndSum(String text){
        if(text == null || text.isEmpty()) {
            return 0;
        } 
        String[] values = text.split(",|:");
        int[] numbers = toInts(values); 
        return sum(values);
        // return sum(toInts(text.split(",|:")));
    }

    private static int[] toInts(String[] values) {
        int[] numbers = new int[values.length];
        for (int i = 0; i < values.length; i++) {
            numbers[i] = Integer.parseInt(values[i]);
        }
        return numbers;
    }

    private static int sum(int[] numbers) {
        int result = 0;
        for (int number : numbers) {
            reuslt += number;
        }
        return result;
    }
}
```

### compose method 패턴 적용

- 메서드의 의도가 잘 드러나도록 **동등한 수준**의 작업을 하는 여러 단계로 나눈다.
- 메서드들의 추상화 수준을 똑같이 만든다.
- 왜 이렇게까지 할까?
    - 장난감 프로젝트이기 때문에
    - 연습할 때는 극단적으로 하자. 그래야 insight(통찰력)가 생긴다.

```java
public class StringCalculator {
    public static int splitAndSum(String text){
        if(isBlank()) {
            return 0;
        } 
        String[] values = split(text);
        int[] numbers = toInts(values); 
        return sum(values);
    }
    
    private static boolean isBlank(String text) {
        if(text == null || text.isEmpty()) {
            return 1;
        }
        return 0;
    }

    private static String[] split(String text) {
        return text.split(",|:");
    }

    private static int[] toInts(String[] values) {
        int[] numbers = new int[values.length];
        for (int i = 0; i < values.length; i++) {
            numbers[i] = Integer.parseInt(values[i]);
        }
        return numbers;
    }

    private static int sum(int[] numbers) {
        int result = 0;
        for (int number : numbers) {
            reuslt += number;
        }
        return result;
    }
}
```

## 문자열 덧셈 계산기 요구사항 추가

- 문자열 계산기에 숫자 이외의 값 또는 음수를 전달하는 경우 RuntimeException 예외를 throw 한다.

![requirements2](/assets/postImages/TddOkkyReview/requirements2.png)

### 테스트 코드 작성

```java
@Test(excepted = RuntimeException.class)
public void 음수값() {
    StringCalculator.splitAndSum("-1,2:3")
}
```

### 프로덕션 코드 작성

```java
public class StringCalculator {
    public static int splitAndSum(String text){
        if(isBlank()) {
            return 0;
        } 
        String[] values = split(text);
        int[] numbers = toInts(values); 
        return sum(values);
    }

    private static int[] toInts(String[] values) {
        int[] numbers = new int[values.length];
        for (int i = 0; i < values.length; i++) {
            // numbers[i] = Integer.parseInt(values[i]);
            numbers[i] = toInt(values[i]);
        }
        return numbers;
    }

    private static int toInt(String value) {
        int number = Integer.parseInt(value);
        if (number < 0) {
            throw new RuntimeException();
        }
        return numbers;
    }
}
```

### 모든 원시값과 문자열을 포장한다.

```java
public class Positive{
    private int number;

    public Positive(String value) {
        this(Integer.parseInt(value));
    }

    public Positive(int number) {
        if(number < 0){
            throw new RuntimeException();
        }
        this.number = number;
    }

    public Positive add(Positive other) {
        return new Positive(this.number + other.number);
    }

    public int getNunber() {
        return number;
    }
}

public class StringCalculator {
    ...
    private static Positive[] toInts(String[] values) {
        Positive[] numbers = new int[values.length];
        for (int i = 0; i < values.length; i++) {
            numbers[i] = new Positive(values[i]);
        }
        return numbers;
    }

    private static int sum(Positive[] numbers) {
        Positive result = 0;
        for (int number : numbers) {
            reuslt += result.add(number);
        }
        return result.getNumber();
    }
}
```

## 클래스 분리 연습을 위해 활용할 수 있는 원칙

- 일급 컬렉션을 쓴다.
- 3개 이상의 인스턴스 변수를 가진 클래스를 쓰지 않는다.

## 장난감 프로젝트 난이도 높이기 (4단계)

- 점진적으로 요구사항이 복잡한 프로그램을 구현한다.
- TDD, 리팩토링 연습하기 좋은 프로그램 요구사항
    - 게임과 같이 요구사항이 명확한 프로그램으로 연습
    - 의존관계(데이터베이스, 웹 UI, 외부 API와 같은 의존관계)가 없이 연습
    - 약간의 복잡한 로직이 있는 프로그램
    - 연습하기 좋은 예
        - 로또
        - 사다리 타기
        - 볼링 게임 점수판
        - 체스 게임
        - 지뢰 찾기 게임

## 의존관계 추가를 통한 난이도 높이기 (5단계)

- 데이터베이스, 웹 UI, 외부 API와 같은 의존관계 추가

# 한 단계 더 나아간 연습을 하고 싶다면

<hr>

- 컴파일 에러를 최소화하면서 리팩토링하기
- ATDD 기반으로 응용 애플리케이션 개발하기
- 레거시 애플리케이션에 테스트 코드 추가해 리팩토링하기

> ATDD(Acceptance Test Driven Development)
> 
> 사용자 **스토리를 기반**으로 **인수 조건**을 도출하여 기능 개발을 진행하는 방법론

# 객체지향 생활체조 규칙

<hr>

1. 한 메서드에 오직 한 단계의 들여쓰기만 한다.
1. else 예약어를 쓰지 않는다.
1. 모든 원시값과 문자열을 포장한다.
1. 한 줄에 점을 하나만 찍는다.
1. 줄여 쓰지 않는다(축약 금지)
1. 모든 엔티티를 작게 유지한다.
1. 3개 이상의 인스턴스 변수를 가진 클래스를 쓰지 않는다.
1. 일급 컬렉션을 쓴다.
1. 게터/세터/프로퍼티를 쓰지 않는다.

# 내가 리더

<hr>

- 1:1 공략, 팀원이 개선할 부분을 말하고, 해결책을 제안하도록 유도
- 팀원들과의 신뢰 형성이 우선, 1:1 면담을 통해 개선할 부분 찾기

## 1:1 면담에서 한 가지만 기억하자.

- 팀원이 문제점을 이야기할 때 문제점에 대한 해답을 제시하려고 X
- '어떻게 하면 될까?', '너라면 어떻게 할 것 같아'라는 반문을 해라.
- 팀 회고를 통해 우선순위가 높으면서 작은 변화를 통해 **가장 높은 효과가 있는 것으로 생각하는 Practice를 선택**한다.

# 결론

<hr>

- 한 번에 모든 원칙을 지키면서 리팩토링하려고 연습하지 마라.
- 한 번에 한 가지 명확하고 구체적인 목표를 가지고 연습해라
- 지속적인 리팩토링이 가능한 이유가 테스트가 뒷받침하기 때문이다.
- 컴포트 존에서 벗어나라. 그러기 위해선 삶의 여유가 있어야 한다.

# Reference

<hr>

- [https://youtu.be/cVxqrGHxutU](https://youtu.be/cVxqrGHxutU)
- [https://youtu.be/bIeqAlmNRrA](https://youtu.be/bIeqAlmNRrA)