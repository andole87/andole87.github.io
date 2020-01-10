---
published: true
layout: single
title: "[Git] git - Head Operator"
category: ETC
comments: true
---

깃의 커밋은 SHA-1 40자로 표시된다. "8eab35865e303267890d6e19ae1887bce1a56806". 무슨 뜻인지 알겠는가? 안다면 할 수 없지만...    
커밋을 다룰 때, 대상 커밋을 지칭할 때가 많다. 체크섬을 외워서 사용하는 것은 반 인륜적이다. 좋은 방법 없나?  

HEAD Operator를 다룰 때가 되었다.  

## HEAD

커밋들은 반드시 부모가 있어야 한다. 최초 커밋은 HEAD 자신이 부모가 된다.  
하지만 실제로 커밋들을 연결시키는 것은 브랜치다. 브랜치는 양방향 연결 리스트로 커밋을 관리한다.  

> 깃이 내부적으로 관리하는 branch의 형태를 보고 싶다면 `.git/logs/refs/heads/브랜치명`파일을 열어보시라. 

많은 깃 설명서에 나온 것처럼, HEAD는 브랜치를 가리키는 포인터다. HEAD는 브랜치를 가리키고, 브랜치는 커밋을 가리킨다.   
즉, **HEAD는 현재 브랜치의 최신 커밋을 가리키고 있다.**   

따라서 HEAD 연산자를 사용하면 현재 커밋을 기준으로 탐색할 수 있다.  
그리고 다음에 나오는 내용 모두, `HEAD`대신 특정 `BRANCH` 이름으로도 동작한다. 왜냐, `HEAD`는 브랜치의 참조 포인터에 불과하기 때문이다.  

### HEAD^

현재 커밋의 이전 커밋을 가리킨다. 여기서 이전 커밋은 당연히 같은 브랜치의 이전 커밋이다.  
`HEAD^`표현은 `reset`할 때 많이 쓰인다.  
```bash
git reset [option] HEAD^^^
```

현재 커밋 기준으로 3개 이전으로 돌아간다.  

### HEAD~n

현재 커밋의 이전 n번째 커밋을 가리킨다. 많은 커밋을 돌아가야 한다면 `^`를 여러번 입력하기보다, `~n`을 사용하시라.  
```bash
git reset [option] HEAD~3
```

현재 커밋 기준으로 3개 이전으로 돌아간다.

### HEAD@{n}

여러 git command로 HEAD가 이동했을 때, command 전의 커밋을 가리킨다.  

`A - B - C - D`  

`D`커밋에서 `reset`명령으로 `A` 커밋으로 돌아간 상황에서, `HEAD@{1}`는 `D`를 가리킨다.  

`reset`을 잘못했을 때 주로 사용했다.  

```bash
git reset --hard HEAD^^^
```

엌! `reset`을 잘못했다. `HEAD^^`로 갔어야 했는데 `HEAD^^^`로 갔다. 난 망해따...  가 아니다.  
이 때 `HEAD@{n}`를 사용하면 `reset`이전으로 돌아갈 수 있다. (물론 가비지컬렉트 전일 때...)  

```bash
git reset --hard HEAD@{1}
```

`reset` 명령 전의 커밋으로 `reset`한다.

### ORIG_HEAD

`HEAD@{1}`의 Alias다. 깃 명령 하나 전의 커밋을 가리킨다.  

### FETCH_HEAD

`fetch` 명령 이후에, 타겟 브랜치는 `FETCH_HEAD`로 참조할 수 있다.  

```bash
git fetch origin BRANCH


git reset --hard origin/BRANCH
git reset --hard FETCH_HEAD
```

아래 두 명령은 똑같이 동작한다.


### BRANCH..BRANCH

두 브랜치에서 공통이 아닌 커밋을 가리킨다. 

```
[master]  B - C
        /  
    A 
        \
[some]    D - E
```

`master..some`은 E D,  
`some..master`는 C B이다.

![double dot](https://user-images.githubusercontent.com/40727649/63639465-3b2edb80-c6ce-11e9-8fa5-d199d2be2bf8.png)

### BRANCH...BRANCH

두 브랜치에서 공통을 제외한 모든 커밋을 가리킨다.

```
[master]  B - C
        /  
    A 
        \
[some]    D - E
```

`master...some`은 E D C B

![Triple dot](https://user-images.githubusercontent.com/40727649/63639464-39fdae80-c6ce-11e9-9121-76abbba228ca.png)
