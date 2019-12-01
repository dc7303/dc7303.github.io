---
layout: post
title: "[Python] 크롤링으로 업무 자동화하기 - (2)크롬 드라이버 인스턴스 생성"
date: 2019-12-01 02:00:00
category: python
tag: 
- python
comments: true
---


셀레니움과 크롬 드라이버를 사용하여 webdriver 인스턴스를 생성하여 크롤링할 준비를 한다.


### 들어가기에 앞서
- 예제는 크롬 버전 78.0.3904.108, 웹 드라이버 버전 [78.0.3904.70](https://chromedriver.storage.googleapis.com/index.html?path=78.0.3904.70/)으로 만들어졌다.
- 인스턴스 생성 코드는 [chromedriver-boilerplate](https://github.com/dc7303/chromedriver-boilerplate.git) 레파지토리에 보일러 플레이트 코드를 만들어 두었다. 다운로드하여 참고할 수 있다.

### 셀레니움이란?
셀레니움은 웹 테스트 자동화를 위해 만들어졌다. 크롬, 파이어폭스, 사파리, 엣지 등은 웹 드라이버를 제공하는데, 이를 통해 웹 UI, 기능 테스트 등에 주로 사용된다.

그리고 크롤링 업무에도 많이 사용된다. 비동기적으로 콘텐츠를 제공하는 웹 사이트를 크롤링하기 적합하기 때문이다.

### 크롬 드라이버 다운로드
셀레니움으로 사용할 수 있는 브라우저는 많다. 크롬, 파이어폭스, 사파리, 엣지 등이 있다. 필자는 평소 메인 브라우저로 사용하는 크롬을 사용했다.

먼저 크롬 드라이버를 다운로드한다. [https://chromedriver.chromium.org/downloads](https://chromedriver.chromium.org/downloads)에 접속하면 드라이버를 다운로드할 수 있다.

주의할 점은 무조건 최신 버전을 설치하면 안 된다. 현재 사용중인 크롬과 버전 호환이 되어야 문제없이 작동하기 때문이다.

크롬 버전 확인은 브라우저에서 확인할 수 있다. 우측 상단 더 보기 아이콘에서 도움말 > Chrome 정보를 클릭한다.

![checkChromeVersion1](/assets/images/post/checkChromeVersion1.png){: width="100%"}*\<크롬 브라우저 버전 확인\>*

그러면 이미지와 같은 페이지가 나오는데, 빨간 박스로 표시해둔 영역에 크롬 버전이 표시되어 있다.

![checkChromeVersion2](/assets/images/post/checkChromeVersion2.png){: width="100%"}*\<크롬 브라우저 버전 확인\>*

버전을 확인하고 운영체제에 맞는 드라이버 파일을 설치하여 프로젝트에 추가한다. 예제는 윈도우, 리눅스, 맥 버전 모두를 다운 받았으며, ***프로젝트/lib/webDriver*** 경로에 추가하였다. 운영체제에 따라 유동적으로 드라이버를 선택할 것이기 때문이다.

![driverDownloaded](/assets/images/post/driverDownloaded.png){: width="100%"}*\<설치된 크롬 드라이버들\>*


### 셀레니움 설치 및 import
먼저 셀레니움을 설치한다.

```bash
$ pip install selenum
```

그리고 `chromedirver.py` 파일을 생성하고, 파일에 셀레니움의 webdrvier를 import한다.

```python
# chromedriver.py

from selenium import webdriver
```

### 인스턴스 생성 함수 정의
#### 필요한 파라미터
인스턴스 생성 함수 `generate_chrome`을 선언한다.

```python
# chromedriver.py

from selenium import webdriver

def generate_chrome(
    driver_path: str,
    download_path: str,
    headless: bool=False
    ) -> webdriver:
    """
    크롭 웹드라이버 인스턴스 생성
    
    :param driver_path: 드라이버 경로
    :param download_path: 파일 다운로드 경로
    :param headless: headless 옵션 설정 플래그
    """
```

이때 필요한 인자 값 드라이버 경로, 파일 다운로드 경로, headless 옵션 플래그가 있다.

`dirver_path`는 크롬 드라이버 경로이고 인스턴스 생성시 드라이버 위치를 찾는데 필요하다.

`download_path`는 우리가 다운로드할 xlsx 파일의 설치 경로이다.

`headless` 플래그는 healess 옵션 사용 여부를 정하는데, 이 옵션은 크롬 옵션 설정하는 부분에서 설명하겠다.

#### 옵션 셋팅
선언했다면 드라이버 생성에 필요한 옵션값을 셋팅해주고 크롬 드라이버 인스턴스를 생성한 후 리턴해주는 코드를 작성한다.

```python
# chromedriver.py

from selenium import webdriver

def generate_chrome(
    driver_path: str,
    download_path: str,
    headless: bool=False
    ) -> webdriver:
    """
    크롭 웹드라이버 인스턴스 생성
    
    :param driver_path: 드라이버 경로
    :param download_path: 파일 다운로드 경로
    :param headless: headless 옵션 설정 플래그

    :return webdriver: 크롬 드라이버 인스턴스
    """

    options = webdriver.ChromeOptions()
    if headless:
        options.add_argument('headless')
        options.add_argument('--disable-gpu')
    options.add_experimental_option('prefs', {
        'download.default_directory': download_path,
        'download.prompt_for_download': False,
    })
    
    chrome = webdriver.Chrome(executable_path=driver_path, options=options)

    return chrome
```

`ChromeOptions`를 통해 옵션값을 셋팅할 수 있다. 예제에서는 headless 옵션과 파일 다운로드 옵션을 설정했다.

`headless`를 셋팅할 경우 브라우저 창이 띄워지지 않고 프로세스를 수행한다. 리눅스 터미널 환경에서 사용할 때와 백그라운드 형태로 사용하기 원할 때 유용하다.

다운로드 옵션은 환경 설정 옵션을 통해 셋팅할 수 있다. `prefs`로 전달할 수 있는데, 값은 딕셔너리 타입으로 전달할 수 있다. 다운로드 옵션에 필요한 기본 설정은 아래와 같다.

- `download.default_directory`는 다운로드할 경로를 설정해준다.
- `download.prompt_for_download`를 False로 지정해줘야 propmt 창이 뜨지 않는다.

#### headless 파일 다운로드 이슈
이렇게 옵션을 주었을 때 한가지 이슈가 있다. headless 모드일 때는 파일이 다운로드되지 않는다. 크롬 드라이버는 악의적인 행동을 예방하기 위해 소프트웨어가 컴퓨터에서 파일을 다운로드 하지 못하도록 한다.

이를 해결하기 위해 크롬 커맨드라인에 다운로드를 허용하는 명령을 추가해야 한다.([stackoverflow 참조](https://stackoverflow.com/questions/45631715/downloading-with-chrome-headless-and-selenium)) 이 업무를 수행해줄 함수를 추가하자.

```python
# chromedriver.py

def _enable_download_in_headless_chrome(driver: webdriver, download_dir: str):
        """
        :param driver: 크롬 드라이버 인스턴스
        :param download_dir: 파일 다운로드 경로
        """
        driver.command_executor._commands["send_command"] = ("POST", '/session/$sessionId/chromium/send_command')

        params = {
            'cmd': 'Page.setDownloadBehavior',
            'params': {
                'behavior': 'allow',
                'downloadPath': download_dir
            }
        }
        driver.execute("send_command", params)
```

#### 크롬 종료 함수
모든 프로세스가 끝날 때 자동으로 크롬을 종료해줄 수 있는 함수를 선언해주자. 이 함수는 `atexit`를 통해 프로세스 종료 직전에 실행될 것이다.

```python
# chromedriver.py

def _close_chrome(chrome: webdriver):
    """
    크롬 종료

    :param chrome: 크롬 드라이버 인스턴스
    """
    def close():
        chrome.close()
    return close
```

#### chromedriver 모듈 완성
추가로 선언한 함수들을 인스턴스 생성 함수에서 사용해주면 최종적으로 `chromedriver.py`는 아래와 같다.

```python
# chromedriver.py

import atexit
from selenium import webdriver


def _enable_download_in_headless_chrome(driver: webdriver, download_dir: str):
        """
        :param driver: 크롬 드라이버 인스턴스
        :param download_dir: 파일 다운로드 경로
        """
        driver.command_executor._commands["send_command"] = ("POST", '/session/$sessionId/chromium/send_command')

        params = {
            'cmd': 'Page.setDownloadBehavior',
            'params': {
                'behavior': 'allow',
                'downloadPath': download_dir
            }
        }
        driver.execute("send_command", params)


def _close_chrome(chrome: webdriver):
    """
    크롬 종료

    :param chrome: 크롬 드라이버 인스턴스
    """
    def close():
        chrome.close()
    return close


def generate_chrome(
    driver_path: str,
    download_path: str,
    headless: bool=False
    ) -> webdriver:
    """
    크롭 웹드라이버 인스턴스 생성
    
    :param driver_path: 드라이버 경로
    :param download_path: 파일 다운로드 경로
    :param headless: headless 옵션 설정 플래그

    :return webdriver: 크롬 드라이버 인스턴스
    """

    options = webdriver.ChromeOptions()
    if headless:
        options.add_argument('headless')
        options.add_argument('--disable-gpu')
    options.add_experimental_option('prefs', {
        'download.default_directory': download_path,
        'download.prompt_for_download': False,
    })
    
    chrome = webdriver.Chrome(executable_path=driver_path, options=options)

    if headless:
        _enable_download_in_headless_chrome(chrome, download_path)

    atexit.register(_close_chrome(chrome))       # 스크립트 종료전 무조건 크롬 종료

    return chrome
```

### 인스턴스 생성
생성 준비는 끝났다. 이제 `main.py` 파일을 생성해서 작성한 모듈을 사용해보자.

먼저 프로젝트 경로, 다운로드 경로, 운영체제 판별을 위한 모듈을 import 한다.

```python
# main.py

import sys
import os
import time

from chromedriver import generate_chrome
```

그리고 프로젝트 경로, 다운로드 경로, 드라이버 경로를 선언해준다.

```python
# main.py

...

PROJECT_DIR = str(os.path.dirname(os.path.abspath(__file__)))
DOWNLOAD_DIR = f'{PROJECT_DIR}/download'
driver_path = f'{PROJECT_DIR}/lib/webDriver/'
```

그리고 `sys` 모듈을 사용하여 운영체제를 판별하여 프로그램이 실행되는 환경에 따른 드라이버를 선택해준다. 그리고 chromedriver 모듈에 작성해놓은 `generate_chrome`을 사용하여 인스턴스를 생성한다. 이때 동작을 직관적으로 확인하기 위해 headless는 사용하지 않는다.


```python
# main.py

...

platform = sys.platform
if platform == 'darwin':
    print('System platform : Darwin')
    driver_path += 'chromedriverMac'
elif platform == 'linux':
    print('System platform : Linux')
    driver_path += 'chromedriverLinux'
elif platform == 'win32':
    print('System platform : Window')
    driver_path += 'chromedriverWindow'
else:
    print(f'[{sys.platform}] not supported. Check your system platform.')
    raise Exception()

# 크롬 드라이버 인스턴스 생성    
chrome = generate_chrome(
    driver_path=driver_path,
    headless=False,
    download_path=DOWNLOAD_DIR)
```

그리고 정상적으로 생성이 되었는지 테스트를 위해 깃허브 로그인 페이지를 띄워서 테스트한다. 요청은 `get()` 메소드를 사용하고, `time.sleep()`을 통해 일정 시간 프로세스를 정지해준다. 정지하지 않을 경우 프로세스 종료와 함께 크롬이 종료될 것이다.

요청 코드를 추가하면 최종적으로 `main.py`는 아래와 같다.

```python
# main.py

import sys
import os
import time

from chromedriver import generate_chrome


PROJECT_DIR = str(os.path.dirname(os.path.abspath(__file__)))
DOWNLOAD_DIR = f'{PROJECT_DIR}/download'
driver_path = f'{PROJECT_DIR}/lib/webDriver/'

platform = sys.platform
if platform == 'darwin':
    print('System platform : Darwin')
    driver_path += 'chromedriverMac'
elif platform == 'linux':
    print('System platform : Linux')
    driver_path += 'chromedriverLinux'
elif platform == 'win32':
    print('System platform : Window')
    driver_path += 'chromedriverWindow'
else:
    print(f'[{sys.platform}] not supported. Check your system platform.')
    raise Exception()

# 크롬 드라이버 인스턴스 생성    
chrome = generate_chrome(
    driver_path=driver_path,
    headless=False,
    download_path=DOWNLOAD_DIR)

# 페이지 요청
url = 'https://github.com/login'
chrome.get(url)
time.sleep(3)
```

코드를 실행해보자.

```bash
$ python main.py
```

![githubLoginPageOpen](/assets/images/post/githubLoginPageOpen.png){: width="100%"}*\<깃허브 로그인 페이지\>*

위 이미지와 같이 깃허브 로그인 페이지가 띄워지고 종료되었다면 성공이다.

다음 포스팅에서 크롤링을 위한 페이지 분석과 로그인, 그리고 다운로드를 다뤄보겠다.