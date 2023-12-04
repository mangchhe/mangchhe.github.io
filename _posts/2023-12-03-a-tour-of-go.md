---
layout: post
title: A Tour of Go
categories: go
tags: go
---

## Packages, variables, and fuctions.

#### Imports

- 괄호로 import를 그룹 짓고 이를 `"factored" import` 문이라고 합니다.

```go
import (
    "fmt"
    "math"
)
```

#### Exported names

- 대문자로 시작하는 이름들만 export되고 그렇지 않다면 패키지 밖에서 접근할 수 없다.

```go
import (
    "fmt"
    "math"
)

func main() {
    fmt.Println(math.Pi)
}
```


#### Functions

- 변수 이름 뒤에 타입이 오고 리턴 타입도 마찬가지이다. 코틀린이랑 유사
- 같은 타입이 연속될 경우 마지막 매개변수 타입을 제외하고 생략 가능하다.

```go
func add(x int, y int) int {
    return x + y
}

func subtract(x, y int) int {
    return x - y
}

func main() {
    fmt.Println(add(42, 13))
    fmt.Println(subtract(42, 13))
}
```

#### Multiple results

- 고는 여러 개를 동시에 리턴할 수 있다. 파이썬이랑 유사

```go
func swap(x, y string) (string, string) {
    return y, x
}

func main() {
    a, b := swap("hello", "world")
    fmt.Println(a, b)
}
```

#### Named return values

- 리턴 값에 이름을 정의할 수 있고 변수처럼 사용된다.
- 인자가 없는 return은 이름이 주어진 리턴 값을 반환하는데 이를 `"naked" return`이라고 한다.

```go
func split(sum int) (x, y int) {
    x = sum * 4 / 9
    y = sum - x
    return
}

func main() {
    fmt.Println(split(17))
}
```

#### Variables

- `var` 키워드를 이용하여 변수를 선언하고 마지막은 타입을 정의한다.
- 명시적인 초깃값이 없을 경우 `zero value`가 주어진다.
  - 숫자 : 0
  - 불린 : false
  - 문자열 : "" (빈 문자열)

```go
var c, python, java bool

func main() {
    var i, j int
    fmt.Println(i, j, c, python, java)
}
```

#### Variables with initializers

- 초깃값이 존재한다면 타입을 생략될 수 있다.

```go
var i, j int = 1, 2

func main() {
    var c, python, java = true, false, "no!"
    var golang = "go!"
    fmt.Println(i, j, c, python, java, golang)
}
```

#### Short variable declarations

- `:=`를 이용하여 암시적 타입 선언을 할 수 있다.
- 단, 함수 바깥에서는 `func`, `var` 키워드로 시작하기 때문에 `:=`는 사용 불가능하다.

```go
func main() {
    i, j := 3, 4
    fmt.Println(i, j)
}
```

#### Basic types

- bool
- string
- int  int8  int16  int32  int64
- uint uint8 uint16 uint32 uint64 uintptr
- byte // uint8의 별칭
- rune // int32의 별칭
- float32 float64
- complex64 complex128

```go
var (
    BOOL   bool
    INT    int
    INT8   int8
    INT16  int16
    INT32  int32
    INT64  int64
    UINT   uint
    UINT8  uint8
    UINT16 uint16
    UINT32 uint32
    UINT64 uint64
    BYTE   byte
    RUNE rune
    FLOAT32 float32
    FLOAT64 float64
    COMPLEX64 complex64
    COMPLEX128 complex128
)
```

#### Type conversinos

- T(v)
- 다른 타입의 요소들 간의 할당에는 명시적인 변환이 필요하다.

```go
func main() {
    var x, y int = 3, 4
    var f float64 = math.Sqrt(float64(x*x + y*y))
    var z uint = uint(f)
    fmt.Println(x, y, f, z)
}
```

#### Type inference

- `=`, `:=` 오른편에 상수일 경우 그 상수의 정확도에 따라 타입이 정해진다.

```go
func main() {
    i := 42           // int
    f := 3.142        // float64
    g := 0.867 + 0.5i // complex128

    fmt.Printf("i is of type %T\n", i)
    fmt.Printf("f is of type %T\n", f)
    fmt.Printf("g is of type %T\n", g)
}
```

#### Constants

- 상수는 `const` 키워드를 이용해 선언된다.
- 상수는 `:=`를 통해 선언할 수 없다.

```go
const Pi = 3.14

func main() {
    const World = "World!"
    fmt.Println("Hello", World)
    fmt.Println("Happy", Pi, "Day")

    const Truth = true
    fmt.Println("Go rules?", Truth)
}
```

#### Numeric Constants

- 숫자형 상수는 **매우 정확한 값**이다.

```go
const (
    Big   = 1 << 100
    Small = Big >> 99
)

func needInt(x int) int { return x*10 + 1 }
func needFloat(x float64) float64 {
    return x * 0.1
}

func main() {
    fmt.Println(needInt(Small))
    fmt.Println(needFloat(Small))
    fmt.Println(needFloat(Big))
}
```

## Flow control statements: for, if, else, switch and defer

#### For

- for 초기화 구문; 조건; 증감식 { ... }
- 초기화 구문과 증감식은 필수가 아니다.
- 세 번째 방식처럼 for문을 while문처럼 활용할 수 있다.
- 네 번째 방식처럼 반복 조건을 생략하면 무한 루프를 만들 수 있다.

```go
func main() {
    var sum int

    // 1.
    for i := 0; i <= 10; i++ {
        sum += i
    }

    // 2.
    for ; sum < 1000; {
        sum += sum
    }

    // 3.
    for sum < 1000 {
        sum += sum
    }

    // 4.
    for {
    }

    fmt.Println(sum)
}
```

#### If

- `if` 키워드, 조건식 괄호는 생략이 가능하다.

```go
func sqrt(x float64) string {
    if x < 0 {
        return sqrt(-x) + "i"
    }
    return fmt.Sprint(math.Sqrt(x))
}

func main() {
    fmt.Println(sqrt(2), sqrt(-4))
}
```

#### If with a short statement

- 조건문 전에 수행될 짧은 구문을 작성할 수 있다.
- 짧은 구문에서 선언된 변수들은 if scope 내에서만 존재한다.

```go
func pow(x, n, lim float64) float64 {
    if v := math.Pow(x, n); v < lim {
        return v
    }
    return lim
}

func main() {
    fmt.Println(
        pow(3, 2, 10),
        pow(3, 3, 20),
    )
}
```

#### If and else

- `else` 키워드

```go
func pow(x, n, lim float64) float64 {
    if v := math.Pow(x, n); v < lim {
        return v
    } else {
        fmt.Printf("%g >= %g\n", v, lim)
    }
    return lim
}

func main() {
    fmt.Println(
        pow(3, 2, 10),
        pow(3, 3, 20),
    )
}
```

#### Switch

- `switch`, `case`, `default` 키워드
- switch문도 짧은 구문을 작성할 수 있고 자동 break를 지원하기 때문에 작성할 필요가 없다.
- switch case는 상수일 필요가 없다.
- 조건을 적지 않는다면 `switch true {}` 와 동일

```go
func main() {
    fmt.Print("Go runs on ")
    switch os := runtime.GOOS; os {
    case "darwin":
        fmt.Println("OS X.")
    case "linux":
        fmt.Println("Linux.")
    default:
        fmt.Printf("%s.\n", os)
    }

    today := time.Now().Weekday()
    switch time.Saturday {
    case today + 0:
        fmt.Println("Today.")
    }

    t := time.Now()
    switch {
    case t.Hour() < 12:
        fmt.Println("Good morning!")
    }
}
```

#### Defer

- `defer`은 자신을 둘러싼 함수가 종료될 때까지 함수 실행을 연기한다.
- `defer`로 쌓인 함수들은 후입선출 순서로 실행된다.

```go
func main() {
    defer fmt.Println("world")

    fmt.Println("hello")
}

func main() {
    fmt.Println("counting")

    for i := 0; i < 10; i++ {
        defer fmt.Println(i)
    }

    fmt.Println("done")
}
```

## More types: structs, slices, and maps.

#### Pointers

- 포인터는 값의 메모리 주소를 가진다.
- `*T` 타입은 T 값을 가리키는 포인터이다. zero value는 `nil`이다.
- `&`는 피연산자에 대한 포인터를 생성한다.
- `*`는 포인터가 가리키는 주소의 값을 나타낸다.

```go
func main() {
    i, j := 42, 2701

    p := &i
    fmt.Println(*p)
    *p = 21
    fmt.Println(i)
    fmt.Println(*p)

    p = &j
    *p = *p / 37
    fmt.Println(j)
}
```

#### Structs

- 구조체는 필드의 집합체이다.
- `type 구조체명 struct {}`
- 필드에는 .(dot)으로 접근할 수 있다.

```go
type Vertex struct {
    X int
    Y int
}

func main() {
    vertex := Vertex{1, 2}
    fmt.Println(vertex.X, vertex.Y)

    // 포인터로도 접근 가능
    vertex2 := &vertex
    vertex2.X = 111
    fmt.Println(vertex.X, vertex.Y)
}
```

#### Struct Literals

- 구조체에 값을 할당하는 방법에는 여러 가지 방법이 있다.

```go
type Vertex struct {
    X, Y int
}

var (
    v1 = Vertex{1, 2}  // has type Vertex
    v2 = Vertex{X: 1}  // Y:0 is implicit
    v3 = Vertex{}      // X:0 and Y:0
    p  = &Vertex{1, 2} // has type *Vertex
)

func main() {
    fmt.Println(v1, p, v2, v3)
}
```

#### Arrays

- `[n]T` 타입은 타입이 T인 n개의 값의 배열이다.
- 파이썬과 같이 배열 슬라이스를 지원한다. `a[low, high]`

```go
func main() {
    var a [2]string
    a[0] = "Hello"
    a[1] = "World"
    fmt.Println(a[0], a[1])
    fmt.Println(a)

    primes := [6]int{2, 3, 5, 7, 11, 13}
    fmt.Println(primes)
}
```

#### Slices are like references to arrays

- 슬라이싱하여 만들어진 배열은 새로운 주소를 가지는 배열이 아니다.
- 배열을 수정하게 되면 기존 배열도 같이 수정되게 된다.

```go
func main() {
    names := [4]string{
        "John",
        "Paul",
        "George",
        "Ringo",
    }
    fmt.Println(names)

    a := names[0:2]
    b := names[1:3]
    fmt.Println(a, b)

    b[0] = "XXX"
    fmt.Println(a, b)
    fmt.Println(names)
}
```

#### Slice defaults

- 슬라이스 표현법

```go
func main() {
    var a [10]int
    a[0:10] // 0 ~ 9
    a[:10] // 0 ~ 9
    a[0:] // 0 ~ N
    a[:] // 0 ~ N
}
```

#### Slice length and capacity

- slice는 length, capacity를 가진다.
- length : 슬라이드가 포함하는 요소의 개수
- capacity : 슬라이스의 첫 번째 요소부터 기본 배열의 요소의 개수

```go
func main() {
    s := []int{2, 3, 5, 7, 11, 13}
    printSlice(s) // len=6 cap=6 [2 3 5 7 11 13]

    // Slice the slice to give it zero length.
    s = s[:0]
    printSlice(s) // len=0 cap=6 []

    // Extend its length.
    s = s[:4]
    printSlice(s) // len=4 cap=6 [2 3 5 7]

    // Drop its first two values.
    s = s[2:]
    printSlice(s) // len=2 cap=4 [5 7]
}

func printSlice(s []int) {
    fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)
}
```

#### Nil slices

- 슬라이스 zero value는 nil이다.
- nil 슬라이스의 길이와 용량은 0이며, 기본 배열을 가지고 있지 않다.

```go
func main() {
    var s []int
    fmt.Println(s, len(s), cap(s))
    if s == nil {
        fmt.Println("nil!")
    }
}
```

#### Creating a slice with make

- 슬라이스는 내장된 `make` 함수로 생성할 수 있다. 이건 동적 크기의 배열을 생성하는 방법이다.

```go
func main() {
    a := make([]int, 5)
    printSlice2("a", a) // a len=5 cap=5 [0 0 0 0 0]

    b := make([]int, 0, 5) // type, length, capacity
    printSlice2("b", b) // b len=0 cap=5 []

    c := b[:2]
    printSlice2("c", c) // c len=2 cap=5 [0 0]

    d := c[2:5]
    printSlice2("d", d) // d len=3 cap=3 [0 0 0]
}

func printSlice2(s string, x []int) {
    fmt.Printf("%s len=%d cap=%d %v\n", s, len(x), cap(x), x)
}
```

## Methods and interfaces

#### Value Receiver

- `func (v Vertex) 함수명()` : 함수명 앞에 붙는 걸 Receiver 라고 부른다.
- 리시버가 붙은 함수는 리시버 타입의 메서드가 된다.

```go
type Vertex struct {
    X, Y float64
}

func (v Vertex) Abs() float64 {
    return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func main() {
    v := Vertex{3, 4}
    fmt.Println(v.Abs())
}
```

- 구조체가 아닌 형식에 대해서도 메서드를 선언할 수 있다.

```go
type MyFloat float64

func (f MyFloat) Abs() float64 {
    if f < 0 {
        return float64(-f)
    }
    return float64(f)
}

func main() {
    f := MyFloat(-math.Sqrt2)
    fmt.Println(f.Abs())
}
```

#### Pointer Receiver

- 리시버 유형으로 `*T`를 가질 수 있다.
- 포인터 리시버는 리시버가 가리키는 값을 메서드 내에서 수정할 수 있다.

```go
type Vertex2 struct {
    X, Y float64
}

func (v *Vertex2) Scale(f float64) {
    v.X = v.X * f
    v.Y = v.Y * f
}

func main() {
    v := Vertex2{3, 4}
    v.Scale(10)
    fmt.Println(v.X, v.Y)
}
```

#### Pointers and functions

- 함수 파라미터도 주소로 전달하게 되면 메서드 내부에서 값을 수정할 수 있다.

```go
type Vertex3 struct {
    X, Y float64
}

func Scale(v *Vertex3, f float64) {
    v.X = v.X * f
    v.Y = v.Y * f
}

func main() {
    v := Vertex3{3, 4}
    Scale(&v, 10)
    fmt.Println(v)
}
```

#### Methods and pointer indirection

- 함수에서 포인터 인자를 받기 위해서는 `&`를 사용해야 한다.
- 하지만 포인터 리시버는 포인터가 아니라 값이라도 `(&T)` 포인터 리시버가 있는 메서드를 자동으로 호출한다.

```go
type Vertex4 struct {
    X, Y float64
}

func (v *Vertex4) Scale(f float64) {
    v.X = v.X * f
    v.Y = v.Y * f
}

func ScaleFunc(v *Vertex4, f float64) {
    v.X = v.X * f
    v.Y = v.Y * f
}

func main() {
    v := Vertex4{3, 4}
    v.Scale(2)
    ScaleFunc(&v, 10)

    p := &Vertex4{4, 3}
    p.Scale(3)
    ScaleFunc(p, 8)

    fmt.Println(v, p)
}
```

- 위와 다르게 포인터를 값 인자로 사용할 경우 역참조가 일어나야 한다.
- 인자에 `*`를 붙여야 하고 리시버 같은 경우에는 `(*T)` 처럼 자동으로 역참조가 일어난다.

```go
func (v Vertex4) Abs() float64 {
    return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func AbsFunc(v Vertex4) float64 {
    return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func main() {
    fmt.Println(p.Abs())
    fmt.Println(AbsFunc(*p))
}
```

#### interfaces

- `interface type`은 메서드의 시그니처 집합으로 정의된다.
- `interface` 유형의 값은 해당 메서드를 구현하는 모든 값을 보유할 수 있다.
- `a = v` Vertex 구조체는 포인터 유형에서만 정의되기 때문에 오류를 발생한다.

```go
type Abser interface {
    Abs() float64
}

func main() {
    var a Abser
    f := MyFloat2(-math.Sqrt2)
    v := Vertex5{3, 4}

    a = f
    a = &v
    a = v // Error

    fmt.Println(a.Abs())
}

type MyFloat2 float64

func (f MyFloat2) Abs() float64 {
    if f < 0 {
        return float64(-f)
    }
    return float64(f)
}

type Vertex5 struct {
    X, Y float64
}

func (v *Vertex5) Abs() float64 {
    return math.Sqrt(v.X*v.X + v.Y*v.Y)
}
```

#### Interfaces are implemented implicitly

- `func (t T) M()`는 I 인터페이스를 구현하고 있지만 따로 명시적으로 나타낼 필요는 없다.

```go
type I interface {
    M()
}

type T struct {
    S string
}

func (t T) M() {
    fmt.Println(t.S)
}

func main() {
    var i I = T{"hello"}
    i.M()
}
```

#### Interface values with nil underlying values

- 인터페이스 자체 내부의 콘트리트 값이 0일 경우, 그 메서드는 nil 리시버로 호출됩니다.

```go
type I2 interface {
    M()
}

type T2 struct {
    S string
}

func (t *T2) M() {
    if t == nil {
        fmt.Println("<nil>")
        return
    }
    fmt.Println(t.S)
}

func main() {
    var i I2

    var t *T2
    i = t
    describe(i) // (<nil>, *main.T)
    i.M() // <nil>

    i = &T2{"hello"}
    describe(i) // (&{hello}, *main.T)
    i.M() // hello
}

func describe(i I2) {
    fmt.Printf("(%v, %T)\n", i, i)
}
```

#### Nil interface values

- Nil 인터페이스 값은 값 또는 콘크리트 유형을 모두 가지지 않는다.
- Nil 인터페이스에서 메서드를 호출하는 것은 런타임 에러이다. 이유는 어떠한 구체적인 메서드를 호출할지 나타내는 인터페이스 튜플 내부의 타입이 없기 때문이다.

```go
type I3 interface {
    M()
}

func main() {
    var i I
    describe2(i)
    i.M() // panic: runtime error: invalid memory address or nil pointer dereference
}

func describe2(i I3) {
    fmt.Printf("(%v, %T)\n", i, i)
}
```

#### The empty interface

- 빈 인터페이스는 모든 유형의 값을 가질 수 있다. (최소 0개의 메서드를 구현)
- 빈 인터페이스는 알 수 없는 값을 처리하는 이용된다.

```go
func main() {
    var i interface{}
    describe3(i) // (<nil>, <nil>)

    i = 42
    describe3(i) // (42, int)

    i = "hello"
    describe3(i) // (hello, string)
}

func describe3(i interface{}) {
    fmt.Printf("(%v, %T)\n", i, i)
}
```

#### Type assertions

- `t := i.(T)` type assertion은 인터페이스 값의 기초적인 콘크리트 값에 대한 접근을 제공한다.
- 만약 i가 T를 갖지 못하면 그 선언은 panic 상태가 된다.
- 인터페이스 값이 특정 유형을 보유하는지 여부를 테스트하기 위해 타입 선언에서 기본값과 선언 성공 여부를 보고하는 불린 값을 반환할 수 있다.

```go
func main() {
    var i interface{} = "hello"

    s := i.(string)
    fmt.Println(s) // hello

    s, ok := i.(string)
    fmt.Println(s, ok) // hello true

    f, ok := i.(float64)
    fmt.Println(f, ok) // 0 false

    f = i.(float64) // panic
    fmt.Println(f)
}
```

#### Type switches

- 타입 스위치는 값이 아닌 타입을 명시하여 비교한다.

```go
func do(i interface{}) {
    switch v := i.(type) {
    case int:
        fmt.Printf("Twice %v is %v\n", v, v*2) // Twice 21 is 42
    case string:
        fmt.Printf("%q is %v bytes long\n", v, len(v)) // "hello" is 5 bytes long
    default:
        fmt.Printf("I don't know about type %T!\n", v) // I don't know about type bool!
    }
}

func main() {
    do(21)
    do("hello")
    do(true)
}
```

#### Stringers

- 가장 잘 알려진 인터페이스는 `fmt` 패키지에 있는 `Stringer`이다.
- 자기 자신을 문자열로 표현한다. 값을 표현하기 위해 해당 인터페이스 많이 찾는다. java의 toString()으로 생각하면 된다.

```go
type Stringer interface {
    String() string
}
```

## References

- [https://go.dev/tour/list](https://go.dev/tour/list)