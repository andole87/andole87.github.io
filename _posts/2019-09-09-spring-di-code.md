---
published: true
layout: single
title: "스프링 컨테이너 직접 제어해보기"
category: Spring
comments: true
---

[스프링 공식문서 읽기](/spring/spring-official-di/)에 이어 코드로 스프링 컨테이너의 동작을 알아본다.

# SpringBootApplication

스프링 부트 프로젝트를 시작하면, 어플리케이션의 메인 메서드를 갖는 클래스에 `@SpringBootApplication` 애너테이션이 붙어 있다. 메인 메서드에는 달랑 `SpringApplication.run(APP.class)` 한줄 있다. 

`@SpringBootApplication` 애너테이션과 `SpringApplication.run()` 메서드가 많은 일을 하고 있는 것을 알 수 있다. `org.springframework.bean`, `org.springframework.context` 패키지가 대부분의 호출을 받는다. 중단점을 찍고 하나씩 구경해보는것도 좋은 것 같다. 

여기서는 스프링 컨테이너를 자동으로 띄우지 않고, 수동으로 스프링 컨테이너를 만들고 빈을 사용해본다.

# ApplicationContext

`ApplicationContext`나 `BeanFactory` 모두 인터페이스이므로 직접 인스턴스화가 불가능하다. 

`ApplicationContext`의 구현체 중 하나인 `GenericApplicationContext`는 설정 정보 없이도 바로 생성이 가능하다. 다른 몇 가지 구현체는 설정정보를 넣어줘야 하기 때문에 나중에 다룬다.

`ApplicationContext`에 빈을 등록하기 위해 크게 두가지 방법을 사용할 수 있다.

- `ApplicationContext` 에 `BeanDefinition` 객체를 등록하는 방법
- `ApplicationContext` 내부 `BeanFactory`에 이미 생성된 빈을 등록하는 방법

## BeanDefinition

{% gist andole98/1033bdf794c38ce46ddbd752538e73e2 %}

코드 어디에서도 `BeanA` 를 생성하지 않았음에도 `BeanA` 를 사용할 수 있다.

간단한 로그가 출력되는데, 다음과 같은 내용을 확인할 수 있다.

```
org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'beanA'
```

`BeanDefinition` 을 컨테이너에 제출하고 `refresh()` 하면 컨테이너가 알아서 빈을 생성하고 관리한다는 것을 알 수 있다.

## registerSingleton

{% gist andole98/6da14ece6afc9b97d4f829d787b4ed3c %}

이번에는 `BeanA` 를 직접 생성하고, `BeanFactory` 에 수동 등록했다. 똑같은 결과가 나오긴 하지만, `BeanDefinition`을 사용했을 때 보았던 로그가 없다. 컨테이너가 스스로 빈을 만들지 않았다는 뜻이다.


## ComponentScan

위에서 수동으로 `ApplicationContext` 를 구성해보았다. 각 빈마다 일일이 `BeanDefinition`이나 인스턴스를 context에 제공해야 했다.

실제로 스프링이 구동될 때는 어떻게 `ApplicationContext`가 구성되는지 궁금해진다.

위에서 잠깐 언급한 `@SpringBootApplication` 애너테이션을 확인해보면 `@ComponentScan` 을 확인할 수 있다. 이 애너테이션은 컨테이너를 시작하기 전에 컴포넌트들을 스캔해서, 위에서 수동으로 한 작업을 자동으로 수행한다.

`@ComponentScan` 애너테이션으로 ApplicationContext를 구성하려면 `GenericApplicationContext` 대신 `AnnotatedConfigApplicationContext` 구현체를 사용해야 한다.

{% gist andole98/8857202e33eaf6db679d37b719fb2a0a %}

수동으로 작업했던 것과 똑같이 수행된다. 

로그를 살펴보면 좀 다른 것을 발견할 수 있다.

```
Creating bean 'InternalConfigurationAnnotationProcessor'

ClassPathBeanDefinitionScanner - Identified candidate component class : file [FILE_PATH_TO_PROJECT_PACKAGE/BeanA.class]

Creating bean 'InternalEventListenerProcessor'

Creating bean 'InternalEventListenerFactory'

Creating bean 'InternalAutowiredAnnotationProcessor'

Creating bean 'InternalCommonAnnotationProcessor'

Creating bean 'InternalPersistenceAnnotationProcessor'

Creating bean 'AnnoatatedContextApplication'

Creating bean 'BeanA'
```

실제 로그는 조금 다르지만 요약해서 적었다.

필요한 빈은 딱 하나 `BeanA`였지만 그 전에 여러 빈들이 생성되었다. 

- `AnnotationProcessor`와 관련된 빈들이 생성되었다.
- `BeanA`의 파일시스템상 Path를 확인했다.
- `InternalEvent`와 관련된 빈들이 생성되었다.
- `ApplicationContext`도 빈으로 생성되었다.
- `BeanA`가 마지막으로 빈으로 생성되었다.

---

자세한 동작은 아직 모르겠지만 로그를 바탕으로 몇 가지 추측해봤다.

1. 어노테이션을 핸들링할 빈을 제일 먼저 만든다. 아마 설정정보를 찾아 준비하고 컴포넌트를 스캔하기 위해서인 것 같다.
2. 생성된 어노테이션 프로세서가 유저레벨의 빈 후보들을 마킹한다.
3. 이벤트리스너를 빈을 만든다. 무엇인가가 이벤트를 퍼블리싱하는 것 같다. 누가 어떤 이벤트를 언제 퍼블리시하는지는 아직 잘 모르겠다.
4. 빈들을 autowiring할 InternalAutowiredAnnotationProcessor빈을 생성한다. 아직 유저 레벨의 빈이 쓰이지 않는데다, 이름으로 보아 스프링 컨테이너 내부적으로도 @Autowired 애너테이션을 사용하는 것 같다.
5. 지금까지 만들어둔 빈으로 스프링 컨테이너(ApplicationContext)를 빈으로 만든다.
6. 마킹해둔 유저레벨의 빈들을 싱글턴으로 생성한다.



## 마치며

공식문서를 읽고 실제 코드를 좀 읽어봤다. 여러가지 인터페이스와 추상 클래스들이 섞여 있어 한번에 이해하긴 어려웠지만, 일관된 네이밍과 구현 패턴으로 나같은 초보도 조금은 알아들을 수 있었다.

메서드나 클래스 이름을 잘 지어두면 이렇게 좋구나 하는 걸 느꼈다. 인터페이스와 구현 클래스 사이에는 언제나 추상 클래스가 있었다. 추상 클래스에서 인터페이스의 대부분을 구현하고, 구현 클래스의 이름과 연관된 부분만이 각 클래스에 있었던 점도 매우 잘 정리된 느낌이었다. 덕분에 코드 읽기가 편했고 동작을 유추하기도 (비교적...)쉬웠던 것 같다.

그래서인지 미지의 무언가를 대하는 것은 원래 어려운 일인데 굉장히 재미있었다. Rod Johnson이 적어놓은 주석이나 가끔 보이는 // TODO 주석도 재미있었다.

`volatile`, `synchronized` 키워드도 좀 보였다. 낯설지만 자주 보이는 컬렉션 `ConcurrentHashMap`도 있었다. 동시성을 갖춘 해시맵인 것은 알겠는데 어떻게 동시성을 갖는지는 잘 모르겠다. 한번 알아둬야겠다 느꼈다.

다음엔 DispatcherServlet을 다룰 예정이다.
