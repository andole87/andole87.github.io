---
published: true
layout: single
title: "파이썬으로 정렬 알고리즘 구현 -1"
category: algorithm
comments: true
---
## 버블정렬 - bubble sort

정렬 방법 중에서 가장 직관적이고 이해하기 쉽다.  
시간 복잡도는 O(n2)이다. 성능이 좋은 방법은 아니다.  
어떻게 작동하냐면..  
![bubble sort](/../assets/BubbleSort_Avg_case.gif) 
> 출처: [알고리즘도감](https://algorithm.wiki/ko/app/)  

1. 배열을 순회하며 배열 끝에서부터, 반대 끝까지 두 요소를 선택한다.
2. 두 요소를 비교한다.
3. 만약 더 작은 요소가 뒤에 있었다면, 자리를 바꾼다.
4. 다음 요소들을 비교한다.
5. 루프 - 만약 배열의 끝에 도달하면 1번으로 돌아간다.

```python
def bubble_sort(array):
    for index in reversed(range(len(array))):
        for i in range(index):
            if array[i] > array[i+1]:
                x[i],x[i+1] = x[i+1],x[i]
```

이 방법은 이미 정렬되어 있을 경우, O(n)이다. 각 요소에 액세스하고 끝나기 때문. 
반대로 최악의 경우 O(n2)의 시간이 소요된다. 길이 n의 배열을 정렬하려면
(n-1)번 + (n-2)번 + (n-3)번 ... + 1 = (n * (n - 1) / 2)의 연산을 수행해야 한다.
코드상으로도 이중 반복문이므로 직관적으로 이해할 수 있다.  

![worst case](/../assets/BubbleSort_worst_case.gif)
> 출처: [알고리즘도감](algorithm.wiki/ko/app/)
