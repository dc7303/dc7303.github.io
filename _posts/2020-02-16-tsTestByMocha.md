---
layout: post
title: "타입스크립트와 모카 사용하기"
date: 2020-02-16 02:00:00
category: 
- typescript
- test
- mocha
tag: 
- typescript
- test
- mocha
comments: true
---

타입스크립트와 모카를 함께 사용하는 방법에 대해 정리한다.

### 타입스크립트
먼저 타입스크립트를 설치한다. 글로벌로 설치해서 사용해도 되지만, 개인적으로 프로젝트 내부에 설치하는 걸 선호한다.

```bash
$ yarn add typescript --dev
```

그리고 package.json과 같은 레벨에 tsconfig.json을 만들어준다.

```json
/* tsconfig.json */
{
  "compilerOptions": {
    "module": "CommonJS",
    "sourceMap": true,
    "target": "ES2015",
    "outDir": "dist"
  },
  "include": [
    "src/**/*"
  ],
  "exclude": [
    "node_modules",
    "**/*.spec.ts"
  ]
```

위 설정은 예제를 위한 간단한 설정값들이다. 타입스크립트 자체를 다루는 글이 아니기 때문에 자세한 설명은 생략한다.  
([타입스크립트 홈페이지](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html)를 읽어보면 좋다)

준비가 끝났으면 `src/index.ts`를 만들고 코드를 작성해보자.

```ts
/* index.ts */

enum Color {Red, Yellow, Green};
const c: Color = Color.Red;
console.log(c);
```

그리고 컴파일이 정상적으로 동작하는지 확인해보자.

```bash
$ tsc
```

### 모카
[모카](https://mochajs.org/)는 노드JS에서 많이 사용되는 테스트 프레임워크다. 특징은 assertion을 지원하지 않는다. 그래서 노드JS에서 제공하는 `assert` 모듈을 사용하거나 다른 Assertion 라이브러리를 설치해서 사용한다.

모카를 설치한다.

```bash
$ yarn add mocha @types/mocha --dev
```

아, 여기서 깜빡이도 없이 `@types/mocha`를 함께 설치했다. 모카 프레임워크에 타입을 정의한 라이브러리이며, 타입스크립트와 모카를 함께 사용할 때 필요하다.

앞에서 말했듯이 모카는 assertion을 제공하지 않기 때문에 [chai](https://www.chaijs.com/)를 설치해주자. 모카와 같이 `@types/chai`도 설치해준다. 이유는 같다.

```bash
$ yarn add chai @types/chai --dev
```

기본적으로 노드JS에서 제공하는 `assert` 모듈을 사용해도 되지만, `chai`를 선호한다. 이유는 BDD 기반의 프레임워크라고 거창한 이유를 말할 수도 있겠지만, 사실 필자는 `chai`가 채이닝 방식으로 테스트할 수 있어서 좋다.


### 테스트
먼저 `src/index.ts` 파일에 사용할 함수를 선언한다.

```ts
/* index.ts */

export function sum(x: number, y: number): number {
  return x + y;
}
```

그리고 `src/test/index.spec.ts` 파일을 생성한 후 테스트 코드를 작성한다.
```ts
import { expect } from 'chai';
import { sum } from '../src/index';

describe('index.ts 테스트', () => {
  it('sum 테스트', () => {
    expect(sum(1, 1)).to.equal(2);
    expect(sum(1, 1)).to.not.equal('귀요미');
  });
});
```

그리고 `package.json`에 테스트 실행 스크립트를 작성해준다.

```json
{
  "scripts": {
    "test": "yarn mocha test/**/*.ts",
  },
}
```

테스트를 돌려보자.

```bash
$ yarn test
```

성공 메시지를 기대했지만, 컴파일 자체가 안된다. 필자의 컴퓨터에서는 아래의 에러 메시지를 확인할 수 있었다.

```bash
yarn run v1.21.1
$ yarn mocha test/**/*.spec.ts
$ /Users/choedongcheol/Workspace/exam/node_modules/.bin/mocha 'test/**/*.spec.ts'
/Users/choedongcheol/Workspace/exam/test/index.spec.ts:1
import { sum } from '../src/index';
^^^^^^

SyntaxError: Cannot use import statement outside a module
```

타입스크립트는 컴파일이 되어야 실행이 가능하다. 근데 우리 개발자는 테스트할 때마다 컴파일하고, 컴파일된 녀석을 참조해서 테스트할 만큼 여유롭지 못하다.

결국 메모리상에서 타입스크립트를 트랜스 파일링하여 실행할 수 있게 해줘야 한다. `ts-node`라는 녀석을 설치해준다.


```bash
$ yarn add ts-node --dev
```

그리고 test를 실행할때 require 옵션을 추가한다

```json
"script": {
    "test": "yarn mocha test/**/*.ts -r ts-node/register",
}
```

자 다시 `yarn test`를 실행해보자.

```bash
  index.ts 테스트
    ✓ sum 테스트
```

기분 좋은 결과를 얻을 수 있다.