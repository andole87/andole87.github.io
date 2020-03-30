---
title: "MongoDB 공간 인덱스"
category: "ETC"
---

## 공간 검색

몽고DB의 공간 인덱스는 크게 두가지 종류가 있다. 유클리드 평면과 비유클리드 평면인 구면 두가지다. 평평한 유클리드 평면은 2d, 구면은 2dsphere를 사용한다. 지구의 위경도 좌표를 이용하는 2dsphere는 `geohash`, `s2geometry` 방법으로 인덱스를 구성할 수 있다. 두 방법 모두 좌표쌍을 단일 값으로 치환해 인덱스를 구성한다.  
좌표쌍은 이차원 값이다. 순서쌍을 특정 알고리즘으로 일차원 스칼라 값으로 변경하는 것을 말한다. (x좌표, y좌표) => (1234)

MySQL, Oracle 역시 공간 검색을 지원한다.

- MySQL: R-Tree
- Oracle: Quad-Tree, R-Tree
- MongoDB: geohash, s2geometry

R-Tree는 좌표쌍 영역을 인덱스로 구성한다. 즉 이차원인 배열의 두 요소를 일차원으로 변경하지 않고 그대로 사용한다. 때문에 일반적인 B-Tree와 호환할 수 없다. 즉 일반적인 숫자, 문자 인덱스와 공간인덱스를 합성할 수 없다.

몽고DB의 공간 인덱스는 좌표쌍을 일차원 값으로 변환하고 구성한다. 일차원 값에 대한 B-Tree를 구성하므로 일반적인 인덱스와 같다. 단지 좌표를 해시하는 알고리즘이 추가될 뿐이다. 때문에 일반적인 인덱스와 호환할 수 있으며, 일반적인 인덱스와 함께 복합 인덱스를 구성할 수 있다.

예) 음식 타입이 한식, 중식, 일식인 반경 1KM내 음식점 위치 찾기.

→ 몽고DB는 하나의 복합 인덱스로 처리 가능하다. R-Tree구조는 음식 타입에 대한 인덱스와 공간 인덱스를 두번 스캔하거나 인덱스 하나만 사용하고 필터링을 수행해야 한다.

## geohash

geohash는 좌표를 사분면으로 재귀 분할한 결과를 해시한다.

```
01 | 11
-------
00 | 10
```

각 비트들은 각 사분면을 나타낸다. 추가적인 정밀도를 제공하려면 각 사분면에 재귀적 반복을 수행한다. 사분면을 다시 사분면으로 나누고 2 비트 값의 뒤에 2 비트 값을 추가한다. `11` 사분면을 예로 들면 `1101`, `1111`, `1110`, `1100` 의 새로운 사분면을 할당한다.

최대 분할 횟수는 60번 이다. 이론적으로는 64번 분할할 수 있으나 분할 결과로 나온 비트를 BASE-32 (5비트씩 한 문자)로 인코딩하므로 60비트까지 사용할 수 있다.

좌표 위치가 근접하면 geohash 값 자체도 근접한다. 비트를 분할할 때 부모 영역의 뒤에 추가 정밀도를 붙이기 때문이다. 같은 부모 영역을 공유하면 앞의 비트는 모두 같게 된다. 서울 주요 구역 8개 geohash 값이 모두 `wydm`으로 시작한다.  
이러한 특성 때문에 문자열 패턴 일치 검색으로 근접 좌표를 쉽게 찾을 수 있다. (예를들면 LIKE 검색). 그리고 BASE-32 인코딩은 디코드도 쉽다.  이것은 큰 장점이며 위치 검색에서 geohash를 사용하게 된 주요 이유다.

## s2geometry

우버, 오로라, 구글맵에도 사용된다. Hilbert Curve라는 이론을 근간으로 한다.

수학자 힐베르트에 따르면 평면은 하나의 직선으로 연결, 표현할 수 있다. 그어지는 선을 따라서 영역에 번호를 매기면 영역이 인접한 경우에는 영역의 번호도 근접한다.

![hilbert curve](https://user-images.githubusercontent.com/40727649/77847900-68f77800-71fb-11ea-9297-c2c399ffb52d.gif)

구면에 대해서는 지구를 감싸는 가상의 육면체를 만들고, 구체의 1/6을 육면체의 각 면에 펼쳐서 사상한다.

![s2geometry](https://user-images.githubusercontent.com/40727649/77846437-013c2f80-71f1-11ea-893f-625225c5a233.png)

s2 geometry 방식에서도 단위 구역은 고유한 값을 가진다. 이 값을 셀 아이디라고 한다. 전체 64비트를 사용하며 인코딩을 거치지 않고 정수 타입 그대로 사용한다.

셀 아이디는 크게 세가지 정보를 포함한다.

- face: 구면을 포장하는 가상 육면체 면의 고유 아이디. 6개 값만 존재.
- cell-id: 하나의 구역을 4개로 분할하는데, 4개 구역 아이디를 2비트씩 사용. 60비트이므로 30번 분할가능
- end-marker: 마지막으로부터 최초로 0이 아닌 비트가 발견되는 위치. 분할 반복수를 표현

```
face              cell-id                        marker
    
001       1010110101011101110111100101010          1           000
```

s2 geometry는 geohash와 비슷하게 인접한 영역을 쉽게 찾을 수 있다. 더해서 비트형식을 그대로 사용하기 때문에 더 빠른 비트 연산으로 주변을 찾을 수 있다. 내부적으로 반경이나 사각형 내 위치 정보를 검색하려면 해당 영역을 커버하는 셀들을 조사해야 한다. 이 셀들은 일반적으로 1개 이상이고 서로 다른 레벨의 셀(분할 횟수가 다른 셀, 크기가 다름)이 조합되는 경우가 많다.

![cell](https://user-images.githubusercontent.com/40727649/77846496-790a5a00-71f1-11ea-9cc8-0cb78e13bb28.jpeg)

s2 geometry 쿼리 성능에 영향을 미치는 2가지 중요한 요소는 다음과 같다.

- 검색 대상 셀의 최소 레벨과 최대 레벨
- 검색 대상 셀의 개수

A: 최소레벨 5, 최대 레벨 20, 최대 셀 개수 10 - 셀 6개 리턴

B: 최소레벨 1, 최대 레벨 20, 최대 셀 개수 100 - 셀 67개 리턴

A: 비트리에서 인덱스 스캔 지점을 찾는 작업이 6번. 대신에 불필요한 데이터를 읽을 가능성 높음

B: 67개를 찾아야함. 대신 자세한 영역이므로 불필요 데이터 읽을 가능성 낮음

## 공간 검색 연산자

몽고DB에서 위치를 검색할 때 여러 연산자를 지원한다. 크게 `near` 계열, `within` 계열 두가지로 나눌 수 있다.  
`near`는 특정 지점을 기준으로 가까운 좌표들을 찾는다.  
`within`은 특정 영역을 기준으로 포함하는 좌표들을 찾는다.  

### near

연산에 대한 인자로 점을 받아 가까운 순으로 주변 위치를 반환한다.  
실행하려면 **반드시 geometry 인덱스가 필요하다.**  

인덱스로 구성된 직선([s2geometry 참고](#s2geometry))에서 인자로 받은 좌표로 점 P를 찾는다. 
이후 주변 좌표에 대해 내부적으로 `DistanceMeta`로 표현되는 거리를 계산한다. 점P를 기준으로 양쪽의 추가 스캔이 필요하며, 쿼리의 결과는 가까운 순 정렬 상태를 유지하게 된다.  

위와 같은 동작 때문에 `near` 연산자가 포함된 쿼리는 다른 인덱스를 사용할 수 있더라도 가장 먼저 공간 인덱스를 사용하게 된다.  
다른 인덱스로 도큐먼트를 필터링하고 나면 공간 인덱스를 사용할 수 없으므로 거리를 계산할 수 없기 때문이다. 어그리게이션에서도 `near` 스테이지는 가장 첫번째 스테이지이여야 한다.

보통 가까운 순으로 정렬이 필요하지 않다면 `near`는 `within`보다 느리다. 단순 일정 영역에 포함되는 좌표를 검색해야 한다면 `near`는 좋은 옵션이 아니다.  

### within

연산에 대한 인자로 영역(Polygon, Circle...)을 받아 포함되는 위치를 반환한다.  
실행할 때 **geometry 인덱스가 없어도 된다.**  

`near`는 도큐먼트가 가진 값이 아닌 인덱스의 값으로 평가를 수행한다면, `within`은 도큐먼트가 가진 값으로 평가를 수행하기 때문이다. 물론 인덱스가 있으면 쿼리 성능이 향상된다.

인덱스로 구성된 직선에서 인자로 받은 영역 R을 구성한다. R에 해당하는 점 P의 집합을 찾아 결과로 반환한다. `near`가 거리를 계산하고 정렬하는 등의 연산이 내부적으로 진행되는 것과는 달리, 마치 B-tree 범위 스캔하듯이 수행된다. 인덱스만으로도 쿼리를 수행할 수 있다. 정렬과 같은 옵션이 필요 없다면 `near`보다 훨씬 빠르다. 단순 범위에 포함되는지 여부만 boolean으로 판단하기 때문이다.

여러 조건으로 쿼리한다면, 옵티마이저가 효율이 가장 큰 인덱스로 먼저 필터링하고 공간 검색을 수행하도록 실행 계획을 수립할 수 있다. `near`는 반드시 공간 인덱스부터 사용하는 것과는 다른 점이다.  

1만개 도큐먼트를 대상으로 두 가지 조건(pk, 공간)의 쿼리에 대해서 `near`는 1600ms, `within`은 20ms정도 걸리는 것으로 확인했다.

## Aggregate with Spatial Data

어그리게이션에서도 위치 관련한 스테이지들을 지원한다. 스테이지 이름은 `$geoNear`다.  
`$geoNear` 는 반드시 어그리게이션 스테이지 중에서 가장 첫번째 스테이지여야 한다. `near`에서 밝힌 바와 같이 공간 인덱스가 있어야만 `near` 연산을 수행할 수 있기 때문이다.  

샤드 환경에서 `$geoNear`와 같은 공간 쿼리, 어그리게이션을 수행하려면 반드시 복합 키 구성을 고려해야 한다. 샤드 환경에서 어그리게이션 스테이지는 개별 샤드가 각각 처리하거나, 특정 샤드만 처리할 수 있는 것으로 나뉜다.  

개별 샤드가 처리할 수 있는 것은 필터링, 변환과 같은 것들이다. `$match`, `$project`와 같은 스테이지는 샤드 스스로가 가진 데이터를 대상으로 완전히 수행할 수 있으므로 개별적으로 처리할 수 있다.  

조인, 그룹, 정렬과 같은 스테이지는 특정 샤드에서만 수행할 수 있다. 곰곰 생각해보면 당연하다. 정렬하려면 모든 도큐먼트가 필요한데, 각 샤드는 전체 도큐먼트를 모르기 때문이다. (샤드키 기준은 제외)

샤드의 목적 중 하나인 부하 분산을 위해서는 최대한 개별 샤드가 각각 처리할 수 있도록 해야 한다. 신중한 샤드 키 선택, 성능을 고려한 스테이지 구성이 필요하다. (간단한 최적화는 옵티마이저가 자동으로 수행할 수 있다)

돌아와서, `$geoNear`는 동작의 특성 때문에 맨 처음에 수행되어야 한다. 만약 어그리게이션 스테이지가 여러 단계로 도큐먼트를 조작하고 있다면 부하 분산과 성능상으로 많은 손해를 볼 수 있다. 최초 스테이지에서 많은 도큐먼트를 필터링해서, 어그리게이션 대상 도큐먼트의 숫자를 줄일수록 성능상 유리하다. 하지만 `$geoNear` 이후 다른 필터링 연산이 많다면 인덱스를 사용할 수 없거나 개별 샤드에서 수행하기 어려울 수도 있기 때문이다.  

위치와 관련된 복잡한 어그리게이션이 필요하다면, `$match` 스테이지에서 `within`연산자로 최대한 필터링하자. 아니면 복합 키 인덱스로 `$geoNear`에서 최대한 걸러내자. 이렇게 개별 샤드에서 연산 대상 도큐먼트를 최대한 줄이고, 경우에 따라 `$limit`, `$sort` 스테이지를 조합해서 구성하는게 부하 분산, 성능에 유리하다. 