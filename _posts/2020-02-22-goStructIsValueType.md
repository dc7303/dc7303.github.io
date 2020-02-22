---
layout: post
title: "[GO] 구조체(Struct)는 값 타입(Value Type)이다"
date: 2020-02-22 03:00:00
category: 
- go
tag: 
- go
comments: true
---

고에서 구조체는 값 타입이다. 간단하게 정리해본다.

### 고는 self가 없다. 포인터가 있다
고는 C와 비슷하게 커스텀 자료형으로 구조체[(Struct)](http://golang.site/go/article/16-Go-%EA%B5%AC%EC%A1%B0%EC%B2%B4-struct)를 사용하고 있다. Java, Python, JS 언어를 다뤄본 사람이라면 쉽게 class와 비슷하다고 생각하면 좋다.

근데 고의 인스턴스 사용 방법은 포인터 개념으로 인해 많은 현대언어와 차이점이 있다.

고는 대부분 사용하고 있는 self 또는 this의 개념이 없고, 많은 언어에서 사용하지 않는 포인터 개념을 다시 사용하고 있다.
<details>
<summary>고 포인터를 어려워하지 말자</summary>
<div markdown="1">
고의 포인터는 C의 포인터처럼 자유도가 높지 않다. 고는 포인터에 대한 연산을 지원하지 않는다.

C에서 연산, 캐스팅 등 포인터에 높은 자유도를 부여했다. 그리고 높은 자유도는 프로그램에 대한 복잡성을 높여주었고, 프로그래밍을 처음 배우는 사람이 '포인터는 어렵다'라고 생각하기 충분했다.

반면 고에서는 참조용으로만 사용하고 있고, 메모리도 GC(Garbage Collector)가 관리해주기 때문에 C보다 쉽게 사용할 수 있다.
</div>
</details>

구조체 자신에게 접근하는 메소드를 선언해보자.

```go
package main

import "fmt"

type Person struct {
    name string
    age int
}

func (p *Person) NextYear() {
    p.age++
}

func main() {
    p := Person{"Margurt", 10}
    p.NextYear()
    fmt.Println(p.age)
}
/*
11
*/
```

고에서 인스턴스 메소드에서 자기 자신을 바라볼 때 `*Persion` 포인터 변수로 접근한다.

만약 일반 변수로 수정한다면 어떻게 될까?

```go
...

func (p Person) NextYear() {
    p.age++
}

...
/*
10
*/
```

결과는 10이다. 왜 이럴까?

### Struct는 Value Type이다
Java, JS, Python 등과 같은 대부분의 고수준 언어들은 포인터가 없다. 정확히 말하면 없다기보다는 프로그래머가 알 필요가 없다.  

Python을 예로 들어보자.

```python
class Person(object):
    def __init__(self, name, age):
        self.name = name
        self.age = age

def next_year(p):
    p.age += 1

p = Person("Margurt", 10)
next_year(p)
print(p.age)

p2 = p
p2.age = 20
print(p.age)
'''
11
20
'''
```

파라미터로 전달할 때, 다른 변수에 할당할 때 모두 생성된 `p` 인스턴스를 바라보고 있다.

이렇게 할당되는 방식을 **얕은 복사(Shallow Copy)**라고 하고, 이렇게 동작하는 타입을 **참조 타입(Reference Type)**이라고 한다.

즉, 참조 타입을 통해 언어가 포인터를 활용하고 있다.

반대로 고에서는 포인터가 존재하기 때문에 기본적으로 **깊은 복사(Deep Copy)**로 할당된다. 새로운 메모리에 값을 복사하는 것이다.

위의 고 예제의 `NextYear` 메소드를 일반 함수로 변경하고 파이썬 예제처럼 테스트해 보자.

```go
...

func main() {
    p := Person{"Margurt", 10}
    NextYear(p)
    fmt.Println(p.age)

    p2 := p
    p2.age = 20
    fmt.Println(p.age)
}

func NextYear(p Person) {
    p.age++
}

...
/*
10
10
*/
```

최초 생성된 `p` 인스턴스에 영향을 주지 못한다. 이런 객체 타입을 **값 타입(Value Type)**이라고 한다.

파이썬과 같이 동작하려면 아래와 같이 수정하면 된다.

```go
...

func main() {
    p := Person{"Margurt", 10}
    NextYear(&p)
    fmt.Println(p.age)

    p2 := &p
    p2.age = 20
    fmt.Println(p.age)
}

func NextYear(p *Person) {
    p.age++
}
/*
11
20
*/
...
```

### 맵과 리스트는 다르다
구조체가 기본적으로 값 타입이라고 해서 언어에서 지원하는 기본 자료형도 값 타입이라고 생각하면 안 된다.

```go
package main

import "fmt"

func main() {
    a := map[string]string{
      "name": "Margurt",
      "age": "10",
    }

    b := a
    b["age"] = "20"
    fmt.Println(a["age"]))
}
/*
20
/*
```

맵 a를 b에 할당하고 b를 변경했다. 그리고 변경은 a에 영향을 주고 있는걸 확인할 수 있다.

리스트도 똑같이 동작한다. 이점을 주의하도록 하자.


