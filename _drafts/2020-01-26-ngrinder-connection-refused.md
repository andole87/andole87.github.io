---
title: "nGrinder Connetion refused"
category: ETC
---

부하 테스트를 하다보니 이상한 현상이 있었다.  
![그래프](https://user-images.githubusercontent.com/40727649/73134454-0e666200-407a-11ea-8ebd-2a4bb37144d3.png)

부하 중간에 아예 응답이 없는 경우가 주기적으로 있었다. 그에 따라 결과가 에러로 집계된 테스트도 급증했다. 11만번 요청에서 4만 5천번이 에러로 기록되니 무슨 문제인가 싶었다.  
테스트 로그를 확인해 보았다.  

```
java.net.ConnectException: Connection refused
	at HTTPClient.HTTPConnection$EstablishConnection.run(HTTPConnection.java:4082) ~[grinder-httpclient-3.9.1.jar:na]
```

nGrinder 에이전트에서 타겟으로 부하를 생성할 때, Connection refused 예외가 발생하고 있었다.  
처음부터 Connection refused가 발생하면 방화벽이나 Listen Port를 체크하겠지만, 부하 중간에 주기적으로 발생하는 것으로 보아 소켓 커넥션 문제 같았다. 서버가 부하 때문에 TCP 요청을 무시하면 이런 현상이 발생하지 않을까? 싶었다.  

톰캣을 내장 서버로 사용하고 있었으니 관련 설정을 찾아 봤다. [톰캣 공홈 설정 정보](https://tomcat.apache.org/tomcat-8.5-doc/config/http.html)  
여기서 acceptCount, maxConnections을 조절해보기로 했다. acceptCount는 요청 대기 큐 사이즈를 말하고 maxConnections은 이름 그대로 최대 커넥션을 말한다.  

application.yml에서 설정하면 된다.  

{% gist andole98/7f2a0bd5726138b03932dab09334c17e %}

서버를 다시 띄우고 테스트해봐도 같은 증상이 발생했다. 이번엔 테스트 도중 클라이언트와 서버의 netstat을 모니터링 해봤다.  
요청을 보내는 쪽, 즉 nGrinder 클라이언트는 macOs다. `netstat -a -p tcp | grep 192.168.100.2` 로 확인했다.  
소켓이 대량으로 생성되면서 요청을 시도하고 있었다. 특별한 문제는 감지하지 못했다.  

서버는 윈도우 데스크탑이다. 윈도우 명령은 조금 다르다. `netstat -ano | findstr 192.168.100.7` 로 확인했다.  
정상적이라면 꾸준히 소켓이 열리고 닫히면서 일정량의 소켓은 계속 남아있어야 했다. 테스트 중간에 **아무 소켓도 보이지 않는 시기**가 있었다. Connection refused 발생할 때였다.  

왜 커넥션이 열리지도 않는 걸까 궁금해서 검색질을 해봤다. 윈도우의 경우 최대 소켓의 수가 제한되어 있다는 걸 알게 되었다. [Window Server Tuning TCP/IP](https://www.filehold.com/help/technical/Windows-Server-Tuning-to-Prevent-TCPIP-Port-Exhaustion)  

현재 구성에서 최대 커넥션 수는 65535를 넘길 수 없다. 소켓의 Identity는 <소스IP:소스포트> <목적지IP:목적지포트>로 구분된다. 클라이언트와 서버는 각 하나씩이므로 소스IP, 목적지IP는 같다. 서버의 리스닝 포트는 8080하나로 고정되어 있으므로 클라이언트의 소스포트만이 소켓을 구분지을 수 있다. 포트의 표현은 16비트이므로 2^16 = 65536이 최대다.

레지스트리 편집기에서 유저 포트를 65534까지 늘리고 TimeWait도 짧게 가져갔다.  

윈도우키 + r 이후에 `regedit`을 실행, 레지스트리 편집기로 진입한다.  
**HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters** 경로로 이동하고 키를 생성하거나 수하하면 된다.  

키를 만들 때에는 [오른쪽 클릭 -> 키 -> 키 타입 설정 -> 이름 설정] 하면 된다.

- 유저 포트  
키 이름 : MaxUserPort  
키 타입 : DWORD  
값 : 65534 (10진수)

![맥스포트](https://user-images.githubusercontent.com/40727649/73134349-ede9d800-4078-11ea-84db-b5f5309749ae.png)
- TIME_WAIT
키 이름 : TcpTimedWaitDelay  
키 타입 : DWORD  
값 : 30 (10진수)  

![타임웨잇](https://user-images.githubusercontent.com/40727649/73134348-ed514180-4078-11ea-8c33-f2954675fb69.png)

그럼 Connection refused 현상은 줄어들겠지?  
다시 돌려보면 빈도는 줄어들었어도 발생하긴 한다. 전에는 7~8번 발생하면 이번에 3~4번 발생한다. 왜그럴까 매우 궁금하다. nGrinder 컨트롤러와 에이전트를 한 인스턴스에서 구성해서 그럴까? 아니다. 여러모로 생각해 볼때 서버 문제가 맞는 것 같다. 설정이 제대로 먹었는지 확인도 잘 안되고... 부하도 100유저면 그리 크지 않다고 생각하는데.. 네트워크, OS 좀 안다고 생각했는데 왜인지 잘 모르겠다. 해결하면 이어서 포스트에 채우겠다.  
