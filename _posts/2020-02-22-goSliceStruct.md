---
layout: post
title: "[GO] 동적 배열"
date: 2020-02-22 02:00:00
category: 
- go
tag: 
- go
comments: true
---

고에서 동적 배열의 동작 원리를 이해하고 사용할 때 주의할 점을 정리한다.

### 동적배열 선언
고에서 동적 배열을 **Slice**라고 부른다. 선언은 아래와 같이 할 수 있다.

```go
package main

import "fmt"

func main() {
    var a []int
    a2 := []int{}
    a3 := []int{1, 2, 3, 4}
    // 2번쨰 인자는 Length, 3번쨰 인자는 Capacity
    a4 := make([]int, 3)
    a5 := make([]int, 3, 4)
}
```

Slice는 내부적으로 길이(length)와 수용범위(Capacity)를 가지고 있다.  
<details>
<summary>길이와 수용범위??</summary>
<div markdown="1">
길이는 현재 값이 존재하는 범위를 나타내고, 수용범위는 해당 배열의 메모리가 할당받는 메모리의 크기라고 생각해볼 수 있다.

만약 길이가 3이고 수용범위가 4인 배열이 있다면 [◼◼︎◼︎︎◻︎] 이런 형태일 것이다.

즉 1개 값을 할당받지 못한 배열 내 메모리 공간이 존재한다는 뜻이다.
</div>
</details>

```go
...

func main() {
    a := make([]int, 3, 4)
    fmt.Printf("len=%d, cap=%d", len(a), cap(a))
}
/*
len=3, cap=4
*/
```

### 동적배열 원리
일반적인 배열은 길이와 타입의 사이즈를 곱한 메모리 영역을 가지고 있다. 그렇기 때문에 메모리를 벗어난 영역에 접근할 경우 프로그램이 에러를 발생시킨다.

하지만 복잡한 프로그램을 작성할 때 고정된 배열은 한계가 있다. 그렇기 때문에 동적 배열이 나온 것이다.

고에서 동적 배열은 범위를 벗어날 때 새로운 메모리 영역에 메모리를 확보하고, 기존에 있던 값들을 복사하여 배열의 길이를 유동적으로 관리한다.

하지만 매번 새로운 메모리 영역을 확보하는 것은 아니다.

코드로 확인해보자.

```go
...

func main() {
    // 초기화
    var a []int
    printSliceInfo(a)

    // (1, 2) 추가
    a = append(a, 1, 2)
    printSliceInfo(a)

    // (3) 추가
    a = append(a, 3)
    printSliceInfo(a)

    // (4) 추가
    a = append(a, 4)
    printSliceInfo(a)
}

// 슬라이스 정보 출력
func printSliceInfo(a []int) {
    fmt.Printf("len=%d cap=%d slice=%v addr=%p\n", len(a), cap(a), a, a)
}

/*
len=0 cap=0 slice=[] addr=0x0                   초기화
len=2 cap=2 slice=[1 2] addr=0xc0000180c0       (1, 2)추가
len=3 cap=4 slice=[1 2 3] addr=0xc000014280     (3)추가
len=4 cap=4 slice=[1 2 3 4] addr=0xc000014280   (4)추가
*/
```
4줄의 출력 결과를 통해 동적 배열이 어떻게 동작하는지 확인해보자.

1. 초기화 했을 때와 (1, 2)를 추가했을 때 메모리 주소를 확인해보면 **다른 영역**인 걸 확인할 수 있다.
2. (3)을 추가했을 때도 새로운 메모리 주소를 가지고 있다. 중요한 것은 메모리가 생성될 때 **메모리 범위(Capacity)를 이전의 두 배 값**으로 확보했다.
3. (4)를 추가했을 때는 **같은 메모리 주소**다. 핵심은 **새로운 메모리를 할당받는 시점은 메모리 범위(Capacity)의 영향을 받는다**.

결과에서 확인할 수 있듯이 고에서 동적 배열은 **새로운 메모리 영역을 생성하거나, 현재 메모리 영역을 사용해서 배열 크기를 동적으로 관리하고 있다.**

이런 일관성없는 메커니즘은 메모리를 불필요하게 낭비하지 않는 이점이 있다. 하지만 일관성이 없으면 꼭 주의할 점이 생긴다.

### 주의
만약 `append()` 함수로 변경된 슬라이스의 값을 수정한다고 가정해보자.

```go
...

func main() {
    a := []int{1, 2}
    printSliceInfo(a)

    b := append(a, 3)
    b[0] = 7
    printSliceInfo(a)

    c := append(b, 4)
    c[1] = 7
    printSliceInfo(b)
}

...
/*
len=2 cap=2 slice=[1 2] addr=0xc000090010
len=3 cap=4 slice=[7 2 3] addr=0xc000094020
len=4 cap=4 slice=[7 7 3 4] addr=0xc000094020
*/
```

결과에서 보면 a와 b는 다른 주소를 가지고 있어서 b의 값을 수정해도 a에게 영향을 주지 않는다.

반면 b와 c는 같은 주소를 가지고 있어서 c의 값이 변경될 때 b의 값도 달라진다.

이런 상황은 배열의 길이가 매우 유동적인 상황일 때 예상하지 못한 버그를 만들어내기 쉽다.

따라서 안전하게 **append() 리턴 값은 항상 append() 하는 대상에 재할당해서 쓰자**

```go
func main() {
    var a []int
    a = append(a, 1, 2)
}
```

이렇게 말이다.