---
layout: post
title: "[JS] MacOS에서 nvm으로 노드 버전 관리하기"
date: 2019-11-17 01:00:00
category:
- javascript
- nodejs
tag:
- javascript
- nodejs
comments: true
---


자바스크립트로 프로그래밍하다 보면 노드JS 버전을 여러 버전으로 관리할 필요를 느낀다.  
그리고 n과 nvm 무엇을 사용할지 고민하다 nvm이 관리하기가 더 편해서 nvm을 사용하고 있다.  
nvm 설치 방법과 기본 사용법을 기록한다.

### nvm 설치
#### nvm 문서
[https://github.com/nvm-sh/nvm](https://github.com/nvm-sh/nvm)

#### 설치 명령어
nvm을 업데이트 또는 설치하기 위해서는 설치 스크립트를 실행해야한다. 명령어를 실행하면 nvm이 설치된다.

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.1/install.sh | bash
```

#### 셋팅
설치가 완료되면 `~/.nvm` 디렉토리를 생성한다.

그리고 `.nvm` 디렉토리에 경로를 `NVM_DIR` 환경 변수를 설정과 터미널이 실행 될때 `nvm.sh`이 실행되도록 설정해주어야 한다.  
매번할 수 없으니 이는 profile에 추가해주도록 하자. (`~/bash_profile`, `~/.zshrc`, `~/.bashrc` 등)

```vim
# NVM
export NVM_DIR=~/.nvm
source $(brew --prefix nvm)/nvm.sh
```

### 노드JS 관리
다양한 옵션은 공식 문서와 `nvm --help` 명령어로 확인할 수 있다.

#### 설치
원하는 버전을 설치할 수 있다.
```bash
nvm install <version>
```

최신 버전을 설치하고 싶다면 아래 방법이 심플하다.
```bash
nvm install node
```

#### 삭제
```bash
nvm uninstall <version>
```

#### 설치된 파일은?
`which` 명령어를 활용해 보자.
```bash
which node

// /Users/margurt/.nvm/versions/node/v12.12.0/bin/node
```

버전별 설치 경로도 확인할 수 있다.
```bash
nvm which <version>
```

위와 같이 생성한 `.nvm` 디렉토리의 versions에서 관리하는 걸 확인할 수 있다.


##### 현재 사용중인 버전 확인
```bash
nvm current
// v12.12.0
nvm version
// v12.12.0
```

##### 설치된 버전 확인
```bash
nvm ls       
```

##### 사용할 버전 변경
```bash
nvm use <version>
```



