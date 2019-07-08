---
published: true
layout: single
title: "슬랙 API 사용 후기"
category: Project
comments: true
---

# 슬랙 API 종류

1. `Incoming Webhooks`

   : 슬랙에 메시지 게시하기. 어떤 시스템이 메시지를 게시할 수 있게 한다.

2. `Slash Commands`

   : 슬랙에 슬래시 커맨드를 추가하기. 

3. `Bots`

   : 유저와 비슷한 App. 채널에 초대하거나 DM을 보낼 수 있다. 

4. `Interactive Components`

   : 상호작용 컴포넌트. 버튼이 대표적이다. 버튼을 누르면 프로필이 바뀌거나, 초대하거나, 특정 스레드를 열거나....

5. `Event Subscription`

   : 슬랙 Event를 감지하기. 등록된 Event가 발생하면 등록한 URL로 메시지를 보내준다.

6. `Permissions`

   : 만든 앱의 권한을 부여하고 조정한다.



## Incoming Webhooks

> **hooking** covers a range of techniques used to alter or augment the behaviour of an operating system, of applications, or of other software components by intercepting function calls or messages or events passed between software components. - _WIKIPedia_

`Hook`은 어떤 기능에 고리를 걸어(`Hooking`) 행동을 변경하거나 부가 효과를 얻는 방법을 말한다.

`Webhook`은 웹버전 훅이다. 웹페이지나 웹앱에 커스텀 콜백을 넘겨 동작하게 하는 것이다.



