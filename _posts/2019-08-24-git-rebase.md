---
published: true
layout: single
title: "[Git] git - rebase"
category: ETC
comments: true
---

깃을 처음 사용할 때는 거의 `add` `push`만 사용했다. 깃을 조금 알게 되면서 브랜치를 나누게 되었다. 브랜치를 나누면서 이력관리를 어떻게 해야 할까 고민하게 되었다.  
rebase를 이용해서 브랜치를 합치고 옮기고 바꿔보자.

## merge? rebase? 

`merge`와 `rebase` 모두 두 브랜치를 합치는 것이다. 뭐가 다를까?  

**`merge`는 두 브랜치를 동시에 가리키는 `merge commit`을 새로 만드는 것이다.**  
`merge commit`을 새로 만들기 때문에, 통합될 **두 브랜치에는 아무 영향이 없다**

**`rebase`는 타겟 브랜치로 현재 브랜치가 이동하는 것이다.**  
`rebase`는 현재 브랜치의 커밋을 **새로 만들기 때문에** `rebase`전과 후의 커밋들은 서로 다르다.

`merge`는 안전성이 높고 이력(커밋)이 순차적으로 나오지 않는다.  
`rebase`는 안전성이 떨어지나 이력(커밋)을 의도에 맞게 구성할 수 있다.

`rebase`의 기존 커밋을 변경하는 특성 때문에 많은 프로젝트(특히 외국)은 merge를 선호하기도 한다.  
그렇지만 `rebase`의 장점 역시 뛰어나기 때문에, 상황에 맞게 쓰면 되겠다.  

## 기본 사용

```bash
// 현재 from 브랜치
git rebase [합칠_브랜치_명]
```

`merge`와 달리 **어디서 어디로**가 헷갈린다.  

`merge`는 메인 브랜치에서 서브 브랜치를 **당겨오는** 느낌,  
`rebase`는 서브 브랜치가 메인 브랜치로 **이동하는** 느낌이다.


## onto 옵션

```bash
git rebase --onto [타겟_브랜치] [옮겨갈_커밋범위의_하나_이전_커밋] [옮겨갈_커밋범위의_마지막_커밋]
```

`--onto` 옵션은 옮겨갈 커밋을 정할 수 있다.

master : A - S  
feat   : A - Z - C - D  

공통 커밋은 `A`이므로 `feat`브랜치의 `Z - C - D`가 옮겨갈 것이다. 그러나 `C - D`만 옮기고 싶다면? `--onto`를 사용하자.

```bash
git rebase --onto master Z D
```
feat 브랜치의 C부터 D까지가 master로 옮겨간다.

`Head Operator`를 사용하면 좀더 편하게 사용할 수 있다.

## exec 옵션

`--exec`는 `rebase`커밋을 만들 때마다 인자로 전달한 커맨드를 실행한다.  

```bash
git rebase --exec [타겟_브랜치] [실행할_커맨드]
```

`rebase`는 옮겨갈 모든 커밋을 새로 만든다. `master`이후로 `feat` 브랜치에서 100개 커밋을 만들었다면 100개 커밋을 새로 만드는 것이다.  
이 옵션은 커밋의 작성자를 바꾸거나 시간을 바꾸는데 유용할 수 있다.

- 커밋 작성자 바꾸기

```bash
git rebase --exec master "git commit --amend --author=\"Linus Torvalds <Linus@zzang.zzang.man>\""
```
브랜치의 모든 커밋이 `master`로 이동하면서, 커밋 작성자의 이름을 "Linus Torvalds", 이메일을 "Linus@zzang.zzang.man"으로 바꾼다.



깃에 관해 말하고 싶은건 많은데... 글로 표현하기 힘들다. 그림이 필요하다...