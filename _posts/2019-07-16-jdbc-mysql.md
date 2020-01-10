---
published: true
layout: single
title: "Spring Boot MySQL 연결"
category: Spring
comments: true
---

스프링 부트 프로젝트에서 MySQL 설정하느라 반나절 삽질했다.  
`dependency`부터 `application.properties`까지 정리해본다.

## Gradle
`build.gradle`
```gradle
dependency { 
    implementation 'org.springframework.boot:spring-boot-data-jpa' //JPA 사용
    runtimeOnly 'mysql:mysql-connector-java'
}
```

## application.properties
[Common Properties](https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html) 공식사이트.  

```gradle
spring.datasource.url=jdbc:mysql://localhost:3306/[SCHEMA]?serverTimezone=UTC
spring.datasource.driverClassName=com.mysql.cj.jdbc.Driver
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5Dialect
spring.datasource.username=[Database Username]
spring.datasource.password=[Database Password]
```

`?serverTimezone=UTC`는 서버 타임존 설정이다. 한국 표준시 `KST`가 인식되지 않는다. `Asia/Seoul`은 되는 것 같다.
`~.dialect` 속성도 중요하다. 드라이버 커넥션에 `mysql`이 있으면 될 줄 알았는데 `dialect`속성도 잡아줘야 한다.
`spring.jpa.hibernate-ddl` 속성도 중요하다. 현재 DB에 스키마가 정의되어 있지 않으면 `create`값을 주어야 한다.
한번이라도 사용해서 스키마가 있다면, `none`이나 `false`를 주어 매번 생성되지 않게 해야 한다.

## 번외
인메모리 테스트용 DB`h2` 설정

`build.gradle`

```gradle
dependency {
    implementation 'com.h2database:h2'
}
```

`application.properties`
```
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=[USER_NAME]
spring.datasource.password=[USER_PASSWORD]
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console
```

`localhost:8080/h2-console`로 이동하면 h2 콘솔로 진입할 수 있다.  
testDb path가 `jdbc:h2:mem:testdb`인지 확인하자.
