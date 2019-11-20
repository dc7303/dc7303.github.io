---
layout: post
title: "[Angular] 배포 후 IE 이슈 기록(feat. punycode.js)"
date: 2019-11-20 08:00:00
category: 
- javascript
tag:
- Angular
comments: true
---

앵귤러 프로젝트를 배포하였는데 IE 9-11 버전에서 초기 렌더링이 되지 않는 이슈가 발생했다. 빌드 설정에는 이상이 없었고, 예상하지 못한 곳에서 문제가 발생한 것이었다.
추후 다른 앵귤러 프로젝트에도 동일한 증상이 발생할 수 있는 문제이기 때문에 기록한다.

### 이슈 발생
페이지 배포 후 IE11에서 렌더링이 되지 않는 문제가 발생한다. 페이지 초기화조차 되지 않는다. 에러가 발생한 파일은 main.js이며, 코드를 따라가 보니 `const f=e=>String.fromCodePoint(...array);`라는 코드에서 문제가 발생했다.

IE9-11에 호환되도록 설정하고 빌드한 것이었기 때문에 화살표 함수가 있어서 의아했다. 함수의 원형이 궁금해서 optimzation 설정을 제거하고 재빌드했다.

optimization설정을 제거하고 다시 빌드하여 `const ucs2encode = array => return String.fromCodePoint(...array);`의 코드 원형을 볼 수 있었다.


### 원인
타입스크립트 컴파일옵션, browserlist, 앵귤러 옵션을 다시 리뷰해도 설정에는 문제가 없었다. 그래서 우리가 사용하고 있는 라이브러리를 의심해보았다.

그리고 라이브러리에서 원인을 찾을 수 있었다. 에러가 발생한 코드는 punycode.js라는 라이브러리에서 온 것이었다.

punycode.js는 우리가 직접 설치한 라이브러리가 아니고 scrollspy를 설치하면서 함께 온 친구로 보인다.(방갑다)

![punycodeDepth](/assets/images/post/punycodeDepth.png){: width="100%" }*\<라이브러리 뎁스 확인\>*

그리고 관련 검색을 해보니 punycode 버전 2.1.1이 문제였다. 이 녀석에서 사용한 화살표 함수가 여러 개발자를 괴롭힌 듯하다. (글 마지막 레퍼런스 참조)


### 개선
`npm install punycode@1.4.1`로 punycode 1.4.1 버전을 Root package에 설치하여 개선했다. 변경하고서 테스트 결과 모든 브라우저에서 이상 없이 동작하는 것을 확인했다.

punycode를 의존하고 있는 라이브러리를 찾아서 관련 이슈 존재 여부 및 개선 여부 확인 후 버전 업데이트를 고려해봐야겠다.


### Reference
- [punycode.js issues](https://github.com/bestiejs/punycode.js/issues/86)
- [angular issues](https://github.com/angular/angular/issues/21202)