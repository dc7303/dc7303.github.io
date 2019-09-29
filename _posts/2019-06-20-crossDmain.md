---
layout: post
title: "[Angular]CORS 이슈"
date: 2019-06-20 10:00:00
category: 
- javascript
tag:
- CORS
- Angular
comments: true
---

Angular 7 + Spring 4.x 프론트, 백엔드 각각 프레임워크 환경에서 개발단계에 PT서버, BT서버 두개를 띄우고 개발을 시작했습니다. 이는 Angular에서 HTTP 요청을 보냈을때 CrossDomain이슈를 발생시켰고 이를 해결하기 위해 Header에 몇가지 셋팅이 필요했습니다. 이는 Angular만이 아니라 Javascript로 개발하다보면 자주 부딪히는 문제임으로 같은 환경에 처해있는 분들은 이글에서 힌트를 얻어 문제의 본질을 파악하고 해결했으면 좋겠습니다.


## CORS란?
브라우저들은 동일 출처 정책(Same Origin Policy)의 이유로 동일한 도메인으로 HTTP요청을 할 수 없었지만, 이러한 환경에서 접근 제어권을 부여하기 위해 사용되는 표준입니다. 쉽게 설명하면 다른 도메인 환경에서 리소스를 사용할 수 있도록합니다.  
여기서 다른 도메인에서 왜 필요한가를 생각해볼 수 있습니다. 때로는 하나의 도메인에서 모든 리소스를 처리하기에 효율성문제로 각 기능별로 여러 서버를 사용할 수 있습니다. 이 분리된 서버(도메인)에 접근하기 위해서 꼭 필요한 정책입니다.

## 문제발생
제가 발생했던 문제는 위에서 문서 초반에 말씀드렸듯 Angular + Spring 환경에서 나타났습니다. Angular 개발단계에서 Angular의 독립된 서버환경을 작동시켜 개발합니다. 이때 게시판 조회를 예를 들면 DB에 접근하기 위해서 JAVA에 HTTP 요청을 보내게 됩니다. 이때 리소스를 가져오는데 실패하고 개발자도구 콘솔에는 아래와 같은 메세지를 푸쉬합니다. 필자는 크롬을 사용하였고 브라우저마다 메세지가 다를 수 있습니다.
~~~ browser
Use Browser: Chrome

Access to XMLHttpRequest at 'http://localhost:8080/board/select'
from origin 'http://localhost:4200' has been blocked by CORS
policy: No 'Access-Control-Allow-Origin' header is present on the
requested resource.
~~~

## 문제 해결 방법
이를 해결하는 방법은 어렵지 않습니다. 문제 해결의 핵심은 CORS정책이고 이는 HTTP Header를 통해 요청하는 도메인의 출처를 확인하고 허용하는 방식으로 진행됩니다. 이러한 해결방식은 MDN에서 간단한 요청, 사전 요청, 인증을 이용한 요청 3가지를 제공합니다.*[(MDN HTTP 접근제어)](https://developer.mozilla.org/ko/docs/Web/HTTP/Access_control_CORS)* 

제가 선택했던 방식은 인증을 이용한 요청방법(Request with Credential)이며 아래 코드는 Angular + Spring 코드 예제입니다. 프론트 단에서 angular에서 제공하는 interceptor를 통해 request를 가로채 withCredentials를 헤더에 추가해주었습니다.  Angular가 낯선분이라도 핵심은 withCredentials: true를 헤더에 추가한다는 것에 있습니다.
~~~typescript
// httpConfig.interceptor.ts

@Injectable()
export class HttpConfigInterceptor implements HttpInterceptor {
  constructor() {}

  intercept(request: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    request = request.clone({
      withCredentials: true
    });

    return next.handle(request);
  }
}
~~~
  
위와 같이 RequestHeader에 자격증명을 한 뒤, 서버에서 또한 ResponseHeader에 대한 작업이 필요합니다. 서버쪽 헤더작업은 필터에서 작업했습니다.  
```java
/** CrossDomainFilter.java */

@Override
public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) throws Exception {
  HttpServletResponse response = (HttpServletResponse) res;
  HttpServletRequest request = (HttpServletRequest) req;

  response.setHeader("Access-Control-Allow-Origin", request.getHeader("origin")); // 허용 URL
  response.setHeader("Access-Control-Allow-Credentials", "true");   // 자격증명
  response.setHeader("Access-Control-Allow-Methods", "POST, GET, DELETE, PUT"); // 허용 메소드
  response.setHeader("Access-Control-Max-Age", "3600");   // 브라우저 캐시 시간
  response.setHeader("Access-Control-Allow-Headers", "X-Requested-With, content-type, enctype");    // 허용 헤더

  chain.doFilter(req, req);
  
}
```
이렇게 서버에서 ResponseHeader를 셋팅해줍니다. 제가 사용한 인증방식(Request With Credential)에서 중요한 점은 Access-Control-Allow-Origin의 값에 '\*' 허용하지 않는점과 Access-Control-Allow-Credentials를 꼭 true로 해주어야 했습니다.


Access-Control-Allow-Origin에 request.getHeader("origin")을 셋팅한 이유는 개발단계에서 '\*'와 같은 동작을 하기 위해서이며, 다른 방법은 직접명시하는 것입니다. (배포포할때는 허용하는 도메인을 직접 명시하는 것이 좋겠죠?)


Access-Control-Allow-Credentials은 브라우저 기본동작이 credentials false로 동작하기 때문에 꼭 true를 명시해줘야합니다. 만약 이 헤더를 가지고 있지 않다면 브라우저는 이 응답을 거절할 것입니다.

이렇게 CORS정책을 사용하여 CrossDomain 이슈를 해결할 수 있었습니다. 다른 방법을 사용하지 않고 인증방식(Request With Credential)을 사용하게된 이유는 서버쪽 쿠키를 사용하기 위해서는 입니다. 크로스 도메인 상태에서 쿠키에 대한 리소스 교류가 이루어지려면 withCredentials가 필요합니다. 

나머지 간단한 요청과 사전 요청 방식도 핵심은 비슷하니 MDN문서*[(MDN HTTP 접근제어)](https://developer.mozilla.org/ko/docs/Web/HTTP/Access_control_CORS)*에서 확인해보시고 프로젝트 환경에 적합한 방식을 사용하면 되겠습니다.

---

## ResponseHeader 정리

##### Access-Control-Allow-Origin 
- 요청 도메인의 URI를 지정한다. 
- 와이드카드(\*)를 사용할 수 있다
- 인증방식(Request With Credential)에서는 와이드카드(\*)를 사용할 수 없다.

##### Access-Control-Expose-Headers
- 기본적으로 브라우저에게 노출이 되지 않지만, 브라우저 측에서 접근할 수 있게 허용해주는 헤더를 지정한다.

##### Access-Control-Max-Age
- 사전요청(Preflight Request)의 결과가 캐쉬에 남아있는지를 나타낸다.

##### Access-Control-Allow-Credentials
- 인증방식(Request with Credential)을 사용할 수 있는지를 지정한다.
- true가 명시되어 있지 않다면, 브라우저는 해당 응답을 거절한다.

##### Access-Control-Allow-Methods
- 서버 리소스에 접근할 수 있는 HTTP Method 방식을 지정한다.

##### Access-Control-Allow-Headers
- 요청에서 사용할 수 있는 HTTP Header를 지정한다.
