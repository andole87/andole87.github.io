---
published: true
layout: single
title: "디버깅과 콜스택"
category: ETC
comments: true
---

> 의도치 않은 결과(버그)를 찾아내고, 문제를 해결하는 과정.   
> 개발자는 대부분의 시간을 디버깅에 쏟는다. == 디버깅을 잘하는게 중요한 능력이다.  
> 프로젝트 말미, 새벽 1시에 디버깅하는 그 기분...ㅗㅜㅑ  

## 그럼 디버깅은 구체적으로 무얼 말하는거야?
개발자는 코드를 쓰고 실행시킨다. 간단한 소스는 직관적으로 알 수 있지만, 여러 모듈과 관계되어 있거나, 처리 흐름이 복잡하다면 개발자는 외울수가 없다.  
어떤 계획을 가지고 코드를 작성했으나 의도하지 않는 결과가 나올 때 실행을 추적하면서 문제를 찾아나갈 수 있다. 문제점을 정확히 알게 되면 문제 해결이 시작된다.  
프로그램이 실행되는 과정을 프로그래머가 찬찬히 뜯어보면서, 어떤 메서드가 콜을 받는지, 처리 시간은 얼마나 걸리는지, 변수가 어떻게 변화하는지 확인하는 과정을 거친다.  
때문에 디버깅을 하면서 많은 공부가 된다. 진짜다.  

## 시나리오: 허접 개발자 andole의 하루

andole은 간단한 웹 앱을 만들었다. ajax로 값을 가져오는 기능을 구현했는데, 직접 실행해 보니 분명 존재하는 값인데 아무것도 보이지 않았다. andole은 디버깅을 시작하기로 마음먹었다.

andole은 login.js에서 의심되는 부분에 breakpoint(중단점)를 설정했다.  
IDE의 디버깅 기능을 실행했다.  
코드가 순차적으로 실행되다가, 중단점에 이르러 실행이 멈추었다.  
디버깅 패널에는 현재 context(맥락)에 맞는 변수들이 표시되었다.  
Step over(한 단계씩 실행)으로 다음 코드를 실행했다. 디버깅 패널의 변수들이 바뀌었다.  
반복문을 호출하는 코드로 옮겨가자, 디버깅 커서가 다른곳으로 점프했다.  
for문이 js에서 어떻게 구현되었는지는 관심 없었으므로, Step out(빠져나오기)를 실행했다.  
콜스택에서 빠져나와 이전의 login.js 다음 라인에 디버깅 커서가 옮겨갔다.  
ajax모듈의 send()호출부분에서, 더 자세히 보고 싶어졌다. Step in(진입)을 실행했다.  
ajax모듈의 생성부터 send()까지 순서대로 디버깅커서가 옮겨가며 실행되었다.  
andole은 끝까지 실행하고도 뭐가 잘못되었는지 찾지 못했다. (andole은 멍청하다)  

andole은 xhttp 상태를 처음부터 추적하기로 했다.  
다시한번 디버깅을 실행하고, watch에 ajax 네트워크 관련 변수를 집어넣었다.  
Step over, Step out, Step in에 따라 ajax의 변화가 watch 패널에 표시되었다.  
andole은 아직도 원인을 찾지 못했다.  
인간 디버깅이 필요하다는 것을 깨달았다...  

## breakingpoint
중단점. 디버깅 실행중에 코드 실행을 멈출 포인트다. 디버거는 중단점 이전까지 정상적으로 프로그램을 실행하고 중단점에서 실행을 멈춘다.

## watch 사용법
관심있는 변수나 상태를 watch에 등록하면, 프로그램 실행 중에 등록한 변수의 변화를 모니터링 할 수 있다.

## call stack
함수 호출 스택이다. 어떤 함수를 호출하면, 그 함수는 콜스택에 저장된다. 프로그램은 마지막에 추가한 콜스택부터 실행하고 그 반환값을 다음 콜스택에 연결, 스택이 비워질 때까지 실행한다.
![콜스택 이미지](https://cdn-images-1.medium.com/max/1600/1*1FL2WcODqRrK40rrzA5QQA.png)

## Step over /Step into / Step out
스텝오버는 다음 코드 실행이다. 디버깅 커서가 다음 라인으로 옮겨간다. 현재 맥락에서 다음 코드를 실행한다.
스텝인투는 콜스택 내부로 진입하는 것. 메서드 선언부로 점프하면서 메서드 내부로 진입한다.
스텝아웃은 콜스택을 빠져나오는 것. 하위 메서드를 빠져나오며 호출자의 다음 라인으로 커서가 이동한다.
