---
published: true
layout: single
title: "디지털 논리회로 - 컴퓨터는 어떻게 덧셈을 계산할까?"
category: TIL
comments: true
---

## 릴레이
릴레이는 입력에 따라 내부 스위치를 바꾸어, 출력을 제어하는 장치다.
![릴레이](/../assets/step1-relay.png)

입력에 전류가 흐르면(1, True) 전자석이 작동한다.  
내부 스위치가 닫혔으므로 출력부분에서 1, True가 검출된다.

## AND 게이트
&& 연산과 같다.  
두개의 입력을 받는다. 둘 모두 true일 때만 출력이 true가 된다.
![and](/../assets/step1-and.png)

기호로 표시할 때 다음과 같이 표시한다.  
![and_symbol](/../assets/step1-and-symbol.png)

## OR 게이트
|| 연산과 같다.  
두개의 입력을 받는다. 하나 이상이 true이면 출력이 true가 된다.
![or](/../assets/step1-or.png)

기호로 표시할 때 다음과 같이 표시한다.  
![or_symbol](/../assets/step1-or-symbol.png)

## INVERTOR
! 연산과 같다.  
출력이 true이면 false로, false이면 true로 바꾼다.
![invertor](/../assets/step1-inverter.png)

기호로 표시할 때는 끝에 버블을 붙인다.  
![invertor_symbol](/../assets/step1-inverter-symbol.png)

## (참고)카르노 맵
불 대수를 표현하기 위한 표.  
![카르노맵](/../assets/K-map_minterms_A.svg.png)

## NAND 게이트
INVERTOR + AND 게이트.  

| NAND  | false | true  |
|-------|-------|-------|
| false | true  | true  |
| true  | true  | false |

- 구현
```cpp
bool nand(bool A, bool B){
    bool answer;
    answer = !(A && B);
    return answer
}
```

## NOR 게이트
INVERTOR + OR 게이트.

| NOR   | false | true  |
|-------|-------|-------|
| false | true  | false |
| true  | false | false |

- 구현
```cpp
bool nor(bool A, bool B){
    bool answer;
    answer = !(A || B);
    return answer;
}
```

## XOR 게이트
입력값 두개를 받아서, 서로 다를 때에만 true를 출력한다.

| NOR   | false | true  |
|-------|-------|-------|
| false | false | true  |
| true  | true  | false |

### 구현방법
- NAND 이용  
![xor-nand](/../assets/300px-XOR_from_NAND.svg.png)

- NOR 이용  
![xor-nor](/../assets/320px-XOR_from_NOR.svg.png)

- NAND, OR, AND 이용  
![xor-mixed](/../assets/254px_3gate_XOR.jpg)

- 구현
```cpp
bool xor(bool A, bool B){
    bool answer;
    answer = (!(A && B) && (A || B));
    return answer;
}
```
## 이진 덧셈기

### 반가산기
![반가산기](/../assets/step1-halfadder.png)  


| 합 | 0 | 1 |
|----|---|---|
| 0  | 0 | 1 |
| 1  | 1 | 0 |



| 자리올림 | 0 | 1 |
|----------|---|---|
| 0        | 0 | 0 |
| 1        | 0 | 1 |

- 구현
```cpp
bool *halfadder(bool A, bool B){
    bool answer[] = {0,0};
    answer[0] = xor(A,B);
    answer[1] = (A && B);
    
    return *answer;
}
```

### 전가산기
![전가산기](/../assets/step1-fulladder-symbol.png)

```cpp
bool *fulladder(bool A, bool B, bool carry){
    bool answer[] = {0,0};

    bool S = xor(A, B);
    bool C = A && B;

    answer[0] = xor(A, B);
    answer[1] = (B && carry) || carry;

    return answer;
}
```

## 가산기(바이트)
> 전가산기, 반가산기를 응용해서 바이트 덧셈기를 만들어본다.
> lsb(leas significant bit)는 배열의 첫번째 요소다. 1011 -> 1, 1, 0, 1

```cpp
bool *byteAdder(bool byteA[], bool byteB[]){
    bool *answer = new bool[8];
    bool *carry = new bool[9];
    bool *temp = new bool[2];

    carry[0] = false;
    for (int i = 0; i < 8; i++){
        temp = fulladder(byteA[i], byteB[i], carry[i]);
        carry[i+1] = temp[0];
        answer[i] = temp[1];
    }

    return answer;
}
```