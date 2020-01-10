---
published: true
layout: single
title: "[Java] Generic - PECS"
category: JAVA
comments: true
---

제네릭을 자주 썼었다. 특히 컬렉션에서. `List<T> list = new ArrayList<>()`...  
이펙티브 자바 item 28을 보고 몇가지 실험을 해 봤다.

## [item 28] Use bounded wildcards to increase API flexiblility
> 요약: API 유연성을 높이기 위해서 한정적 와일드카드를 사용하라.  
> When **P**roducer, use <? **e**xtends T>. When **C**onsumer, use <? **s**uper T>  
> 생산자일때는 extends, 소비자일때는 super

생산자, 소비자를 봤을 때 OS에서 말하는 그 생산자 소비자인줄 알았다. [생산자-소비자 문제](https://ko.wikipedia.org/wiki/생산자-소비자_문제)  
여기서 말하는 생산자 소비자는 조금 맥락이 다르다.

### 생산자

> 타입 T의 인스턴스를 제공하는 것.   
> List<T>를 순회하며 T를 가져다가 뭔가를 한다면, List<T>가 T의 생산자가 된다.

```java
public void produce(List<T> list) {
    for (T t : list) {
        t.something();
    }
}
```

### 소비자

> 타입 T의 인스턴스를 소비하는 것.  
> List<T>에 T를 넣는 것. List<T>가 T의 소비자가 된다.

```java
public void consume(List<T> list) {
    list.add(new T());
}
```



테스트를 위해 순서대로 계층구조를 갖는 A B C 클래스를 준비했다.

```java
class A {}; 
class B extends A {}; 
class C extends B {};
```



메서드 파라미터가 아닌 지역변수로 와일드카드를 써봤다.

```java
List<? super B> list = new ArrayList<>();
//        list.add(new A()); err
        list.add(new B());
        list.add(new C());
```

띠용? `super` 키워드가 붙었는데 왜때문에 A가 안들어가고 C가 들어가지? 


```java
List<? extends B> list = new ArrayList<>();
//        list.add(new A()); err
//        list.add(new B()); err
//        list.add(new C()); err

```

`extends`키워드가 있으면 아무것도 안들어간다?????????

이리저리 구글링해봤다. 거의 "Producer면 extends하고 Consumer면 super 해" 라는 내용이었다. **아 됐고, 대체 왜 그러는데?**

제네릭을 이해하려면 `type erasure`부터 시작해야 한다.

### type erasure

타입 소거라고 부를 수 있겠다. 제네릭 형인자는 컴파일 이후 사라진다.    
즉, 제네릭은 컴파일 타임에 `type-safety`를 보장하기 위한 기능이다. 제네릭이 런타임에 동작하는게 아니라는 말이다.

```java
// 컴파일 이전
List<Integer> list = new ArrayList<>();
list.add(1);

// 컴파일 이후
List list = new ArrayList();
list.add(1);
```

제네릭을 사용하는 이유는, **컴파일 타임에 type-safety를 보장하기 위함**이다. 컴파일타임, 런타임, type-safety키워드로 위 문제들을 이해해야 한다.  

`type-safety` 예시를 보자.

```java
// non-generic 컴파일 타임에 아무 에러도 없음!!
List list = new ArrayList();
list.add(1);
list.add("123");
list.add(new A());
```

그런데, 이 컬렉션은 결국 어디서 사용될 것이다. 컬렉션 내부 값을 모두 더하는 메서드를 만들었다고 해보자.

```java
// 아무 컴파일 에러 없음
public int sum(List list) {
    int sum = 0;
    for (int i = 0; i < list.size(); i++) {
        sum += list.get(i);
    }
}
```

위의 list를 sum()의 인자로 넣고 실행해보자. 어떻게 될까? 당연하게도 에러가 난다. 모든 요소를 `+` 연산자로 연산하려 했는데 Object는 `+` 연산을 지원하지 않는다. 

그렇다면 런타임 동작을 방어하기 위해서 모든 메서드 부분에 try-catch해줘야 하나? 타입 체크를 해야 하나? 제네릭은 이래서 나왔다. 복잡하게 타입 체크를 하지 않아도, type-safety를 보장한다. 오류의 여지가 있으면 컴파일 타임에 보고하도록 하는 것!  
즉, 제네릭을 사용하여 형인자 T를 **컴파일러가** 체크하는 것이다. 런타임에는 아무 상관 없다.

위 문제로 돌아가기 전에, 다음 코드를 보자.

```java
Animal cat = new Cat();
Cat cat = new Animal(); // 이건 안됨
```

컴파일러는 `cat`을 무슨 타입으로 이해하고 있을까? Animal? Cat? **Animal**이다. 

위 `<? super T>` 부분에서, C는 B의 서브타입인데 왜 들어가지?에 대한 힌트다.

```java
List<? super B> list = new ArrayList<>();
list.add(new B()); // B 타입이니까 당연하다.
list.add(new C()); // WTF???????????????????

// list.add(new C()) 를 컴파일러는 이렇게 이해한다.
B something = new C();
list.add(something); // 아하...
```

자, 그러면 왜 `extends`가 안 되는지도 알 수 있다. 

```java
List<? extends B> list = new ArrayList<>();
// list.add(new A()) err!
// list.add(new B()) err!
// list.add(new C()) err!
```

왜 `extends`는 안될까? 위계관계에 있는 A,B,C 외 **다른 모든 것을 넣어봐도 안 될** 것이다. `null`을 제외하고.

컴파일러는 위 행동을 **금지하는** 것이다. 컴파일 타임에 `type-safety`를 보장할 수 없으니까.  
`<? extends B>` list에 B의 서브타입을 `add`하는 것을 허용하고 컴파일 해준다고 생각해보자.  
이 list에는 무엇이 들어갈 수 있을까? 별별놈들이 다 들어올 수 있을 것이다. `상속 extends`에는 **제한이 없으니까**.   D extends B, E extends B ...  
이런 클래스들도 들어와 버리면 `non-generic`과 다를 바가 없으니, `type-safety`를 위해서 컬렉션에 무엇을 넣을 때는 `extends`가 허용되지 않는 것이다.


반대로, list에서 무언가를 꺼낼 때를 생각해보자.

```java
public void produce(List<? extends B> list) {
    B b = list.get(index); // 가능!!
    // 다르게 표현하면 
    B b = new C();
    
    //이게 되니까!!
    B b = new B의 모든 하위 타입 T(); 
}
```

`super`경우에는?

```java
public void produce(List<? super B> list) {
//    B b = list.get(index); // 에러!
    // 다르게 표현하면
    B b = new A(); // 될리가 없다
}
```
PECS에 대해 헷갈릴 일이 없을 것 같다. 거의 이틀 내내 매달렸으니까.  
컬렉션 소스코드도 좀 뒤적거리고 컴파일러에 대해서도 조금은.. 알게 되었다.

~~그냥 Get하면 extends 쓰시고요, Put하면 super 쓰세요.~~

### 덧

그럼, B의 하위 타입을 담을 컬렉션은 만들 수 없나요? 할 수 있다. 타입 T를 래핑한 클래스를 만들면 된다.

```java
class Thing <T extends B> {}
List<Thing> things = new ArrayList<>();
// things.add(new Thing<>(new A())); error! 형인자 T는 B의 하위 클래스여야 함
things.add(new Thing<>(new B()));
things.add(new Thing<>(new C()));

```
