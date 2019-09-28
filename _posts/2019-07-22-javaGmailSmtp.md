---
layout: post
title: "[Java]smtp 배포서버에서 동작을 안한다면?"
subtitle: 로컬에서 작동했는데.. 억울하다..
date: 2019-07-22 10:00:00
background: '/img/posts/06.jpg'
category: java
tag: java, smtp,
comments: true
---
### 문서의 목적
현재 진행하고 있는 프로젝트에서 고객에게 메일링을 하는 프로세스가 존재하기 때문에 google smtp를 통해 기능을 구현했습니다. 

그리고 이 기능은 로컬서버에서 잘 동작했지만 테스트 서버에 빌드 후 테스트해보니 제대로 동작하지 않는 이슈가 발생했습니다. 

결론부터 말씀드리면 google 계정에서 발생한 문제였습니다. smtp 구현 후 서버환경이 변할때 누구나 한번쯤 나와 같은 고생을 할 수 있다는 생각에 글을 작성하게 됐습니다.

### MailAuthenticationExeption
메일전송이 실패하여 서버쪽 로그를 확인해보니.. 아래 이미지와 같이 MailAuthenticationExeption이 발생했습니다.

![mailAuthFail](/img/posts/java/mail-authentication-exception.png){: width="100%" }*\<MailAuthenticationException\>*

문제 해결을 위해 로그에서 안내하는 [(https://support.google.com/mail/answer/78754)](https://support.google.com/mail/answer/78754)에 접속해봅니다.

![googleMailAnswer](/img/posts/java/google-mail-answer.png){: width="100%" }*\<구글 클라이언트 접속 문제 가이드\>*

접속해보면 이메일 클라이언트 접속이 안될때 가이드해주는 문서가 나타납니다. 개발중 MailAuthenticationException발생하면 이 가이드를 따라가면 좋을 것 같습니다.

보통 MailAuthenticationException을 검색해보면 2단계 3번째에 '보안 수준이 낮은 앱'을 허용하지 않아서 이슈가 발생하는 사례가 많은 것 같습니다. 하지만 저는 보안 수준이 낮은 앱을 허용하고 있는 상태였고, 다른곳에서 원인을 찾았습니다.

### 문제해결

![authBlock](/img/posts/java/auth-block.png){: width="100%" }*\<차단 당했구나..\>*

구글 계정을 살펴보던 중 최근 발생한 이벤트가 있길래 확인해보니 IDC 주소의 테스트 시간대 정보가... 

원인은 이렇게 IDC에서 계정에 접근할때 구글에서 차단한 것에서 나타났습니다. 이미지는 내가 접근하게 맞다고 확인해준 뒤 촬영하여 선택하는 버튼이 없지만 '예', '아니오'로 접근을 허용할 수 있습니다.

google 뿐만 아니라 많은 email을 제공하는 업체들이 보안수준이 높은 것으로 알고 있습니다. smtp 구현시 먼저 브라우저에서 계정 보안 환경을 파악하는 것도 중요한 것 같습니다.