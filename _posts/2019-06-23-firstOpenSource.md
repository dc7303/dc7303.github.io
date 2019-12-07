---
layout: post
title: "나의 첫 오픈 소스기여"
date: 2019-06-23 15:00:00
category:
- essay
tag:
- essay
comments: true
---

취업 전 프로그래밍을 공부할 때부터 지금까지도 오픈 소스를 매우 좋아합니다. 평소 취미로 오픈 소스를 분석하면서 보내기도 하니까요.  
그리고 오픈 소스를 좋아하는 제가 현업 첫 프로젝트에서 사용하게 된 오픈 소스에서 문제가 발생하였고, 이 문제를 해결하기 위해 오픈 소스에 기여하게 된 일화를 기록했습니다.  


### 오픈소스에 기여하다
오픈소스에 기여한 경험에 관해 이야기를 공유하고 싶습니다. 평소 오픈소스에 관심이 많고 남의 코드를 분석하는 데 흥미를 느끼는 저에게 신입 개발자 입사 초기 오픈소스 기여는 매우 흥미롭고 설레는 경험이었습니다.

그래서 오픈소스에 대해 어려움을 느끼는 개발자분이 있으시다면 이 글을 읽고 시작이 어렵지 않다는 걸 느꼈으면 하는 바람으로 글을 작성하게 됐습니다.

오픈소스에 기여하게 된 계기는 초기 웹 프로젝트 진행에 앞서 게시판에 에디터를 적용할 때입니다. 입사 전 함께 공부하는 친구들과 웹프로젝트를 진행할 때 TOAST에서 제공하는 무료 오픈소스인 tui-editor를 사용해본 경험이 있었고, 사용된 부트스트랩 버전이 현 프로젝트와 호환이 가능했기 때문에 이를 사용하기로 했습니다. 그리고 앵귤러를 사용하고 있는 환경에 좀 더 효율적이고 쉽게 적용할 수 있는 방법이 있지 않을까 검색하던 중 [ngx-tui-editor](https://github.com/tylernhoward/ngx-tui-editor)를 찾게됩니다.

[ngx-tui-editor](https://github.com/tylernhoward/ngx-tui-editor)는 tui-editor를 앵귤러 환경에서 간단하게 적용할 수 있도록 어댑터 역할을 해주는 오픈소스로 현 프로젝트에서 쉽게 빌드해서 사용할 수 라이브러리입니다.  

이 라이브러리를 프로젝트에 적용하는 중 몇가지 이슈가 발생했습니다. 

### 초장부터 에러냐?
오픈소스를 설치하고 컴포넌트에 적용하여 serve를 시도했지만 트랜스파일링 단계에서 에러가 발생합니다.

![consoleErr](/assets/images/post/ngx-error.png){: width="100%"}*\<콘솔 에러 메세지\>*

위와같이 TuiEditor 타입을 찾을 수 없다는 것이였습니다. 저는 바로 에러에서 알려주는 경로로 들어가 원인을 분석했습니다. 

![codeErr](/assets/images/post/ngx-code.png){: width="100%"}*\<에러의 원인인 라이브러리 내부의 index.ts 파일\>*

프로젝트 index.ts파일에서 발생했습니다. 제 추측으로는 개발자가 tui-editor를 import 하는 과정에서 TuiEditor객체를 타입으로 선언한게 문제를 일으킨 것 같습니다. 

설치된 ngx-tui-editor에서 tui-editor선언할 때는 이미 자바스크립트로 트랜스파일링된 tui-editor의 index.js를 참조하고 있습니다. 따라서 타입스크립트에서 TuiEditor 객체를 타입으로 선언할 수 없던 것이라고 추측합니다. 

저는 이 문제의 타입을 any로 수정하여 간단하게 문제를 해결했습니다.

### 개발자에게 수정을 요청하자
문제는 저만 수정한다고 되는 것이 아니었습니다. node_modules폴더가 ignore되어 버전 관리가 되는 상황에서 협업하고 있는 개발자들이 프로젝트 셋팅할때 마다 npm install을 해주어야 하는데, 이슈 해결을 위한 문서가 있다해도 모두가 똑같은 수정을 하는 작업은 매우 불필요한 리소스 낭비라고 생각했습니다.

저는 같은 일이 반복되는 상황이 싫었기 때문에 Pull Request를 해보자고 결정했습니다. 제가 수정한 부분으로 인해 발생되는 이슈가 있는지 세밀하게 분석했습니다.(Pull Reqest시점에 확인해보니 이 이슈는 누군가 글로 알렸지만, 개발자는 Pull Request를 요청하더군요.)

수정 후 문제는 없었지만 우리의 프로젝트에 적용하는데 있어 또 다른 이슈가 발생합니다.

### 에디터만 있고 뷰어는..?
tui-editor를 쓰기로 한 이상 viewer 또한 TOAST에서 제공하는 오픈소스를 적용하고 싶었습니다. 하지만 ngx-tui-editor는 tui-viewer를 불러오는 기능은 구현하지 않았더라고요.. 하.....


![tuiCodeJs](/assets/images/post/crying.jpg){: width="100%"}*\<좀 해놓지...\>*


그래서 제가 직접 그 기능을 간편하게 불러올 수 있도록 수정하기로 합니다. tui-editor에서 제공하는 document를 살펴보면서 TuiEditor의 viewer를 불러올 수 있는 static method인 factory() 메소드를 찾게됩니다. 그리고 ngx-tui-editor를 불러올때 옵션 플래그 값을 추가하여 factory()함수를 사용할 수 있도록 수정했습니다.  


```javascript
/**
 * 수정전 라이브러리 내부 객체를 생성하는 코드입니다.
 */

fucntion(options) {
  this.editor = new TuiEditor(Object.assign({
    el: document.querySelector('.ngx-tui-editor'),
    initialEditType: 'markdown',
    previewStyle: 'vertical',
    height: '300px',
  }, options));

  return this.editor;
}
```

```javascript
/**
 * 수정후 코드입니다.
 */
fucntion(options) {
  if (options.viewer) {
    options.el = document.querySelector('.ngx-tui-editor');
    options.height = '300px';
    this.editor = TuiEditor.factory(options);
  } else {
    this.editor = new TuiEditor(Object.assign({
      el: document.querySelector('.ngx-tui-editor'),
      initialEditType: 'markdown',
      previewStyle: 'vertical',
      height: '300px',
    }, options));
  }
  
  return this.editor;
}
```

수정 후 프로젝트에서 tui-viewer를 렌더링할 때 굉장히 뿌듯했습니다. 얼굴 한번 본적 없는 개발자의 프로젝트를 제가 직접 더 효율적으로 사용할 수 있게 수정했다는 게 신입인 저로서는 제 자신이 대견했습니다. 

이렇게 수정한 후 직접 추가한 코드로 인해 얻는 이점이 많았기 때문에 이 변경사항도 Pull request 해보자고 생각했습니다. 

입사한 지 안된 신입개발자에겐 기대보다 걱정이 컸습니다. 이유는 내가 짠 코드가 마음에 안 들면 어쩌지? 불필요한 기능인가? 더 큰 버그를 만들어내면 어쩌지? 등 아직 제 코드에 대한 자신감이 없을 시기였기 때문인 것 같습니다. 

### 첫 Pull Request 시도하다
![pullReq](/assets/images/post/pull-req.png){: width="100%"}*\<Pull Request 내용 [(링크)](https://github.com/tylernhoward/ngx-tui-editor/pull/9)\>*


![pullReqCode](/assets/images/post/pull-req-code-change.png){: width="100%"}*\<변경된 코드 [(링크)](https://github.com/tylernhoward/ngx-tui-editor/commit/b2947794388eefed7080847b7bc013944b4e85e0)\>*

이미지에서 보면 제게 얼마나 조심스러운 일이었는지 알 수 있습니다. Pull Request 글에서 보면 'And before you review this, please understand my lack of English.'라고 수줍게 부족한 영어 실력을 이해해달라는 쓸데없는 소리를 하는 걸 보면 느끼실 겁니다.

### 반영되다
![pullReqCommit](/assets/images/post/pull-req-commit-img.png){: width="100%"}*\<커밋된 제 코드 [(링크)](https://github.com/tylernhoward/ngx-tui-editor/commits/master)\>*

이틀을 기다리니 제 요청이 반영된 걸 확인할 수 있었습니다. 지금 생각하면 별거 아니지만 이때는 이 세상에 엄청난 기여를 한 것 같은 기분이었습니다.

### 마무리하며
지금 생각해보면 대단한 일도 아닌데 처음이라는 게 큰 의미로 다가왔던 것 같습니다. 그때의 저를 생각하면 대견스럽고 뿌듯합니다. 이 일을 통해 지금은 제 코드뿐만 아니라 누군가의 코드를 분석하고 도움이 될 때 큰 즐거움을 느끼고 있습니다. 원래 재밌었던 개발이 다른 부분에서도 재미를 찾게 된 것이죠.

이 경험은 운이 좋았던 것도 있습니다. 상황이 오픈 소스에 기여한다는 부담 없이 현 프로젝트에 최적화되도록 수정하고 싶다는 마음으로 접근했으니까요. 이런 마음가짐 덕분에 자연스럽게 시도했던 것 같습니다. 

그리고 첫 오픈소스 분석에 알맞은 난이도의 프로젝트인 것도 행운이었던 것 같습니다. 만약 처음부터 스프링을 분석해서 기능을 추가하는 미션이었다면 이렇게 즐겁게 끝나진 않았을 것 같습니다. 

어려웠던 점은 다른 곳에서 나타났습니다. 영어 해석은 되지만 직접 작성하려니 오히려 요청문 작성에서 큰 어려움에 부딪혔습니다. 요청문을 작성하는데 '이 문장이 우스워 보이지 않을까?'라는 고민에 많은 리소스를 낭비했으니까요. 하지만 번역기 덕분에 이 문제를 해결할 수 있었습니다.

이 경험을 통해 현재의 저는 좋아하는 오픈소스를 분석하면서 많은 걸 배우고 있습니다. 이제는 어려운 숙제라기보다는 재미있는 놀이이자 취미가 된 것이죠.

만약 오픈소스에 기여한 경험이 없거나 두려우신 개발자라면 꼭 평소에 관심 있던 오픈소스를 Fork해서 분석해보라고 권해드리고 싶습니다. 우리 프로그래머에게 분석, 문제해결, 기능추가 같은 일들은 늘 있는 일이고 오픈소스에 기여한다는 건 이것과 같은 일이니까요. 저와 같은 경험을 해보셨으면 좋겠습니다.
