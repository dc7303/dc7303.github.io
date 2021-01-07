---
layout: post
title: "클린 아키텍처 맛보기"
date: 2020-12-13 01:00:00
category: 
- clean architecture
- java
- spring
tag: 
- clean architecture
- java
- spring
comments: true
---

1년 동안 열심히 만든 프로젝트가 있다고 가정하자. 그리고 이 프로젝트는 테스트 자동화, 리팩토링, 아키텍처에 대한 고민없이 기능 개발에만 집중해 왔다. 어느덧 이 프로젝트를 통해 조직은 성장했고, 그 만큼 많은 개발 인력을 채용했다. 회사는 늘어난 인력만큼 개발팀의 높은 생산성을 기대할 것이다.  
하지만 개발팀의 인원이 늘어난 것과 반대로 생산성은 낮아지고 있다. 여기저기서 발생하는 오류, 스파게티 코드, 돌이킬 수 없이 강하게 묶여버린 의존관계 등과 같은 문제들 때문에 코드 하나 수정하기가 힘든 상태가 원인이다. 이를 해결하기 위해 기존 개발자들의 노력으로 겨우 안정을 찾지만, 새로운 비지니스에 대한 진도는 나가지 못했다. 그리고 회사는 또 개발자를 채용하여 이 현상을 반복한다.  
낮아진 생산성은 늘어난 프로젝트 규모와 복잡도 때문이라고 위안을 삼으며 기대하는 평균 생산성의 수치가 낮아지는 비극적인 사태에 이른다. 참 슬픈 일이다.


## 글을 쓰는 이유
<!-- 앞서 언급한 이야기는 극단적인 가설이다. 하지만 정도만 다를뿐 흔치않게 볼 수 있는 현상이다. 많은 솔루션 회사들이 테스트 자동화, 클린 아키텍처, 리팩토링, 코드리뷰 처럼 프로젝트 안정성에 관련된 기술과 문화를 사용하고 있다. 심지어 채용공고 요구사항에 포함되어 있는 걸 쉽게 볼 수 있고, 이런 지식들이 개발자 역량에서 중요한 부분을 차지하고 있다는 걸 알 수 있다.

비지니스와 마케팅에서 오는 요구사항을 처리하기 바빠죽겠는데, 왜 개발팀은 이런 것들에 리소스를 사용할까? 이유는 간단하다. 솔루션 회사는 프로젝트가 완성된 이후부터 진짜 시작이기 때문이다.  
저비용 고효율은 모든 산업에서 중요하게 다루는 문제다. 소프트웨어 개발이라고 예외는 아니다. 개발팀은 회사의 리소스를 효율적으로 쓰기 위해 고군분투해야할 의무가 있다.

이유는 알겠고, 뭐 부터 시작하면 좋을까? 이런 것들을 수행하기 위해 무엇부터 시작하면 좋을까? 사실 순서는 없다. 테스트, 리팩토링, 아키텍처, 코드리뷰 등 모든 것은 연결되어 있다고 봐도 된다.  

예 1)
1. 좋은 아키텍처를 유지하기 위해 지속적으로 리팩토링 해야한다.
2. 리팩토링을 위해 단위 테스트 코드가 필요하다.
3. 지속적으로 유지하려면 코드리뷰가 필요하다.

예 2)
1. 단위 테스트를 작성하고 싶다.
2. 단위 테스트를 작성하기 위해서는 기능의 역할을 잘 분리되서 리팩토링이 필요하다.
3. 테스트 범위를 잘 관리하고 중복 테스트를 예방하기 위해 아키텍처를 잘 설계하고 관리해야 한다.
4. 지속적으로 유지하려면 코드리뷰가 필요하다. -->




## 들어가기전에
### 글을 읽는 방법
클린 아키텍처는 개발자가 필수로 읽어야하는 책 중 하나라고 생각한다. 그래서 이 책을 많은 개발자가 읽었으면 하는 마음으로 맛보기 글을 작성한다.  


### 고수준과 저수준
아래 예제에서 저수준과 고수준이라는 단어가 나올 것이다. 수준의 경계를 이해하고 가는게 좋을 것이다.
클린 아키텍처에서 고수준과 저수준의 정도는 I/O와의 거리라고 말한다. I/O와 가까울 수록 저수준이고, 반대로 멀어질 수록 고수준인 것이다.  
항상 의존 방향은 저수준에서 고수준 방향으로 흘러야하며, 고수준은 저수준의 세부사항을 알면 안된다.

아래 이미지는 '클린 아키텍처' 저자인 밥 아저씨가 소개한 이미지로 컴포넌트 별 수준을 잘 나타내주고 있다. 이 이미지가 도움을 줄 수 있을 것이다. 원 안쪽으로 갈수록 고수준이다.

![cleanArchitecture](/assets/images/post/clean-architecture-sample-CleanArchitecture.jpg){: width="100%"}*\<[The Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)\>*






## 컴포넌트 경계가 없는 코드
게시글을 등록하는 API를 작성해보자. Controller, Service, Dao 클래스를 추가하고 그 안에 게시글 등록 로직을 작성한다.

```java
// Controller
@Controller
public class BoardController {
    private BoardService boardService;

	@PostMapping("/board")
	public ResponseEntity<String>  writeBoard(@RequestBody BoardVo board) {
        if (board.validation()) {
            ... // 실패 처리
        }
        int cnt = boardService.write(board);
        if (cnt == 0) {
            ... // 실패 처리
        }

        return ...
	}
}

// Service
@Service
public class BoardService {
    private BoardDao boardDao;

    public int write(BoardVo board) {
        return boardDao.write(board);
    }
}

// Repository
@Repository
public interface BoardDao {
    public int write(BoardVo board);
}
```

위 코드는 컨트롤러에서 비지니스 로직과 인터페이스 어댑터 역할 모두를 책임지고 있다. 또한 논리적으로 저수준과 고수준의 경계가 제대로 분리되지 않은 잘못된 코드다.  
이런 설계는 API 인터페이스를 수정할 때, 그리고 비지니스 로직을 수정할 때도 `BoardController`를 수정해야 한다. 컨트롤러의 책임이 너무 크다.


## 책임 분리하기
컨트롤러에는 인터페이스 어댑터 역할, 서비스에는 비지니스 로직을 배치해보자.

```java
// Controller
@Controller
public class BoardController {
    private BoardService boardService;

	@PostMapping("/board")
	public ResponseEntity<String>  writeBoard(@RequestBody BoardVo board) {
        try {
            boardService.write(board);
        } catch (...) {
            return ...
        }
	}
}

// Service
@Service
public class BoardService {
    private BoardDao boardDao;

    public void write(BoardVo board) throws Exception {
        if (board.validation()) {
            throw ...
        }
        int cnt = boardDao.write(board);
        if (cnt == 0) {
            throw ...
        }
    }
}

// Repository
@Repository
public interface BoardDao {
    public int write(BoardVo board);
}
```

컴포넌트의 책임이 명확해졌기 때문에 개발자는 하나의 컴포넌트에 집중할 수 있다. 

## 새로운 요구사항과 컴포넌트 추가
게시글이 등록될 때 메신저로 유저의 아이디와 게시글 제목을 전송하는 기능을 추가해보자. 해당 예제에서는 매터모스트를 사용한다고 가정한다.  
매터모스트 메시지를 전송하려면 JSON 스트링 값이 필요하다고 가정하고 코드를 작성하였다.

```java
// Service
@Service
public class BoardService {
    private BoardDao boardDao;
    private Mattermost mattermost;

    public void write(BoardVo board) throws Exception {
        if (board.validation()) {
            throw ...
        }
        int cnt = boardDao.write(board);
        if (cnt == 0) {
            throw ...
        }

        String jsonStr = "{\"title\":" + board.title + ",\"userId\":" + board.userId + "}";
        mattermost.send(jsonStr)
    }
}

// Mattermost
@Componenet
public class Mattermost {
    public void send(String jsonStr) {
        ...
    }
}
```

매터모스트 클래스를 추가했다. 그리고 `BoardService`에서 매터모스트가 필요한 데이터를 생성하여 전달했다. 이 코드는 문제없이 작동할 것이다.  
하지만 이 코드의 문제는 결합도가 높다는 것이다. 또한 Mattermost는 `External Interface`이기 때문에 저수준 컴포넌트다. 고수준 컴포넌트인 `BoardService`가 저수준에 의존하고 있으면 안된다.  
왜 이런 의존 관계를 문제라고 하는지는 유지보수 단계에서 알 수 있다. 이어서 살펴보자.

## 요구사항 변경
메신저를 슬랙으로 교체해야하고, 슬랙 API는 XML 데이터를 요구한다고 가정해보자.
여기서 의존 결합이 가져오는 문제가 발생한다.

```java
// Service
@Service
public class BoardService {
    private BoardDao boardDao;
    // private Mattermost mattermost;
    private Slack slack;

    public void write(BoardVo board) throws Exception {
        if (board.validation()) {
            throw ...
        }
        int cnt = boardDao.write(board);
        if (cnt == 0) {
            throw ...
        }

        // String jsonStr = "{\"title\":" + board.title + ",\"userId\":" + board.userId + "}";
        // mattermost.send(jsonStr)
        String xmlStr = "<employee><title>" + board.title + "</title><userId>" + board.userId + "</userId></employee>";
        slack.send(xmlStr);
    }
}

// Slack
@Componenet
public class Slack {
    public void send(String xmlStr) {
        ...
    }
}
```

메신저를 교체하기 위해 나는 `BoardService`를 수정하고 `Slack`을 추가했다. 

1. 메시전 전송 컴포넌트는 I/O와 가깝기 때문에 비교적 저수준 컴포넌트다. 근데 저수준 컴포넌트 변경이 고수준 컴포넌트인 서비스에 영향을 줬다.
2. 보드 서비스는 메신저의 책임과 게시판의 책임 모두를 가지고 있는 것으로 단일 책임 원칙(SRP)를 위배하고 있다.
3. 만약 슬랙에서 또 다른 메신저로 변경한다면? 또 메신저 API의 요구사항에 맞춰 고수준 컴포넌트가 수정되어야 할 것이다.


## 개선
어떻게하면 이 문제를 해결할 수 있을까? 어떻게하면 두 컴포넌트 사이의 결합도를 낮추고 유지 보수성을 높일 수 있을까?  
우리는 `의존 역전 원칙(DIP)`를 통해 해결할 수 있다.


```java
// Service
@Service
public class BoardService {
    private BoardDao boardDao;
    private Messenger messenger;

    public void write(BoardVo board) throws Exception {
        if (board.validation()) {
            throw ...
        }
        int cnt = boardDao.write(board);
        if (cnt == 0) {
            throw ...
        }

        messenger.send(board.title, board.userId);
    }
}

public interface Messenger {
    public void send(String title, String userId);
}

// Mattermost
@Componenet
public class Mattermost implements Messenger {
    @Override
    public void send(String title, String userId) {
        ...
    }

    private formatJson(String title, String userId) {
        return ...
    }
}

// Slack
@Componenet
public class Slack implements Messenger {
    @Override
    public void send(String title, String userId) {
        formatXml(title, userId)
        ...
    }

    private formatXml(String title, String userId) {
        return ...
    }
}
```

인터페이스를 통해 고수준 컴포넌트는 더 이상 저수준 컴포넌트의 세부사항을 알 필요가 없어졌다. 의존 관계가 역전한 것이다.  
만약 카카오톡으로 전송하기 위해 메신저를 추가한다고 가정해보자. 고수준 컴포넌트를 몰라도 되며, `Messenger`를 구현하고 있는 `Kakao` 클래스만 추가하면 된다.  
이것은 개방 폐쇄 원칙(OCP)를 지키는 것이다.