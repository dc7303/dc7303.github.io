---
layout: post
title: "맥 카탈리나 업데이트 후 크론 작동 안되는 현상"
date: 2019-12-11 02:00:00
category: 
- etc
tag: 
- cron
comments: true
---

맥OS 카탈리나 업데이트 이후 Crontab에 등록된 예약 업무들이 작동하지 않을 때 솔루션 공유.

### 원인
카탈리나로 업데이트 되면서 맥OS 보안 정책이 강화되어 `cron`에 접근 권한을 줘야한다.

### 해결

`System Preferences > Security & Privacy > Privacy` 로 이동한다. 그리고 `Full Disk Access` 항목에서 cron을 추가해준다.

![fullDiskAccess](/assets/images/post/macCatalinaCronIssue-fullDiskAccess.png){: width="100%"}*\<크론 접근 권한주기\>*

cron은 `/usr/sbin/cron` 경로에 있다. 숨김 폴더가 안보일 때 `command + shift + >` 단축키를 사용하면 보인다.

![findCron](/assets/images/post/macCatalinaCronIssue-findCron.png){: width="100%"}*\<크론 찾기\>*
