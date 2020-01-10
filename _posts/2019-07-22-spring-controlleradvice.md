---
published: true
layout: single
title: "Spring Boot Controller Advice - 컨트롤러 예외처리"
category: Spring
comments: true
---

## Controller Advice
> 컨트롤러가 Exception을 던졌을 때 Controller Advice가 캐치하여 처리한다.

특정 리퀘스트를 받았을 때, 리퀘스트를 처리하다 Exception을 만날 수 있다. 컨트롤러가 직접 `try-catch`할 수 있지만, 중복을 피할 수 없다.  
스프링에서 `ControllerAdvice`를 제공한다. `ControllerAdvice`는 설정에 따라 모든 컨트롤러, 특정 컨트롤러, 특정 패키지 하위 컨트롤러 등에 일괄 적용할 수 있다.  
일관된 예외 처리를 보장할 수 있으므로 무조건 사용하는게 좋겠다.  
특히 **로그인 처리**와 관련해서는 쓰임새가 아주 좋다.

`Controller Advice`를 사용하기 위해선 `ControllerAdvice`구현체가 필요하다.

## ControllerAdvice
- 애너테이션 `@ControllerAdvice`를 추가한다. 애너테이션 안에서 특정 컨트롤러인지, 모든 컨트롤러인지, 어떤 패키지 아래인지 설정할 수 있다.
- 예외를 처리할 메서드에 `@ExceptionHandler` 애너테이션을 추가한다. 애너테이션 안에서 어떤 `Exception`을 처리할 지 명시한다.

```java
@ControllerAdvice(annotations = Controller.class)
public class MyControllerAdvice {
    
    @ExceptionHandler(MyException.class)
    @ResponseStatus(value = HttpStatus.NOTFOUND)
    public String myHandler (MyException e, Model model) {
        // error.html의 ${error} 변수를 e.getMessage()로 주고 리턴한다.
        model.addAttribute("error", e.getMessage());
        return "error";
    }
}
```
