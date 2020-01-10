---
published: true
layout: single
title: "Spring Boot Logger - HTTP 통신 로그 logback-access"
category: Spring
comments: true
---
스프링 프로젝트를 로컬에서 실행해보면, 콘솔창에 주루룩 무언가 표시된다. 

![스프링_실행시_콘솔](/assets/info.png)

스프링이 스스로 실행하면서 단계별로 자신의 상태를 콘솔에 출력하는 것이다. 이렇게 애플리케이션의 동작이나 이벤트를 실행하며 남기는 메시지를 **로그**라고 하며, **로그**를 구성하는 것을 **로깅**이라고 한다. 이 로그들을 겁먹지 말고 하나씩 읽어보면 스프링을 이해하는데 도움이 되는 것 같다. (그렇다고 한다)  

(스프링 뿐만 아니라) 로그는 레벨이 있다. 보통 ERROR > WARNNING > INFO > DEBUG > TRACE 순으로 자세하다. `TRACE`레벨은 스택 트레이스처럼 매우 자세한 로그들까지 로깅한다. 하지만 너무 많은 로그는 오히려 알아보기 어렵고 성능에 악영향을 준다. `ERROR`는 에러시에만 로그를 남기므로 성능에 주는 영향이 적다. 그러나 주요 이벤트나 관심사항이 로그로 남지 않는다.  

스프링 기본 로깅 레벨은 `INFO`다. 개발할 때는 이보다 자세한 `DEBUG`를 추천한다. 어떤 요청이 왔고, 파라미터는 무엇이고, 어느 컨트롤러 어느 메서드로 매핑했는지, 결과는 어떤지 알 수 있기 때문.
스프링의 `application.properties`에 간단히 `debug=true`만 잡아주면 된다.  

![DEBUG_콘솔](/assets/debug.png)

여기서 더 나아가서, HTTP 요청과 응답의 원본을 보고 싶다면 `logback - access` 라이브러리 사용을 권한다. 특히 웹 초보자라면 웹의 기반 기술을 익히는데 큰 도움이 된다.  

![logback-access](/assets/logback-access.png)

## Logback-access

의존성 추가가 필요하다. `build.gradle`에 다음을 추가하자.

```groovy
dependacy {
    runtimeOnly 'net.rakugakibox.spring.boot:logback-access-spring-boot-starter:2.7.1'
}
```

`logback-acess`의 설정도 필요하다. 로그 포맷을 결정해줘야 한다.
설정 파일 위치에 `logback-acess.xml`파일을 생성한다. 설정 파일들의 기본 위치는 `src/main/java/resources/`이다. 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%fullRequest%n%n%fullResponse</pattern>
        </encoder>
    </appender>
    <appender-ref ref="CONSOLE"/>
</configuration>
```
UTF-8 인코딩을 사용하며, 콘솔에 출력하고, HTTP리퀘스트와 HTTP리스폰스 전체를 출력한다는 설정이다.

