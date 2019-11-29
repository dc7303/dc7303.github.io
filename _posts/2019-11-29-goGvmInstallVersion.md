---
layout: post
title: "[GO] 맥OS에서 gvm 설치 그리고 이슈"
date: 2019-11-29 7:00:00
category: go
tag: 
- go
comments: true
---

고랭 버전관리를 위한 gvm라이브러리를 설치했다. 다른 버전관리 라이브러리와 같이 간단할 줄 알았는데 이슈가 있었다. 같은 이유로 고통받을 누군가를 위해 기록한다.

### 시스템 환경
macOS Catalina 10.15.1

### gvm 설치
엑스코드 커맨드라인 툴과 머큐리얼이 설치가 안되어 있다면 설치한다.
```bash
$ xcode-select --install
$ brew update
$ brew install mercurial
```


설치가 완료되면 [가이드 문서](https://github.com/moovweb/gvm)에 따라 curl을 사용하여 gvm을 설치해준다.

```bash
$ bash < <(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)
```

설치가 완료되면 gvm 스크립트를 실행해야 사용할 수 있다는 메세지가 나온다. 매번 할 수 없으니 아래와 같이 터미널 설정파일에 추가한다.

```bash
# ex) .zshrc, .bashrc, .bash_profile 등 ..
$ [[ -s "$HOME/.gvm/scripts/gvm" ]] && source "$HOME/.gvm/scripts/gvm"
```

그리고 재시작하면 `gvm` 명령어가 작동하는걸 확인할 수 있다.

```bash
$ gvm version
```

### golang 설치
가이드에서는 go 1.5+ 버전부터는 툴 체인에서 C 컴파일러를 제거하고 고랭으로 작성된 것으로 교체했기 때문에 먼저 go1.4 바이너리 버전을 설치하라고 한다.

아래 예제처럼 말이다.
```bash
$ gvm install go1.4 -B
$ gvm use go1.4
$ export GOROOT_BOOTSTRAP=$GOROOT
$ gvm install go1.5
```

하지만 필자는 이렇게 따라했을 때 아래의 에러 메세지를 얻었다.

```bash
Installing go1.5...
 * Compiling...
failed MSpanList_Insert 0x129e000 0xf7a2267b4450 0x0
fatal error: MSpanList_Insert

runtime stack:
runtime.throw(0x7ffa6b)
	/usr/local/go/src/runtime/panic.go:491 +0xad fp=0x7ffeefbfee40 sp=0x7ffeefbfee10
runtime.MSpanList_Insert(0x82af28, 0x129e000)
	/usr/local/go/src/runtime/mheap.c:692 +0x8f fp=0x7ffeefbfee68 sp=0x7ffeefbfee40
MHeap_FreeSpanLocked(0x827b20, 0x129e000, 0x100)
	/usr/local/go/src/runtime/mheap.c:583 +0x163 fp=0x7ffeefbfeea8 sp=0x7ffeefbfee68
MHeap_Grow(0x827b20, 0x8, 0x0)
	/usr/local/go/src/runtime/mheap.c:420 +0x1a8 fp=0x7ffeefbfeee8 sp=0x7ffeefbfeea8
MHeap_AllocSpanLocked(0x827b20, 0x1, 0x0)
	/usr/local/go/src/runtime/mheap.c:298 +0x365 fp=0x7ffeefbfef28 sp=0x7ffeefbfeee8
mheap_alloc(0x827b20, 0x1, 0x7f0000000012, 0x7fff6d834cb6)
	/usr/local/go/src/runtime/mheap.c:190 +0x121 fp=0x7ffeefbfef50 sp=0x7ffeefbfef28
runtime.MHeap_Alloc(0x827b20, 0x1, 0x10000000012, 0x977a9)
	/usr/local/go/src/runtime/mheap.c:240 +0x66 fp=0x7ffeefbfef88 sp=0x7ffeefbfef50
MCentral_Grow(0x82f898, 0x122d000)

......

ERROR: Failed to compile. Check the logs at /Users/choedongcheol/.gvm/logs/go-go1.5-compile.log
ERROR: Failed to use installed version
```

이상해서 `go version` 명령어를 통해 go를 확인했다. 그런데 위 메세지에서 컴파일 관련 메세지를 제와한 동일한 메세지가 출력됐다.

[gvm issue](https://github.com/moovweb/gvm/issues)를 찾아보니 같은 문제로 고통받는 사람들이 있었다.

해결방법은 이렇다. go1.10+ 바이너리 버전을 설치하고 원하는 버전을 설치하는 것이다.

```bash
$ gvm install go1.10 -B
$ gvm install go1.10.8
$ export GOROOT_BOOTSTRAP=$GOROOT
$ gvm use go1.10.8
```

필자는 1.13.4 버전을 원했기 때문에 아래와 같이 설치했다.

```bash
$ gvm install go1.13 -B
$ gvm install go1.13.4
$ export GOROOT_BOOTSTRAP=$GOROOT
$ gvm use go1.13.4
```

이렇게 설치하고 나면 설치가 성공하는걸 확인할 수 있을 것이다.

에러가 발생한 원인은 go1.4 버전은 오래되었고, 현재 맥OS 버전과 맞지 않아 발생한 것으로 예상된다. 다른 사례를 찾아보면 Docker Desktop for Mac에서도 [동일한 증상](https://stackoverflow.com/questions/48950249/docker-fails-with-on-macos-sierra-with-mspanlist-insert-0x8f1000-0x81d2db0339-0)이 발생하는 것을 확인할 수 있었다. 그리고 이 이슈의 원인은 Docker Desktop for Mac은 MacOS X Sierra 10.12+에서만 지원하기 때문에 이하 버전은 호환이 안되어 발생한 것임을 알 수 있다.

### Reference
[gvm issues #264](https://github.com/moovweb/gvm/issues/264)