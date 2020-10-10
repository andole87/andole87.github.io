---
title: "JPA N+1 과 Projecting"
category: "Spring"
---

엔터티 관계 설정에 ManyToOne 단방향을 선호하는 편이다. One이 Many들을 갖고 있는게 직관적으로는 그럴듯 하지만 꼭 그럴 필요는 없기 때문이다. 게다가 데이터베이스에서는 Many가 One을 바라보는게 자연스럽다.

양방향이 어색하다 느껴지는 이유는 POJO에서 상호참조하는게 이상하게 느껴지기 때문이다. 조회 때문에 양방향 관계가 만들어진다는 것은 매우 공감하는 일이지만, 이런 경우는 대부분 어드민 프로젝트에서 발생한다. 비즈니스적으로 깊은 그래프 탐색이 필요하다면 엔터티설계를 그렇게 안 했을 것이다. 아마도. 어드민에서는 목적에 맞게끔 spring.data.jdbc를 이용해 복잡한 쿼리를 수행하는게 더 좋다고 본다.

그래서 N+1문제를 만날 일이 거의 없었다. 레거시에는 나름대로 해결책들을 준비해 두었고 내가 신규로 하는 프로젝트에는 ManyToOne 단방향을 썼으니. 그러나 이번에 N+1문제를 만나 조금 당황스러웠다.

## 상황

```
public class Order {
	...
}

public class Item {
	@ManyToOne Order order;
	private BigDecimal price;
	...
}

public interface ItemRepository extends JpaRepository<Item, Long> {
	// OK
	List<Item> findByOrder(Order order);
	
	// N+1!! WTF??
	List<Item> findByPriceGreaterThan(BigDecimal price);
}

```

Order를 기준으로 컬렉션을 조회했을땐 당연히 문제가 없지만, Item의 필드가 WHERE clause에 존재하면 N+1개 쿼리가 발생했다.

## TLDR;

### 원인

1. ItemRepository에서 쿼리 메서드가 만들어질때 Order는 관심사에 벗어난다.
2. 쿼리 메서드의 리턴 타입에 따라 쿼리가 결정된다.
3. FetchType.EAGER는 최초 쿼리 후, TwoPhaseLoader에서 핸들링된다. FetchType설정은 쿼리가 만들어질 때 영향을 주지 않는다.

### 해결

1. FetchType.LAZY
2. @EntityGraph
3. JPQL JOIN FETCH
4. Projecting

## 쿼리 메서드

JPA Repository는 신기하게도, 메서드 이름만 적절하게 지어주면 자동으로 쿼리를 만들어준다. 쿼리메서드라고 한다. 그냥 그렇다고만 알고 있었고 문서에서도 그렇게 이야기하길래 그런가보다 하고 넘어갔었는데, 이참에 차근히 짚어봤다.

어플리케이션 로딩 시점에, 스프링은 Repository를 스캔하고 해당 Repository의 프록시를 빈으로 등록한다. 대부분의 Repository는 구현체로 SimpleJpaRepository 를 사용하며, MethodInterceptor 를 기본으로 8개 등록한다.

즉, Repository 빈은 프록시이며, 타겟은 SimpleJpaRepository, 메서드 인터셉터에는 TransactionInterceptor, PersistenceExceptionTranslator ,

CrudMethodMetadataPostProcessor, QueryExecutionMethodInterceptor ...등이 등록된다.

QueryExecutionMethodInterceptor 는 필드로 RepositoryQuery를 가지고 있다. 이 인터셉터는 어플리케이션 로딩 시점에 쿼리 메서드를 읽어 RepositoryQuery를 만들어 둔다. 나중에 어플리케이션에서 쿼리 메서드를 호출하면, 이 인터셉터가 쿼리 메서드를 실행하도록 하는 것이다.

Repository의 구현체는 SimpleJpaRepository라고 했다. 이 구현체는 당연하게도 커스텀 쿼리 메서드를 구현하고 있지 않기 때문에, Repository 프록시는 타겟인 SimpleJpaRepository로 포워딩하지 않고 인터셉터에서 실행되도록 만들어 둔 것이다.

QueryExecutionMethodInterceptor 에서 RepositoryQuery를 만드는 과정을 보면, EntityManager.createQuery() 를 사용한다. 보통 커스텀 레포지터리에서 쓰이는 방식을 그대로 사용한다. 애너테이션이 붙은 쿼리, 저장 프로시저 쿼리, 네이티브 쿼리, 네임드 쿼리 등으로 구분해서 처리한다. 이번 케이스는 아무것도 해당사항이 없으므로 쿼리메서드로 처리된다. 쿼리 메서드의 구현체는 PartTreeQuery이고, 이름으로 알 수 있듯이 JPQL문을 트리구조로 파싱한다.

쿼리를 만들 때 주요하게 다뤄지는 부분 중 하나가 메서드의 ReturnType이다. 리턴 타입에 따라 쿼리와 동작이 바뀐다.

결론적으로 만들어지는 쿼리는 `SELECT i FROM item i WHERE i.price > :price` 와 같은 형태가 된다. 이 JPQL은 실제 쿼리가 사용될 때 SQL문으로 번역되어 데이터베이스로 날아간다.

여차저차해서 어플리케이션 로딩이 끝났고 만들어둔 레포지터리의 쿼리 메서드는 실행할 준비가 끝났다. 이제 쿼리를 실행해보자.

여러 전처리들이 실행된다. validation, 세션(커넥션) 얻어오기, 트랜잭션 ... 최종적으로org.hibernate.loader.Loader 에서 JPQL을 실제 SQL로 바꿔 실행된다.

`SELECT item.id, item.price, item.order_id FROM ITEM as item WHERE item.price > :price`

위 SQL을 실행한 후 ResultSet을 받는다. Loader는 ResultSet을 바로 리턴하지 않고 TwoPhaseLoad.initializeEntity()를 실행한다. 이름에서 알 수 있듯 TwoPhaseLoad는 필요한 필드가 갖춰져 있는지 검사하고, LazyLoad를 책임진다.

ResultSet을 순회하면서 엔터티 필드를 검사한다. 필드 중 LazyLoading이 있다면 스킵하고, 아니라면 추가적으로 쿼리를 수행해서 Entity가 만들어질 수 있는 ResultSet을 준비한다.

**이부분이 N+1이 발생하는 주요 포인트다. ResultSet을 한건씩 순회하면서 후속 처리를 수행하므로 N+1이 발생하는 것이다. 즉 Loader에서 최초 의도한 쿼리가 수행되고, TwoPhaseLoad에서 해당 필드가 LazyLoading 여부를 확인, 아니라면 추가 쿼리를 수행한다.**

현 상황에는 ManyToOne 관계이므로 디폴트로 EAGER가 잡혀 있는 상태다. 그러므로 TwoPhaseLoad에서 필드를 검사할 때 추가적으로 필요하다 판단하고 추가 쿼리를 수행하는 것이다.

FetchType.LAZY로 변경하면 어떻게 될까? TwoPhaseLoad에서는 조건문에 들어가지 않으므로 스킵된다. TwoPhaseLoad가 끝나면 Loader는 이벤트를 발행한다. 기본으로 구성된 이벤트리스너는 DefaultLoadPostProcessor가 있다. 이 프로세서는 이벤트를 받아서 프록시 객체를 만들어야 하는지 판단하고 프록시를 구성한다. 잘 알려진 대로 프록시는 LazyLoading 필드에 접근이 필요할 때 추가 쿼리를 수행한다.

자, 그럼 N+1문제를 해결할 수 있는 포인트를 알게 되었다. 크게 두가지 방법이 있겠다.

- FetchType.LAZY로 바꾸는 것. 가장 간단하게 해결할 수 있으나, 모든 Item의 Order가 필요하다면(직렬화 등..) 의미없는 방법이다.
- TwoPhaseLoad가 처리할 일이 없게 만드는 것. 최초 Loader에서 필요한 필드를 모두 가져오면 된다.
    1. EntityGraph
    2. JPQL JOIN FETCH
    3. Return Type 변경 (Projection)

### EntityGraph

@EntityGraph 애너테이션으로 필요한 객체 그래프를 설정할 수 있다. 이런 경우 쿼리 메서드가 만들어지는 시점에 다른 쿼리가 만들어진다.

즉, EntityManager.createQuery() 부분에서 다른 처리가 들어간다.

```
org.springframework.data.jpa.repository.query.AbstractJpaQuery

...
protected Query createQuery(JpaParametersParameterAccessor parameters) {
	return applyLockMode(
						applyEntityGraphConfiguration(
								applyHints(
										doCreateQuery(parameters),
							  method), 
						method), 
					method);
}
...

```

쿼리를 만들고, 힌트를 적용한 다음, EntityGraph 설정을 적용하고, Lock모드를 설정한다. 애너테이션으로 설정하지 않고, 커스텀 레포지터리에서 쿼리를 직접 만들수도 있다.

EntityGraph가 설정된 쿼리는 다음과 같다.

```
SELECT order.id, item.id, item.price, item.order_id 
FROM ITEM item 
LEFT JOIN ORDER order ON item.id = order.id 
WHERE item.price > :price
```

최초 Loader에서 SQL을 수행할 때 필요한 필드를 모두 가져오므로 TwoPhaseLoader에서 추가로 수행할 것이 없다.

문제가 될만한 부분은 LEFT JOIN 이다. INNER JOIN으로 수행하려면 EntityGraph로는 불가능하다. EntityGraph는 조인과는 약간 다른 개념이다. 타겟 엔터티를 가져오는데 디펜던시 그래프를 어디까지 그릴 것인지가 목적이므로 LEFT JOIN을 수행하는게 당연하고 또 그래야 한다.

### JOIN FETCH

가장 일반적인 방법이다. 커스텀 레포지터리를 만들고 JPQL을 직접 작성하는 것이다.

```
SELECT i FROM Item i INNER JOIN FETCH i.order WHERE i.price > :price
```

EntityGraph와 마찬가지로 TwoPhaseLoader에서 추가 수행할 작업이 없으므로 N+1 문제가 발생하지 않는다.

### Projecting

이게 왜 되는지 궁금해서 위 모든 것을 알아보게 되었다. 쿼리 메서드가 엔터티를 반환하지 않고, 다른 타입을 반환하게 만드는 것이다. JPA에서는 Projecting이라고 부른다. 가져온 엔터티를 가지고 리턴 타입으로 투영 or 주입한다.

```
public interface ItemMapping {
	Order getOrder();
	BigDecimal getPrice();
	Long getId();
}

public interface ItemRepository extends JpaRepository<Item, Long> {
	List<ItemMapping> findByPriceGreaterThan(BigDecimal price);
}

```

Entity와 아무 관계 없는 Interface를 리턴하게 바꿨다. 인터페이스와 엔터티 필드 이름을 매핑해주면 쿼리 결과로 해당 인터페이스를 리턴해준다. 인터페이스 구현체는 AbstractJpaQuery.TuperConverter.TupleBackedMap 이다.

어플리케이션 로딩 시점에 쿼리를 만들어서 준비한다고 했다. 쿼리 메서드의 리턴 타입으로 SELECT 절을 결정한다고 했다. 리턴 타입이 인터페이스라면, 특별한 처리를 해준다. 컨버터로 TupleConverter를 지정해서 ResultProcess할 수 있도록 한다.

인터페이스 메서드들과 엔터티 필드를 체크해서, 가능한 필드인지 검사한다. 만약 필드와 매핑되지 않는 메서드가 있다면 어플리케이션 로딩에 실패한다. ItemMapping 에는 getOrder() 가 있으므로, Order 엔터티가 필요하다는 것을 JPA는 확인한다. 생성하는 JPQL에 INNER JOIN FETCH 가 추가된다.

Loader에서 처음 쿼리를 수행하고, TwoPhaseLoader에서는 리턴타입이 인터페이스라면 추가 작업을 하지 않고 넘긴다. 이후에 TupleConverter에서 ResultSet을 Tuple로 변환하고 TupleBackedMap으로 저장한다.

서비스 레이어로 넘어간 ItemMapping 은 메서드가 호출될 때마다, Tuple로부터 값을 꺼내오게 된다.

이게 Projecting의 과정이다. 오픈 프로젝팅, 다이나믹 프로젝팅도 가능하지만 좋은 방법 같지 않으므로 남기지 않는다. 오픈 프로젝팅은 SpEL을 사용하므로 오버헤드가 크다. 다이나믹 프로젝팅은 런타임에 에러가 발생할 수 있다. 안쓰는게 좋을 것 같다.

클래스 프로젝팅도 있다. 흔히 보는 SELECT new com.full.qualified.name.Type(e.id, …) FROM Entity e …

## 세줄요약

N+1문제의 원인은 TwoPhaseLoader에 있다.

쿼리 메서드는 리턴 타입에 따라 실제 쿼리가 달라진다.

Loader와 TwoPhaseLoader는 데이터베이스와 인터랙션하는 인터페이스를 제공하고 있다.

### 어노테이션 프로세싱

그래, 매핑할 인터페이스를 만들면 N+1이 일어날 일이 없다는건 알겠다. 근데 그거 하자고 인터페이스 추가하고 또 필드에 맞춰 메서드 만들어야하는건 짜증나는데?

어노테이션 프로세서가 자동으로 매핑 인터페이스를 만들 수 있다. 이건 다음에.