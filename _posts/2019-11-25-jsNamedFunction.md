---
layout: post
title: "[JS] 함수에서 자기참조할 때 유명(named)함수를 사용하자"
date: 2019-11-25 10:00:00
category:
- javascript
tag:
- javascript
comments: true
---

함수 내부에서 함수 자기 자신을 참조할 때 유명함수를 사용해보자.

### 유명(named)함수란?
이름 그대로 이름이 존재하는 함수다. 자바스크립트의 익명함수와 반대 개념이다.

```js
const func = function f() {
    console.log(f);
};

func.name;
// OUTPUT:
// "f"
```

`f`처럼 이름을 지은 함수를 유명함수라고 한다.  

### 자기 참조
먼저 익명 함수에서 함수가 자기 자신을 참조하면 아래와 같다.

```js
let func = function() {
    console.log(func);
};

func();
func.name;
// OUTPUT:
//
// f () {
//     console.log(func);
// }
//
// "func"
}
```

위와 같이 자기 참조할 경우 변수 값이 변경되었을 때 문제가 발생한다.

```js
let func2 = func;
func = 'changed value';

func2();
func2.name;
// OUTPUT:
// changed value
// "func"
```

이렇게 변경되면 func 자기 자신을 참조하고 있던 함수는 없어진다. func는 변경되었고 func2는 func를 바라보기 때문이다. 더 이상 자기 참조하고 있는 함수를 사용할 수 없다.

이런 문제를 유명함수가 해결해줄 수 있다.

```js
let func = function f() {
    console.log(f);
};
func();
func.name;

// OUTPUT:
//
// f() {
//     console.log(f);
// }
//
// "f"

let func2 = func;
func = null;
func2();
func2.name;

// OUTPUT:
//
// f() {
//     console.log(f);
// }
//
// "f"
```

name값은 func와 func2 모두 `f`다. func와 func2는 유명함수 `f`를 바라보고 있기 때문에 func가 변경되도 `f`자기 자신을 참조할 수 있다.

그리고 또 하나의 안전성을 보장하는 것은 유명함수의 이름은 내부 스코프에서만 참조 가능하다는 점이다.
```js
const a = function f() {
	console.log(f);
}
console.log(f);

// OUTPUT:
// VM839:5 Uncaught ReferenceError: f is not defined
//    at <anonymous>:5:13
// (anonymous) @ VM839:5
```

내부 스코프에서만 접근할 수 있기 때문에 매우 안정적으로 자기 참조가 가능하다.

### arguments.callee 대신 사용하자
arguments.callee는 현재 실행중인 함수를 나타낸다. 익명함수 내부에서 자기 자신을 호출할때 유용하게 사용할 수 있다.

하지만 [MDN arguments.callee](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Functions/arguments/callee) 문서를 보면 ES5 엄격모드에서 사용할 수 없다는 것을 알 수 있다. 이유는 문서에 매우 자세히 설명되어 있다.

그리고 문서에서는 이름을 주거나 함수 자체를 호출해야 하는 곳에 함수 선언을 사용하여 arguments.callee() 사용을 피하라고 지침하고 있다.

이점을 인지하고 callee 대신 유명함수를 사용하는 습관을 들이자.

### Reference
- [MDN 함수 표현식](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/function)
- [유인동님의 '함수형 자바스크립트 프로그래밍'](http://www.yes24.com/Product/Goods/56885507)