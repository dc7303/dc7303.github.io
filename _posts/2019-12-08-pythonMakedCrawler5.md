---
layout: post
title: "[Python] 크롤링으로 업무 자동화 - (5)슬랙으로 결과 전송"
date: 2019-12-08 02:00:00
category: 
- python
tag: 
- python
- slack
comments: true
---

파일 비교 결과를 슬랙으로 전송하자.

### 들어가기에 앞서
- 예제 코드는 [pycrawler-exam](https://github.com/dc7303/pycrawler-exam)을 통해 다운로드 받을 수 있다.
- [[Python] 크롤링으로 업무 자동화 - (4)파일 비교](https://dc7303.github.io/python/2019/12/07/pythonMakedCrawler4/)를 안읽었다면 먼저 읽기를 권한다.
- 예제는 크롬 버전 78.0.3904.108, 웹 드라이버 버전 [78.0.3904.70](https://chromedriver.storage.googleapis.com/index.html?path=78.0.3904.70/)으로 만들어졌다.

### 슬랙 준비
슬랙(Slack)은 협업을 위한 메신저 서비스다. 그리고 메신저 외 많은 기능을 가지고 있다. 많은 기능 중 채널 관리, 마크다운 지원, Github와 연동, 봇 기능들 때문에 현재 팀 내에서 사용하고 있다.

이번 포스팅에서는 이전 포스팅에서 얻은 파일 비교 결과를 슬랙으로 전송할 것이다.

#### 워크스페이스 만들기
먼저 워크스페이스가 없다면 [슬랙 공식 홈페이지](https://slack.com/intl/en-kr/)에서 메일로 워크스페이스를 만들어야 한다.

홈페이지 접속 후 `GET STARTED` 버튼을 클릭한다.

![slackJoin1](/assets/images/post/pythonMakedCrawler5-slackJoin1.png){: width="100%"}*\<슬랙 워크스페이스 만들기 (1)\>*

다음 두 메뉴에서 `My team isn't using Slack yet` 메뉴를 선택한다.

![slackJoin2](/assets/images/post/pythonMakedCrawler5-slackJoin2.png){: width="100%"}*\<슬랙 워크스페이스 만들기 (2)\>*

사용할 메일을 입력 후 `Confirm` 버튼을 클릭한다.

![slackJoin3](/assets/images/post/pythonMakedCrawler5-slackJoin3.png){: width="100%"}*\<슬랙 워크스페이스 만들기 (3)\>*

전송받은 인증번호를 입력한다. 주의할 점은 스팸으로 분류될 수 있다. 메일이 안 온다면 스팸 보관함을 확인한다.

![slackJoin4](/assets/images/post/pythonMakedCrawler5-slackJoin4.png){: width="100%"}*\<슬랙 워크스페이스 만들기 (4)\>*

사용할 팀 이름을 입력 후 `Next` 버튼을 클릭한다.

![slackJoin5](/assets/images/post/pythonMakedCrawler5-slackJoin5.png){: width="100%"}*\<슬랙 워크스페이스 만들기 (5)\>*

프로젝트 이름을 입력 후 `Next` 버튼을 클릭한다.

![slackJoin6](/assets/images/post/pythonMakedCrawler5-slackJoin6.png){: width="100%"}*\<슬랙 워크스페이스 만들기 (6)\>*

팀원을 추가한다. 봇을 개발하기 위한 테스트 워크스페이스를 만드는 것이기 때문에 스킵한다.

![slackJoin7](/assets/images/post/pythonMakedCrawler5-slackJoin7.png){: width="100%"}*\<슬랙 워크스페이스 만들기 (7)\>*

모든 입력을 끝내면 생성한 워크스페이스를 볼 수 있다.

![slackJoin8](/assets/images/post/pythonMakedCrawler5-slackJoin8.png){: width="100%"}*\<슬랙 워크스페이스 만들기 (8)\>*


#### 슬랙 API 토큰 발급
메세지를 보내기 위해서 워크스페이스에 접근하기 위한 토큰이 필요하다.

워크스페이스에 로그인이 된 상태로 [Slack API](https://api.slack.com/)에 접속한다. 그리고 `Start Building` 버튼을 클릭한다.

![slackApi1](/assets/images/post/pythonMakedCrawler5-slackApi1.png){: width="100%"}*\<슬랙 API (1)\>*

버튼을 클릭하면 슬랙 앱을 만드는 입력창이 나온다. 앱 이름과 적용할 워크스페이스를 선택한다.

![slackApi2](/assets/images/post/pythonMakedCrawler5-slackApi2.png){: width="100%"}*\<슬랙 API (2)\>*

앱이 생성되면 `Bot Users` 메뉴에서 봇을 생성한다.

![slackApi3](/assets/images/post/pythonMakedCrawler5-slackApi3.png){: width="100%"}*\<슬랙 API (3)\>*

입력 값을 적어주고 `Add Bot User` 버튼을 클릭하여 생성한다.

![slackApi4](/assets/images/post/pythonMakedCrawler5-slackApi4.png){: width="100%"}*\<슬랙 API (4)\>*

봇 생성이 되었으면 `OAuth & Permissions` 메뉴에서 `Install App to Workspace` 버튼을 클릭한다. 워크스페이스에 현재 생성된 앱을 설치하는 것이다.

![slackApi5](/assets/images/post/pythonMakedCrawler5-slackApi5.png){: width="100%"}*\<슬랙 API (5)\>*

`Allow` 버튼을 클릭한다.

![slackApi6](/assets/images/post/pythonMakedCrawler5-slackApi6.png){: width="100%"}*\<슬랙 API (6)\>*

최종적으로 아래와 같이 토큰을 발급받을 것이다. 사용할 토큰값은 `Bot User OAuth Access Token`이다.

![slackApi7](/assets/images/post/pythonMakedCrawler5-slackApi7.png){: width="100%"}*\<슬랙 API (7)\>*

#### Slacker
[슬랙커](https://github.com/os/slacker)는 서드파티 라이브러리로 파이썬 코드에서 슬랙으로 메세지를 전송할 때 필요하다.

아래의 명령어로 설치한다.

```bash
pip install slacker
```

`slackhandler.py` 파일을 생성하고 슬랙커 인스턴스를 다루기 위한 클래스를 추가한다.

```python
# slackhandler.py

from slacker import Slacker


class Slack(object):
    """
    슬랙 API 핸들링을 위한 객체

    :param token: 슬랙 API 토큰
    :param channel: 채널 이름
    :param username: 전송자 이름
    """
    def __init__(self, token: str, channel: str, username: str):
        self.slack = Slacker(token)
        self.channel = channel
        self.username = username

    def send_slack_msg(self, text: str = '변경된 내용이 없습니다.'):
        """
        슬랙 메세지 전송

        :param text: 변경 내용
        """
        self.slack.chat.post_message(
            channel=self.channel,
            username=self.username,
            text=text)
```

슬랙 API에서 발급받은 토큰으로 Slacker 인스턴스를 생성한다. 그리고 `slacker.chat.post_message()` 함수에 채널명, 보낸이 이름, 메세지 내용 등 옵션을 포함하여 전송한다.

클래스를 추가했으니 `main.py`에 객체를 생성한다.

```python
# main.py

...

from slackhandler import Slack

...

# 슬랙 생성
SLACK_TOKEN = '슬랙 토큰'
SLACK_CHANNEL = '채널 이름'
SLACK_SENDER_NAME = '보낸이 이름'
slack = Slack(token=SLACK_TOKEN, channel=SLACK_CHANNEL, username=SLACK_SENDER_NAME)
```

이제 메세지를 생성해서 전송만 하면 된다.


### 메세지 생성
슬랙은 마크다운을 지원한다. 일반적인 마크다운과 차이가 있으니 [Slack Help Center](https://slack.com/intl/en-kr/help/articles/202288908-format-your-messages)를 참고하여 원하는 포맷으로 메세지를 꾸며보면 좋다.

#### 삭제 파일, 생성된 파일
`slackhandler.py`에 삭제 파일과 생성된 파일 리스트를 메세지로 변환해주는 함수를 추가한다.

```python
# slackhandler.py

...

def gen_total_file_update_info_text(deleted_file_list: list, new_file_list: list):
    """
    전체파일 업데이트 정보 텍스트 생성.
    추가/삭제 된 내용을 텍스트로 가져온다.

    :param deleted_file_list: 삭제된 파일리스트
    :param new_file_list: 추가된 파일리스트
    """
    text = '*전체 서비스 명세서 추가/삭제 업데이트 리스트*: \n\n'

    if len(deleted_file_list) != 0:
        for f in deleted_file_list:
            text += f'>삭제 - {f}'
        text += '\n'

    if len(new_file_list) != 0:
        for f in new_file_list:
            text += f'>추가 - {f}'
        text += '\n'

    if len(deleted_file_list) == 0 and len(new_file_list) == 0:
        text += '>추가/삭제 된 파일이 없습니다.'

    return text
```

이전에 분리한 삭제 파일 리스트와 생성 파일 리스트를 받아서 변환한다. 

메세지를 추가할 때 `>`를 앞에 추가한다. `>`는 슬랙 마크다운 문법중 하나로 라인을 들여쓰기한다.

#### 파일 ROW 변경 정보
`slackhandler.py`에 파일 ROW 변경 정보 리스트를 메세지로 변환해주는 함수를 추가한다.

```python
# slackhandler.py

...

def gen_diff_row_info_text(file_diff_info_list: list):
    """
    파일별 ROW에 대한 변경 정보를 텍스트로 생성한다.

    :param file_diff_info_list: 파일 ROW 다른 정보 리스트
    """
    text = '*명세서별 변경된 ROW 정보*: \n\n'

    if len(file_diff_info_list) != 0:
        for f in file_diff_info_list:
            text += f'`{f.file_name}`:\n{f.get_diff_row_format_str()}\n'
    else:
        text += '>변경된 정보가 없습니다.'

    return text
```

이전에 ROW 데이터를 비교하면서 생성한 `FileDiifInfo` 객체 리스트를 인자로 받는다. 그리고 파일 이름과 `get_diff_row_format_str()` 함수를 사용하여 메세지를 만든다.

여기서도 마크다운 문법을 사용하여 메세지를 꾸며준다.


### 메세지 전송
이제 준비는 끝났다 마지막으로 메세지 전송을 한다. `main.py`에서 생성된 메세지들을 조합하여 결과 메세지를 전송한다.

```python
# main.py

...

import datetime

...

# 삭제 파일, 생성된 파일 리스트 관련 메세지 생성
total_file_update_info_text = gen_total_file_update_info_text(deleted_file_list, new_file_list)
# 파일별 달라진점 객체 리스트 문자 데이터 생성
file_diff_info_text = gen_diff_row_info_text(file_diff_info_list)

result_msg = f'{datetime.now()}\n크롤러 결과============================================\n\n\n\n'
if total_file_update_info_text is None and file_diff_info_text is None:
    result_msg += '변경된 내용이 없습니다.'
else:
    if total_file_update_info_text is not None:
        result_msg += f'{total_file_update_info_text}\n\n\n\n\n'

    if file_diff_info_text is not None:
        result_msg += f'{file_diff_info_text}\n\n\n'
result_msg += '====================================================='

slack.send_slack_msg(text=result_msg)
```

이렇게 코드를 추가하면 `main.py`와 `slackhandler.py` 최종 코드는 아래와 같다.

```python
# main.py

import sys
import os
import time
import zipfile
import glob
import shutil
from datetime import datetime
from selenium.webdriver.common.keys import Keys

from logger import setup_custom_logger
from chromedriver import generate_chrome
from xlsxhandler import get_dir_update_info, get_file_diff_info_list
from slackhandler import Slack, gen_total_file_update_info_text, gen_diff_row_info_text

logger = setup_custom_logger('main.py')
logger.debug('Run crawler!!!!')
PROJECT_DIR = str(os.path.dirname(os.path.abspath(__file__)))
DOWNLOAD_DIR = f'{PROJECT_DIR}/download'

driver_path = f'{PROJECT_DIR}/lib/webDriver/'
platform = sys.platform
if platform == 'darwin':
    logger.debug('System platform : Darwin')
    driver_path += 'chromedriverMac'
elif platform == 'linux':
    logger.debug('System platform : Linux')
    driver_path += 'chromedriverLinux'
elif platform == 'win32':
    logger.debug('System platform : Window')
    driver_path += 'chromedriverWindow'
else:
    logger.error(f'[{sys.platform}] not supported. Check your system platform.')
    raise Exception()

# 크롬 드라이버 인스턴스 생성    
chrome = generate_chrome(
    driver_path=driver_path,
    headless=True,
    download_path=DOWNLOAD_DIR)

# 페이지 요청
url = 'https://github.com/login'
chrome.get(url)
time.sleep(3)

# 깃허브 로그인
login_page = chrome.page_source

elm = chrome.find_element_by_id('login_field')
elm.send_keys('깃허브 아이디')
elm = chrome.find_element_by_id('password')
elm.send_keys('깃허브 비밀번호')
elm.send_keys(Keys.RETURN)

time.sleep(5)

# 페이지 이동
url = 'https://github.com/dc7303/pycrawler-exam-dummy-data'
chrome.get(url)
time.sleep(5)

# 다운로드 토글 오픈
elm = chrome.find_element_by_xpath('//*[@id="js-repo-pjax-container"]/div[2]/div/div[3]/details[2]/summary')
elm.click()

# 다운로드 버튼
elm = chrome.find_element_by_xpath('//*[@id="js-repo-pjax-container"]/div[2]/div/div[3]/details[2]/div/div/div[1]/div[3]/a[2]')
elm.click()
time.sleep(5)

# zip파일 경로와 압축해제 후 디렉토리 경로 셋팅
repo_name = 'pycrawler-exam-dummy-data-master'
zip_file_path = f'{DOWNLOAD_DIR}/{repo_name}.zip'
xlsx_dir_path = f'{DOWNLOAD_DIR}/{repo_name}'

# 이전에 비교한 디렉토리가 있다면 삭제
if os.path.isdir(xlsx_dir_path):
    shutil.rmtree(xlsx_dir_path)

# ZIP 파일 존재여부 확인 후 압축 풀기
if os.path.isfile(zip_file_path):
    z = zipfile.ZipFile(zip_file_path)
    z.extractall(DOWNLOAD_DIR)
    z.close()
    os.remove(zip_file_path)

# 압축 해제한 파일 디렉토리 경로 선언
before_dir_path = f'{xlsx_dir_path}/before'
after_dir_path = f'{xlsx_dir_path}/after'

# 파일 경로 리스트 조회
before_xlsx_list = glob.glob(f'{before_dir_path}/*.xlsx')
after_xlsx_list = glob.glob(f'{after_dir_path}/*.xlsx')

# 파일 삭제, 추가 정보 비교 분석
deleted_file_list, new_file_list = get_dir_update_info(before_xlsx_list, after_xlsx_list)

# 파일 비교 분석 후 가져오기
file_diff_info_list = get_file_diff_info_list(after_xlsx_list, before_dir_path)

# 슬랙 생성
SLACK_TOKEN = '슬랙 토큰'
SLACK_CHANNEL = '채널 이름'
SLACK_SENDER_NAME = '보낸이 이름'
slack = Slack(token=SLACK_TOKEN, channel=SLACK_CHANNEL, username=SLACK_SENDER_NAME)

# 삭제 파일, 생성된 파일 리스트 관련 메세지 생성
total_file_update_info_text = gen_total_file_update_info_text(deleted_file_list, new_file_list)
# 파일별 달라진점 객체 리스트 문자 데이터 생성
file_diff_info_text = gen_diff_row_info_text(file_diff_info_list)

result_msg = f'{datetime.now()}\n크롤러 결과============================================\n\n\n\n'
if total_file_update_info_text is None and file_diff_info_text is None:
    result_msg += '변경된 내용이 없습니다.'
else:
    if total_file_update_info_text is not None:
        result_msg += f'{total_file_update_info_text}\n\n\n\n\n'

    if file_diff_info_text is not None:
        result_msg += f'{file_diff_info_text}\n\n\n'
result_msg += '====================================================='

slack.send_slack_msg(text=result_msg)
```

```python
# xlsxhandler.py

import pandas as pd
from models import FileDiffInfo
from logger import setup_custom_logger

logger = setup_custom_logger('xlsxhandler')


def _compare_file_list(compare_list: list, compare_target_list) -> list:
    """
    두 파일 리스트를 비교하여 다른 파일 정보가 있다면
    해당 파일 이름을 리스트에 담아 리턴한다.

    :param compare_list: 비교할 리스트
    :param compare_target_list: 비교 대상 리스트
    :return: 다른 항목 리스트
    """
    result_list = []
    for name in compare_list:
        if name not in compare_target_list:
            result_list.append(name)

    return result_list


def get_dir_update_info(before_xlsx_path_list: list, after_xlsx_path_list: list) -> (list, list):
    """
    이전 파일 리스트와 현재 파일리스트를 비교하여
    삭제된 파일과 추가된 파일을 파악하여 반환

    :param before_xlsx_path_list: 이전 파일경로 리스트
    :param after_xlsx_path_list: 업데이트 후 파일경로 리스트
    :return: 삭제 파일 리스트와 생성된 파일 리스트
    """
    after_file_name_list = [after.split('/')[-1] for after in after_xlsx_path_list]
    before_file_name_list = [before.split('/')[-1] for before in before_xlsx_path_list]
    deleted_file_list = _compare_file_list(before_file_name_list, after_file_name_list)
    new_file_list = _compare_file_list(after_file_name_list, before_file_name_list)

    return deleted_file_list, new_file_list


def get_file_diff_info_list(after_xlsx_path_list: list, before_dir_path: str) -> list:
    """
    파일 시트 다른 부분 정보 리스트 가져오기

    :param after_xlsx_path_list: 최신 서비스명세서 엑셀 파일경로 리스트
    :param before_dir_path: 읽을 파일 디렉토리 경로

    :return list: 파일 변경된 정보 객체 리스트
    """
    diff_info_list = []
    for xlsx_path in after_xlsx_path_list:
        # 엑셀파일 읽기
        try:
            after_df = pd.read_excel(xlsx_path)
            file_name = xlsx_path.split('/')[-1]

            # 이전 버전 조회
            before_df = pd.read_excel(f'{before_dir_path}/{file_name}')
        except FileNotFoundError:
            continue

        # 시트 데이터가 같은지 비교 후 같지 않다면 상세 비교
        if not before_df.equals(after_df):
            # 두 데이터 다른 부분 추출
            df = pd.concat([before_df, after_df])
            duplicates_df = df.drop_duplicates(keep=False)

            before_list = [str(r) for r in before_df.values.tolist()]
            after_list = [str(r) for r in after_df.values.tolist()]

            # 변경 전 데이터, 변경 후 데이터 분류
            changed_list = []
            duplicates_list = [str(r) for r in duplicates_df.values.tolist()]
            for row in duplicates_list:
                try:
                    before_list.index(row)
                    changed_list.append(f'~{row}~')
                except ValueError:
                    pass

                try:
                    after_list.index(row)
                    changed_list.append(row)
                except ValueError:
                    pass

            # 변경된 정보를 핸들링할 객체 생성
            info = FileDiffInfo(file_name, changed_list)
            diff_info_list.append(info)

    return diff_info_list
```

### 결과
`main.py`를 실행한다. 그러면 슬랙 워크스페이스에 아래와 같은 결과가 나올 것이다.

![resultMsg](/assets/images/post/pythonMakedCrawler5-resultMsg.png){: width="100%"}*\<결과 메세지\>*

성공적이다. [다음 포스팅](https://dc7303.github.io/python/2019/12/08/pythonMakedCrawler6/)에서 이 크롤러를 `cron`을 사용하여 알아서 작동하도록 해보자.