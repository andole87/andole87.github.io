---
published: true
layout: single
title: "Spring Boot Login 구현 - Interceptor"
category: TIL
comments: true
---

## Session & Cookie
`HTTP`는 `Stateless` 프로토콜이다. 즉, 클라이언트와 서버는 한 번의 트랜잭션 이후 서로의 상태를 알지 못한다.  
github.com에 로그인 한 후 페이지를 이동할 때마다 로그인 해야 한다면, 암 환자가 폭증할 것이다.  
`Stateless`한 `HTTP`에 자격을 증명할 수 있게 하는 방법 중 하나가 `Session` 또는 `Cookie`다.  
로그인 이후에는 `Session`이나 `Cookie`에 로그인 정보를 저장, 클라이언트가 요청할 때마다 서버에 같이 실어서 보낸다.  
서버는 받아보고 로그인 중인 클라이언트인지 알아낼 수 있다.  
그럼에도, 백엔드 시스템에서 매번 `Session`을 검사 해야하는 것은 고된일이다. 모든 엔드포인트마다 세션 검사를 진행할 수는 없는 노릇.  
그래서 스프링은 `Interceptor`라는 계층을 제공한다. `Interceptor`에서 로그인 검증을 수행하면, `Controller`에서는 로그인을 검사할 필요가 없다.  

## Interceptor
![interceptor](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=http%3A%2F%2Fcfile10.uf.tistory.com%2Fimage%2F992590395ABF406F180F86)
> 출처: [https://victorydntmd.tistory.com/176](https://victorydntmd.tistory.com/176)

`Interceptor`는 `Request`를 가로챈다. `Request`는 `DispatcherServlet`으로부터 `Controller`로 전달되는데, `Interceptor`는 `Controller` 앞단에서 요청을 가로챈다.  
`Interceptor`의 `preHandle()` 메서드에서 `true`를 반환하면 `Controller`로 요청을 이어주고 `false`를 반환하면 `Controller`에 넘겨주지 않는다.  

## ATDD
로그인과 무관한 `Controller`의 테스트는 어떻게 해야 할까? 요청이 컨트롤러에 전달되기 위해서는 `Interceptor`계층을 통과해야 한다.  
`WebTestClient`는 어떻게 `Interceptor`의 방해를 받지 않고 `Controller`에 진입하게 할 수 있을까?

최초 발급받는 `Session`을 캐치하여 요청의 `Header`에 포함시키면 된다.  

```java
// 실패하는 메서드. 인터셉터의 방해를 받고 로그인을 요구받는다.
@Test
public void FailTest() {
    webTestClient.get().uri("SOME_URL")
    .exchange()
    .expectStatus().isOk();
}

@Test
public void SuccessTest() {
    // 로그인 과정. 로그인 후 받는 cookie를 할당한다.
    String cookie = webTestClient.post().uri("LOGIN_URL")
                .contentType(MediaType.APPLICATION_FORM_URLENCODED)
                .body(BodyInserters.fromFormData("ID", "ID")
                        .with("PASSWORD", "PASSWORD"))
                .exchange()
                .returnResult(String.class).getResponseHeaders().getFirst("Set-Cookie");

    // 성공
    webTestClient.get().uri("SOME_URL")
        .header("Cookie",cookie)
        .exchange()
        .expectStatus().isOk();
}
```

당연하지만 `@BeforeEach` 메서드에 `Cookie` 세팅 부분을 넣어두면 모든 테스트에 적용 가능하다.

> 주의)  
> `WebTestClient`는 같은 메서드 바디에 있어도 매 요청시 새로운 세션을 구성한다.
