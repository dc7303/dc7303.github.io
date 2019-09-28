---
layout: post
title: "[JS]호이스팅과 스코프"
subtitle: 자바스크립트의 스코프는 특별하다
date: 2019-08-14 01:00:00
background: '/img/posts/06.jpg'
category: javascript
tag: javscript
comments: true
---
### 스코프란?
프로그램이 실행 중일 때 실행 컨텍스트에서 변수에 도달할 수 있는 범위를 **스코프(Scope)**라고 합니다. 

자바스크립트를 제대로 이해하려면 이 스코프가 매우 중요하기 때문에 잘 이해하고 사용하는 것이 중요합니다.

### 정적 스코프(Lexical Scope)
정적 스코프(Lexical Scope)란 함수를 정의할 때 존재하는 변수에만 접근할 수 있는 것을 의미합니다.

```js
const name = 'Mark';
function user() {
  console.log(`name = ${name}`);
  console.log(`age = ${age}`);
}

{
  const age = 20;
  user();
}

/**
 * OUTPUT:
 * name = Mark
 * ReferenceError: age is not defined
 */
```
*코드 예제를 나타낼 때 중괄호만 사용하여 블록을 구현하였으나 실제로 자바스크립트에서 이렇게 코드를 작성할 일은 없습니다. 스코프를 표현하기에 좋은 방법이기 때문에 중괄호만 사용한 것이고 실제로 if, for, while 등으로 이해하시면 됩니다*

위 코드에서 보면 전역 범위에 변수 name과 user 함수를 정의합니다. 그리고 블록 내부에서 age를 선언한 뒤 user 함수를 호출합니다. 이때 결과는 age를 찾을 수 없다고 합니다.

이렇게 함수를 정의할 때 존재하는 변수에만 접근 가능한 것을 정적 스코프라고 합니다. 자바스크립트에서 정적 스코프는 전역 스코프(Global Scope), 블록 스코프(Block Scope), 함수 스코프(Function Scope)의 범위를 가지고 있습니다.

### 전역 스코프(Global Scope)
코드 내 모든 스코프에서 도달할 수 있는 스코프입니다. 익숙한 **전역 변수**가 전역 스코프를 가집니다. 전역 스코프는 모든 영역에서 도달할 수 있는 범위 특성상 높은 의존성을 가질 수 있습니다.

만약 프로그래머의 실수로 변경되지 말아야 할 때 이 변수가 변경된다면, 의존하고 있던 다른 프로세스에 영향을 줄 수 있어 변경되지 않을 값이 아니면 사용을 권장하지 않습니다.

### 블록 스코프(Block Scope)
중괄호로 묶인 범위를 블록 스코프라고 합니다. ES6에서 추가된 let과 const는 블록 스코프로 선언됩니다.

```js
const x = 1;
{
  const y = 0;
}
console.log(`x = ${x}`);
console.log(`y = ${y}`);

/**
 * OUTPUT:
 * x = 1
 * ReferenceError: y is not defined
 */
```

위 코드와 같이 전역 중괄호 내에 선언된 y를 블록 밖에서 사용할 때 에러가 발생합니다. 이렇게 블록 스코프는 블록 범위 안에서만 도달할 수 있습니다.  

다른 프로그래밍 언어에 익숙하신 분들은 당연한 이야기를 하는 것 같지만, 자바스크립트는 let과 const가 추가되어 블록 스코프 구현이 쉬워 진 것입니다.

var 변수만 존재할 때와 ES6의 개선된 let과 const가 추가된 현재를 비교했을 때, 혁명적이라고 해도 과언이 아니기 때문에 let과 const의 블록 범위를 아는 것이 중요합니다

이런 자바스크립트의 변화에 대해서는 호이스팅 개념을 이해하면 알 수 있습니다.

### 함수 스코프(Function Scope)을 알기 전 호이스팅(Hoisting)을 알자
함수 스코프를 이해하기 이전에 **호이스팅(Hoisting)** 개념을 먼저 이해하셔야 합니다. 위에서 언급했듯이 let과 const의 블록스코프가 왜 중요한지도 알 수 있습니다.

```js
console.log(x);

let x = 1;
/**
 * OUTPUT:
 * ReferenceError: x is not defined
 */
```

위 코드 예제는 대부분의 프로그래밍 언어와 같이 동작합니다. x가 선언되기 이전에 x를 호출하게 되면 에러가 발생하죠. 하지만 아래 코드는 다릅니다.

```js
console.log(x);

var x = 1;
/**
 * OUTPUT:
 * undefined
 */
```

예상과 다르게 undefined가 출력되는 걸 확인할 수 있습니다. 이유는 위 예제는 아래 예제와 같기 때문입니다.

```js
var x;
console.log(x);
x = 1;
```

예제에서 볼 수 있듯이 호이스팅이란 개념은 변수가 선언될 때 최상단으로 끌어 올려지는 것입니다.

호이스팅은 블록 안에서도 적용됩니다.

```js
console.log(x);
{
  var x = 1;
}
/**
 * OUTPUT:
 * undefined
 */
```

좀 더 복잡한 예제를 보겠습니다.

```js
for (var i = 1; i <= 5; i++) {
  setTimeout(function() {
    console.log(i);
  }, i * 1000);
}
/**
 * OUTPUT:
 * 6
 * 6
 * 6
 * 6
 * 6 
 */
```

위 코드는 호이스팅 개념을 모르고 있는 개발자라면 1 2 3 4 5이 출력될 것으로 생각할 것입니다. 하지만 i변수는 호이스팅 되어 전역 변수이기 때문에 setTimeout 내부 콜백 함수에서는 for문으로 증가한 전역 변수 i를 참조하게 되어 위와 같은 출력을 하게 됩니다. 

예제들에서 볼 수 있듯이 자바스크립트는 실행 시점에 코드 전체적으로 var 변수를 찾아 호이스팅합니다.

그리고 호이스팅은 함수 선언에도 적용됩니다. (자바스크립트에서 함수도 변수입니다)

```js
print('Hello World');

function print(str) {
  console.log(str);
}
/**
 * OUTPUT:
 * Hello World
 */
```


### 함수 스코프(Function Scope)
호이스팅을 이해하면 var가 가지는 함수 스코프를 이해할 수 있습니다. 

```js
console.log(`x = ${x}`)
console.log(`y = ${y}`);

var x = 0;

function foo() {
  var y = 1;
}
/**
 * OUTPUT:
 * x = undefined
 * ReferenceError: y is not defined
 */
```

위 코드 예제를 보면 x는 호이스팅 되어 undefined가 출력되지만, y를 출력하는 시점에 에러가 발생합니다. 코드와 같이 var y는 함수 내부에 선언되어 있고 y는 함수 스코프(Function Scope)를 가지고 있기 때문입니다.

쉽게 정리해보면 var 변수는 실행 시점에 호이스팅 되어 지는데, 함수 스코프를 가진 var 변수는 함수 내부에만 존재하는 것입니다.

### 마무리하며
호이스팅과 스코프들을 이해하고 나면 var는 다른 프로그래밍 언어의 변수와 다르다는 걸 확인할 수 있습니다. 이런 문제를 해결하기 위해 IIFE, 클로저, Strict Mode 등과 같은 기술을 활용한 코딩이 필요했고, 이렇게 작성된 코드는 그 형태가 아름답지는 않았습니다.

하지만 현재 ES6이 대중화된 시점에 var 변수를 사용하면서 겪은 문제들을 신경쓰지 않고 다른 프로그래밍 언어와 유사하게 프로그래밍 할 수 있도록 개선되었습니다.

실제로 많은 자바스크립트 코딩 가이드에서 var 변수를 사용하지 않는 것을 권장하고 있고 let과 const만을 사용할 것을 권장합니다. (필자는 [airbnb/javascript Style Guide](https://github.com/airbnb/javascript)를 선호합니다)

글을 작성하면서 생각해보니 var를 써본 기억이 처음 자바스크립트를 공부할 때 말고는 없군요.

그래도 이런 개념을 알고 있어야 더 고급스러운 자바스크립트 기술을 구현할 수 있기 때문에 꼭 알아야 한다고 생각합니다.