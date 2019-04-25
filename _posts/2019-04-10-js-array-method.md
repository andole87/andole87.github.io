---
published: true
layout: single
title: "js 배열 메서드"
category: TIL
comments: true
---

>자바스크립트는 멀티 패러다임 언어이다. prototype을 이용한 객체지향 프로그래밍도 가능하고 함수형 프로그래밍 역시 가능하다. 많은 배열 메서드 중에서 함수형 메서드를 다뤄본다.

### 참고: Array(배열)
배열은 같은 자료형을 메모리 공간에 연속 배치한 자료구조다. C언어에서 `int[n]`배열은 4바이트(int)씩, n개의 공간을 메모리에 준비한다. 총 `4 * n`만큼의 메모리공간이 필요하다. 배열의 index번째 요소에 접근하려면, 배열의 맨 처음을 기준으로 4바이트씩 offset을 적용한 메모리에 찾아가면 된다. 즉 `array[k]`에 접근하려면 array의 `첫번째 메모리 주소 + 4byte * k`로 가면 된다.  
이렇듯 배열을 다룰 때에는 index가 필수적으로 활용된다.

### forEach
js의 Array prototype에는 `forEach`라는 메서드가 존재한다. 배열의 모든 요소를 순회하며 콜백을 실행하는 메서드다. 
```js
Array.forEach( callback, [thisArg])
// 반환값은 undifined
```
callback은 **요소 값**, **현재 인덱스**, **순회중인 배열**을 인자로 받는다.  
for문과 비슷하게 쓰일 수 있다. for문과의 다른점은  
- 순회할 index를 선언하지 않아도 된다. 인덱스변수의 스코프를 신경쓰지 않아도 된다.
- 모든 요소에 callback을 실행할 것을 보장한다. for문에서 경계조건을 잘못 설정하는 실수를 걱정하지 않아도 된다.
- **도중에 중단할 수 없다**. for문은 break 등으로 반복을 제어할 수 있지만, forEach는 원칙적으로 도중에 중단할 수 없다. (에러를 발생시켜 멈추게 할 수 있긴 하나, 권장되지 않는다.)
- this 바인딩이 가능하다. 유연한 동작이 가능하다.  
 
```js
array = [1,2,3,4,5];
// arrow function
array.forEach( (element, index, arr) =>{
    arr[index] = element * 2;
})
// 반환값은 undifined.
console.log(array)
// [2,4,6,8,10]

// 기본 function
array.forEach( function (element, index, arr){
    this[index] = element * 2;
},array)
// 반환값은 undifined
console.log(array)
// [4,8,12,16,20]
```

기억해야 할 것은 반환값이 **undifined**인 것. 때문에 메서드 체이닝 중간에 쓰면 안된다.  

### map
map 메서드는 배열을 순회하며 callback을 실행한 **새로운 배열**을 리턴한다.  
`map(callback(current value, index, array),thisArg)`
```js
array = [1,2,3,4,5];
array.map( (element,index,arr) => element*2))
// 반환값은 [2,4,6,8,10]. 기존 array는 변화 없다.
console.log(array)
// [1,2,3,4,5]
// 기존 배열을 바꾸고 싶다면 this바인딩이나 배열인자를 이용할 수 있다.
array.map( (element,index,arr) => arr[index] = element*2) )
// [4,8,12,16,20]
console.log(array)
// [4,8,12,16,20]
```

새로운 배열을 리턴하기 때문에 메서드 체이닝이 가능하며 권장된다.

### filter
filter 메서드는 배열을 순회하며 callback에 맞는 값만 필터링한 **새로운 배열**을 리턴한다.
`filter(callback(current value, index, array),thisArg)`
callback을 통과한 요소들로 이루어진 새로운 배열을 리턴한다.
```js
array = [1,2,3,4,5]
array.filter( (element, index, arr) => element % 2 === 0)
// [2,4]
console.log(array)
// [1,2,3,4,5]
```

역시 새로운 배열을 리턴하기 때문에 메서드 체이닝이 가능하다.

### reduce
reduce 메서드는 배열을 순회하며 **새로운 형태의 결과**를 리턴한다.  
reduce는 '줄이다'는 영어단어다. 고차원의 형태를 쪼개어 저차원으로 반환하는 의미로 해석할 수 있다.  
예컨대 배열을 String으로 바꾼다던지, 2차원 배열을 1차원 배열로 바꾼다던지, 배열을 객체로 바꾼다던지.....등등.
`reduce( (accumulator, current value, index, arr), initialValue)`
callback의 `accumulator`에 집중해보자. `accumulator`는 누산기라는 뜻으로, 배열을 순회하면서 `accumulatror`에 콜백의 결과를 **누적**시킨다. 때문에 `initialValue`에 String을 넣으면 반환값이 String, 배열을 주면 배열로, 객체를 주면 객체가 리턴된다. `initialValue`를 생략하면 배열의 첫 요소와 같아진다.
```js
array = [1,2,3,4,5]
array.reduce( (accumulator, currentValue, index, arr) =>{
    return accumulator + currentValue;
},0)
// 15
array.reduce( (accumulator, currentValue, index, arr) =>{
    return [...accumulator, currentValue * 2];
},[])
// [2,4,6,8,10]
console.log(array)
// [1,2,3,4,5] 원본은 변하지 않음
```


### 조합
```js
array = [1,2,3,4,5]
array
    .filter(v => v % 2 === 0)
    .map((e) => e * 2)
    .reduce( (a,c,i) =>{
        return a += `${i}번째 값은 ${c} `;
    },"");
// 0번째 값은 4 1번째 값은 8
```
