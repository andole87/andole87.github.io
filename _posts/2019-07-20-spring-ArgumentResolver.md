---
published: true
layout: single
title: "Spring Boot ArgumentResolver - Request 객체 매핑"
category: Spring
comments: true
---

## ArgumentResolver
리퀘스트의 여러 정보를 객체로 매핑하여 컨트롤러에 전달하는 역할을 수행하는 것들을 `ArgumentResolver`라고 한다.  
`POST` 요청의 경우 단순 `Dto`로 바로 매핑할 수 있으나, `Session`과 같은 `HttpServletRequest`와 관련한 정보들은 바로 매핑할 수 없다.  
`Controller`에서는 일일이 `HttpSession`을 받아 처리해야 한다.  
`ArgumentResolver`는 컨트롤러 파라미터로 무엇이 필요한지 인지하고, 관련 정보를 객체로 매핑하여 컨트롤러에 전달해준다.

`ArgumentResolver`를 사용하기 위해선 `HandlerMethodArgumentResolver` 인터페이스를 구현한 클래스와 `WebMvcConfigurer`의 설정, 마지막으로 매핑될 클래스 세 가지가 필요하다.  

컨트롤러에서 로그인 상태를 파악하기 위해서 세션 정보를 받아야 한다고 가정하고 예시를 남긴다.  

```java
// ArgumentResolver 사용 전
@Controller
public LoginController {
    ...

    @GetMapping(...)
    public String login (HttpSession session) {
        if (session.getAttribute("userName") == null) {
            return "signup";
        }
        return "login";
    }
    ...
}
```

## Parameter Class
컨트롤러가 받을 파라미터 클래스다. 로그인을 위해 세션정보를 받을 `UserSessionInfo`를 준비한다.
```java

public class UserSessionInfo {
    private String name;
    private String email;
    ...
    constructor, getter, setter
    ...
    public boolean isCorrectUser () {
        ...         
    }
}
```


## HandlerMethodArgumentResolver
메서드 2개를 오버라이드해야한다.
- supportsParameter : 컨트롤러에 매핑해서 전달할 파라미터 타입을 정의한다.
- resolveArgument : 컨트롤러에 전달할 파라미터를 구성한다.

```java
public class SessionArgumentResolver implements HandlerMethodArgumentResolver {
    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        return parameter.getParameterType() == UserSessionInfo.class;
    }

    @Override
    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {
        HttpServletRequest request = (HttpServletRequest) webRequest.getNativeRequest();
        HttpSession session = request.getSession();
        return new UserSessionInfo(String.valueOf(session.getAttribute("userName")), String.valueOf(session.getAttribute("userEmail")));
    }
}
```
> 주: session.getAttribute() 의 반환 타입은 Object다. Object.toString()하면 String을 얻을 수 있다. 하지만 Object가 null일 경우 NPE가 발생한다.
> Object.toString() 대신 String.valueOf(Object)를 사용하자.

## WebMvcConfigurer 
준비한 `ArgumentResolver` 사용을 위해 설정이 필요하다.  
`WebConfigurer`를 `implements`한 클래스가 필요하다.
```java
@Configuration
public class MyblogConfiguration implements WebMvcConfigurer {
    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
        resolvers.add(new SessionArgumentResolver());
    }
}
```

## Controller
컨트롤러는 `HttpSession`대신 `UserSessionInfo`를 받아 활용할 수 있다.
```java
// ArgumentResolver 사용 전
@Controller
public LoginController {
    ...

    @GetMapping(...)
    public String login (UserSessionInfo userInfo) {
        getLoginForm(userInfo);
    }

    private String getLoginForm (UserSessionInfo userInfo) {
        if (userInfo.isCorrect()) {
            return "login";
        }
        return "signup";
    }
    ...
}
```