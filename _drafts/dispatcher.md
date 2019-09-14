# DispatcherServlet

다른 웹 프레임워크처럼, Spring MVC는 중앙 서블릿인  DispatcherServlet을 프론트 컨트롤러로 활용한다. DispatcherServlet은 요청 처리를 위한 공유 알고리즘을 제공하고, 실제 작업은 구성 가능한 요소에 의해 진행되도록 위임한다. 이 모델은 유연하며 다양한 동작 흐름을 제공한다.

다른 Servlet들 처럼 DispatcherServlet은 서블릿 스펙에 따라 설정을 선언하고 매핑해야 한다. DispatcherServlet은 스프링 설정을 사용하여 Request Mapping, View Resolution, Exception handling을  확인한다.

## Context Hierarchy

DispatcherServlet에는 WebApplicationContext(ApplicationContext의 확장) 가 필요하다. WebApplicationContext는 ServletContext, Servlet과 연관된 링크를 갖고 있다.

대부분의 어플리케이션에는 하나의 WebApplicationContext만 있어도 간단하고 충분하다. 하나의 WebApplicationContext가 각각의 하위 WebApplicationContext 구성과 함께 여러 DispatcherServlet 인스턴스에서 공유되는 컨텍스트 계층 구조를 가질 수도 있다.

Root WebApplicationContext는 보통 Infrastructure Bean을 포함한다. 여러 서블릿들이 공유해야 하는 Data Repository나 비즈니스 서비스들이다. 이 빈들은 하위 WebApplicationContext로 효과적으로 상속되거나 재구성된다.







## Special Bean Types

DispatcherServlet은 다음 표에 있는 빈들을 사용한다.

| Bean Type                | Explanation                                                  |
| ------------------------ | ------------------------------------------------------------ |
| HandlerMapping           | 요청을 핸들러에 매핑한다. 핸들러는 전처리나 후처리를 위한 Interceptor를 가질 수 있다. `HandlerMapping`의 구현 디테일에 따라 매핑된다.  <br /> HandlerMapping의 두 주요 구현체는 RequestMappingHandlerMapping (@RequestMapping), SimpleUrlHandlerMapping이다. |
| HandlerAdapter           | DispatcherServlet이 매핑된 핸들러를 호출하는 것을 돕는다. 핸들러가 실제로 호출되는것과 관계 없이 수행된다.(Adapter에 주목) <br /> HandlerAdapter의 주된 목표는 DispatcherServlet이 요청을 위임하는데 추상화를 제공하는 것이다. |
| HandlerExceptionResolver | 요청을 핸들러에 매핑하거나, HTML에러를 보여주거나, 기타 동작 중에 예외를 인지하고 결정하는 전략을 담당한다. |
| ViewResolver             | 핸들러가 반환한 문자열 기반으로 뷰 네임을 결정하고 응답을 렌더링한다. |
| LocaleResolver           | 글로벌 서비스를 제공할 수 있도록, Locale을 결정한다.         |
| ThemeResolver            | 어플리케이션이 사용할 수 있는 테마를 결정한다.               |
| MultipartResolver        | Multi-part 요청(파일 요청)에 대해 추상화된 파싱방법 또는 파싱 라이브러리를 제공한다. |
| FlashMapManager          | input, output FlashMap을 저장하고 불러온다. FlashMap은 어떤 요청으로부터 다른 요청으로 attributes들을 전달할 수 있는데, 주로 Redirect에 사용된다. |



## Web MVC Config

어플리케이션은 위 표에 있는 Special Bean들, 요청을 처리하기 위한 빈들을 추가하거나 제어할 수 있다. DispatcherServlet은 WebApplicationContext를 통해서 이 빈들을 체크한다. 이 과정이 실패하면 `DispatcherServlet.properties`에 있는 기본값으로 설정된다.

대부분의 경우, MVC Config가 적절한 시작점이 된다. MVC Config는 XML이나 자바 코드로 필요한 빈들을 정의할 수 있다.

## Servlet Config

서블릿 3.0 이상의 환경에서, 서블릿 컨테이너를 `web.xml` 대신 프로그래밍적으로 설정할 수 있다.

```java
import org.springframework.web.WebApplicationInitializer;

public class MyWebApplicationInitializer implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext container) {
        XmlWebApplicationContext appContext = new XmlWebApplicationContext();
        appContext.setConfigLocation("/WEB-INF/spring/dispatcher-config.xml");

        ServletRegistration.Dynamic registration = container.addServlet("dispatcher", new DispatcherServlet(appContext));
        registration.setLoadOnStartup(1);
        registration.addMapping("/");
    }
}
```

`WebApplicationInitializer` 는 Servlet3 컨테이너를 시작하기 위해 제공되는 인터페이스다. `AbstractDispatcherServletInitializer`는 이 인터페이스를 구현하는 기본 템플릿이다. 더 쉬운 자동 설정을 지원한다.

다음과 같이 설정하는 것은 추천한다.

```java
public class MyWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {
    @Override
    protected Class<?>[] getRootConfigClasses() {
        return null;
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class<?>[] { MyWebConfig.class };
    }

    @Override
    protected String[] getServletMappings() {
        return new String[] { "/" };
    }
}
```

`AbstractDispatcherServletInitializer`는 `Filter` 객체를 편리하게 추가하는 것도 지원한다.

```java
public class MyWebAppInitializer extends AbstractDispatcherServletInitializer {
    @Override
    protected Filter[] getServletFilters() {
        return new Filter[] {
            new HiddenHttpMethodFilter(), new CharacterEncodingFilter() };
    }
}
```



## Processing

`DispatcherServlet` 은 다음과 같이 요청을 처리한다.

1. 컨트롤러나 기타 요소가 사용할 WebApplicationContext를 요청에 바인딩한다. 기본으로 `DispatcherServlet.WEB_APPLICATION_CONTEXT_ATTRIBUTE` 키에 바인딩된다. 
2. `LocaleResolver`를 요청에 바인딩한다. `Locale`은 요청을 처리할 때 (View 렌더링 등)에 사용된다. `Locale` 을 결정할 필요 없다면 `LocaleResolver`가 없어도 된다.
3. `ThemeResolver`를 요청에 바인딩한다. `ThemeResolver`는 뷰와 같은 요소가 테마를 결정할 때 사용한다. 테마를 사용하지 않으면 무시해도 된다.
4. 특정한 `MultipartFileResolver`를 지정했다면 요청에서 파일을 검사한다. 파일이 발견되면 요청은 추가 처리를 위해 `MultipartHttpServletRequest`로 랩핑된다.
5. 적절한 핸들러를 찾는다. 핸들러를 찾으면 모델 또는 렌더링을 준비하기 위해 핸들러와 연관된 실행 흐름(`execution chain`),  `preprocessor`, `postprocessor`, `controller` 등을 실행한다. 또 애너테이션이 부여된 컨트롤러의 경우 뷰를 반환하는 대신 HandlerAdapter 내에서 응답을 처리할 수도 있다.
6. 핸들러가 모델을 리턴하면 뷰가 렌더링된다. 모델이 리턴되지 않으면 (요청을 intercept하는 `preprocessor` 나 `postprocessor` 때문일 수 있음) 요청이 이미 이행되었을 수 있으므로, 뷰가 렌더링되지 않는다.
7. `WebApplicationContext`가 관리하는 `HandlerExceptionResolver` 빈은 위의 요청을 처리하는 중에 발생한 예외를 처리한다.

## Interception

모든 `HandlerMapping` 인터페이스 구현체는 핸들링 인터셉터를 제공한다. 인터셉터는 특정 요청마다 정해진 기능을 수행할 때 유용하다. 아마도 `Principal`을 체크하거나, 로그를 남기는 기능 들이 이에 포함될 것이다.

`Interceptor`는 반드시 `org.springframework.web.servlet`패키지의  `HandlerInterceptor`를 구현해야 한다. 이 인터페이스에는 세가지 메서드가 있는데, 이 메서드는 모든 전처리, 후처리를 커버할 수 있다.

- `preHandle(...)` : 실제 핸들러가 실행되기 전
- `postHandle(...)` : 핸들러가 실행되고 난 후
- `afterCompletion(...)` : 요청 처리가 끝나고 난 후

`preHandle(...)` 메서드는 불리언을 반환한다. 그래서 실행 흐름(execution chain)을 break하거나 continue하게 만들 수 있다. `ture`를 반환하면 실행 흐름이 계속해서 이어진다.(continue) `false`를 반환하면 `DispatcherServlet`은 인터셉터 스스로 요청을 처리했다고 가정하고 다음 실행 체인(다른 interceptor를 포함한)을 수행하지 않는다.(break)

`HandlerAdapter`에서 응답을 작성하기 위한 목적이라면, `postHandle` 보다는 `@ResponseBody`나 `ResponseEntity` 가 더 유용하다. 그러므로 응답 헤더를 변경하는 것 같이, 어떤 응답을 변경하는 것은 성능이 좋지 못하다. 이 경우, `ResponseBodyAdvice`를 구현하고 `ControllerAdive`에서 설정하거나 `RequestMappingHandlerAdapter`에 직접 설정하는게 좋다.

## Exception

요청을 매핑하면서 예외가 발생하거나, 요청을 처리하면서(@Controller같은 곳에서) 예외가 발생하면 `DispatcherServlet`은 `HandlerExceptionResolver` 에 예외를 결정하고, 예외를 처리할 수 있도록 위임한다.

다음은 `HandlerExceptionResolver`의 구현체 목록이다.

| HandlerExceptionResolver          | Description                                                  |
| --------------------------------- | ------------------------------------------------------------ |
| SimpleMappingExceptionResolver    | 예외 클래스 이름과 `Error View` 이름을 매핑한다. 브라우저 어플리케이션에서 에러 페이지를 렌더링할 때 유용하다. |
| DefaultHandlerExceptionResolver   | Spring MVC가 발생시킨 예외를 HTTP 상태 코드에 매핑한다. 비슷한 역할을 하는 `ResponseEntityExceptionHandler`도 있다. |
| ResponseStatusExceptionResolver   | `@ResponseStatus`애너테이션으로 예외를 매핑한다. HTTP 상태 코드도 애너테이션에서 정의한 대로 매핑한다. |
| ExceptionHandlerExceptionResolver | `@ExceptionHandler`, `@Controller`, `@ControllerAdvice` 클래스의 메서드로 예외를 매핑한다. |

### Chain of Resolvers

여러 `HandlerExceptionResolver`빈을 정의하는 것으로 ExceptionResolver를 구성할 수 있다. 스프링 설정으로 이 Resolver들의 순서를 설정할 수 있다. 높은(higher) 순서를 가질수록 Resolver가 체인의 뒤쪽에 위치한다.

`HandlerExceptionResolver`의 리턴값은 다음 세 가지 종류가 있다.

- 에러 뷰를 가리키는 `ModelAndView`를 리턴
- `Resolver` 내부에서 예외를 처리했다면 비어있는 `ModelAndView`를 리턴
- 예외가 아직 결정되지 않았다면, 후순위의 `Resolver`가 예외를 핸들링할 수 있도록 `null`을 리턴한다. 마지막까지 `null`이 계속해서 리턴된다면 `ServletContainer`로 버블링된다.



## View Resolution

스프링 MVC는 `ViewResolver`와 `View` 인터페이스를 제공한다.





