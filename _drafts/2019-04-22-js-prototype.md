---
published: true
layout: single
title: "자바스크립트의 객체지향 프로그래밍 - 프로토타입"
category: TIL
comments: true
---

## 자바스크립트에서의 OOP
자바스크립트는 멀티패러다임 언어이다. 객체지향 패러다임으로도 사용할 수 있다. 그러나 자바스크립는는 특이하게 디자인되어 있다.

### 클래스
자바스크립트에서, 사실 클래스는 없다. **클래스처럼 동작하는** 객체가 있을 뿐이다.  
클래스는 특정한 객체를 정의하기 위해 존재한다. 객체가 만들어질 때에는 클래스의 생성자가 객체를 초기화한다. 자바스크립트에서 **클래스는 생성자와 같다.** 정확히 말하면 클래스 표현식은 생성자 표현식으로 바꿀 수 있다.
```js
class classTest {
    constructor(name){
        this.name = name;
    }
}

const constructorTest = function (name) {
    this.name = name;
}
// 다음 두 표현식은 같은 결과를 낸다.
const classObj = new classTest('andole');
const constructorObj = new constructorTest('andole');
```
자바스크립트에서, 함수가 `new` 키워드와 함께 호출되면 함수는 새로운 객체를 만들고 리턴한다.  
클래스가 `new`키워드와 함께 호출되면 `constructor`함수가 실행되어 객체를 만들고 리턴한다.  

## 프로토타입
모든 클래스`Object`에는 `prototype`이라는 속성이 있다. `prototype` 은 자신의 상위 객체를 가리킨다. 상위 객체 역시 자신의 `prototype`객체를 가리킨다.  `prototype`의 값이 `null`이 될때까지 반복된다.  
프로토타입을 