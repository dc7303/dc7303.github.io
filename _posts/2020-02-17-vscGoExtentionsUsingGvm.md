---
layout: post
title: "VSC GO Extention, gvm 사용할 때 작동 이슈"
date: 2020-02-17 02:00:00
category: 
- go
- vsc
tag: 
- go
- vsc
comments: true
---

고를 사용할때 많이 사용하는 에디터로 VSCode가 있다. 그리고 관련 Extention을 설치해서 사용한다.  
여기서 gvm으로 고를 설치한 유저는 추가 설정이 필요하다. 시간을 아끼기 위해 기록해둔다.

### 이슈
VSCode를 설치하고 Go Extention을 설치한 후 VSCode를 리로드한다.

만약 gvm을 사용하고 있다면 아래와 같은 메세지를 볼 것이다. (보통 우측 하단에 있다)

```bash
Failed to run "go env" to find GOPATH as the "go" binary cannot be found in either GOROOT(undefined) or PATH(......)
```

`go env`를 실행해서 GOPATH를 찾지 못했다고 한다. Extention이 실행될 때 `go env`를 실행하는 걸로 확인된다.

그런데 gvm을 사용할때 고는 다른점이 있다. gvm은 쉘이 실행될때 gvm 스크립트를 사용해서 활성화해주고, 활성화되면 고 바이너리들을 `$PATH`에 셋팅한다. 그리고 이 작업을 쉘 스크립트가 실행될때 수행하도록 설정파일에 추가해서 사용한다. ([참고](https://dc7303.github.io/go/2019/11/29/goGvmInstallVersion/))

여기서 추측할 수 있듯이 Extention이 실행되는 시점과 gvm이 실행되는 시점이 맞지않아 발생한 이슈인 것으로 추측된다.

### 해결
VSCode user settings에서 GOPATH와 GOPATH를 설정할 수 있다.

JSON Settings 파일에 아래의 설정을 추가한다.

```json
{
  ...
  
  "go.gopath": "/Users/choedongcheol/.gvm/pkgsets/go1.13.8/global",
  "go.goroot": "/Users/choedongcheol/.gvm/gos/go1.13.8",

  ...
}
```

저 경로 그대로를 복사하면 안된다. 저건 내 경로다. 내꺼다.

`go env` 명령어를 통해 GOPATH와 GOROOT를 입력하도록 한다.

그리고 페이지를 리로드하면 정상적으로 작동하는걸 확인할 수 있다.