---
published: true
layout: single
title: "[Git] git - 초급"
category: CS
comments: true
---

## Overview
git은 DVCS, Distributed Version Control System 이다. 분산 버전관리 시스템으로 이해할 수 있다.  
왜 git을 쓸까? 세이브포인트를 마련하는게 첫번째, 뛰어난 브랜치 기능으로 여러 사람과 프로젝트를 잘 진행할 수 있는 점이 두번째다.  
git의 방대한 내용을 다 다룰수 있을지 두렵지만, 일단 남겨본다.

## git의 작동방식
git은 커밋을 단위로 스냅샷을 관리한다. 이를 위해 git은 세가지 영역을 준비하고 있다.  
working tree, stage area, commit이 그것이다.  


![깃 구성](/../assets/areas.png)
>출처: 프로깃
- working tree  
실제 작업 공간이다. 파일을 새로 만들거나, 소스를 변경한 것은 working tree에 있다.
- stage area  
커밋을 준비하는 공간이다. working tree의 작업 내용을 staging 해야, 변경점을 커밋할 수 있다.  
- commit  
stage area의 내용을 binary로 보관한다. 커밋은 반드시 부모 커밋이 있어야 하며(아래 서술), 각 커밋은 SHA-1 26자를 부여받는다.

## 커밋에 대해
커밋은 stage area에 있는 내용을 트리객체와 함께 저장된 것을 말한다. 부모 커밋이 필요한 이유는, **각 커밋은 해당 시점의 파일 내용 전체를 보관하지 않기 때문이다.** 변경사항을 stage area에 올리면 부모 커밋을 기준으로, 변경사항을 **append**한다. 커밋을 수행하면 부모 커밋에 대하여 변경된 사항을 해당 커밋에 저장해 놓는 것이다.  
**바로 이 점 때문에, git은 분산 관리가 가능하고, 마법같은 브랜칭이 가능하며, 쉬운 merge와 빠른 속도가 가능한 것이다.**

## 커밋하기 - add, commit, push
git의 cli 명령은 git으로 시작한다.
```bash
$ git [command] [option]
```  
- 변경사항을 stage area에 올리기  
```bash
$ git add [option] [file]
```  
`git add .` 명령은 모든 변경사항을 stage area에 올리는 것이다.  
- stage area의 내용을 커밋하기  
```bash
$ git commit [option]
```
커밋을 수행한다. 커밋은 커밋 메시지가 작성되면 진행되고, 메시지가 정상적으로 작성되지 않으면 abort된다.  
`git commit -m "message"` : message라는 커밋 메시지를 가진 커밋을 즉시 생성한다.  
`git commit --amend` : 직전 커밋을 불러오고, 직전 커밋을 덮어쓴다.  
- 원격 저장소에 커밋 올리기  
`git push [remote] [branch]` : [remote] 저장소의 [branch] 브랜치에 저장(푸시)한다.  
`git push [remote] +[branch]` : 강제로 푸시한다. 리모트 저장소의 기존 커밋은 사라지고, 로컬 저장소의 커밋으로 덮어쓴다.  

## 브랜치 - 분기 나누기
git의 브랜치는 git에서 매우 큰 부분을 차지한다. 브랜치가 다른 VCS와의 가장 큰 경쟁우위다.  
브랜치는 사실 HEAD가 움직일 양방향 링크드리스트다. HEAD의 경로를 갖고 있으므로 브랜치간 이동이 간편하다. 
> 여기서 알 수 있는 사실 : `git checkout [commit]`을 수행하면 [commit]이 브랜치에서 떨어져나온다.(detached commit) 다시 브랜치에 넣어주지 않으면 해당 commit은 미아가 된다. (링크드 리스트에서 떨어져나간다) 이후 가비지컬렉터가 미아 커밋을 수집해 버리면 커밋이 사라질 위험이 있다. 

브랜치로 순식간에 특정 시점으로 이동할 수 있고, 안전하게 이력을 관리할 수 있으며, 부담없이 테스트를 진행할 수 있다.  
적절한 브랜칭 전략은 프로젝트를 더 쉽고 재미있게 만든다.  

- 현재 커밋(HEAD)를 기준으로 브랜치 만들기  
`git branch [branchname]`
- 특정 브랜치로 이동하기  
`git checkout [branchname]`
- 현재 커밋을 기준으로 브랜치 만들고 이동하기  
`git checkout -b [branchname]`
- 브랜치 삭제  
`git branch -d [branchname]`
- 브랜치 상태 보기  
`git branch`
- 브랜치를 특정 커밋을 가리키게 하기  
`git branch -f [commit]`  
rebase나 히스토리 정리할때 자주 쓰인다. 

## Merge - 브랜치 합치기
각 이슈에 맞게 브랜치를 나누어 진행하던 프로젝트가 완성되었다. 완성본을 위해서는 브랜치를 합쳐야 한다. Merge로 브랜치들을 합칠 수 있다.  
Merge는 두가지 종류가 있다.  
- fast-forward
![fast](/../assets/basic-branching-5.png)


fast-forward는 앞으로 가기라는 뜻이다. 단순히 master의 HEAD를 앞으로 옮기기만 한다.  
충돌이 일어날 일이 없고 안전한 방식이다.

- 3way-merge
![3way](/../assets/basic-merging-2.png)

각 브랜치가 각자 진행사항이 있다. 이 때 같은 요소를 두 브랜치에서 변경하였다면 충돌(conflict)가 발생한다.  
충돌이 발생하면 수동으로 해결하기, 기본 브랜치로 해결하기, 머징 브랜치로 해결하기 3가지 방법이 있다.
- 수동  
```
<=== master
something code
===========
another code
===> merging branch
```
마스터와 브랜치 사이의 내용을 선택하면 된다. 기호들을 모두 지우고 남길 코드만 선택해 남겨두면 된다.  

## 버전 관리하기 reset, rebase, cherrypick
버전관리는 커밋을 중심으로 수행된다. 커밋이 존재한다면 거의 모든 짓(?)을 할 수 있다.  
- reset 이전 커밋으로 돌아가기  
```bash
git reset --[soft, mixed, hard] HEAD[^][~n]
```
옵션 중 soft는 HEAD만, mixed는 stage area까지, hard는 working tree까지 선택 시점으로 돌아간다.  
HEAD는 커밋을 가리키는 참조변수다. ^ 연산자는 직전을 의미한다. HEAD^^^는 3 커밋 전을 가리킨다.  
HEAD~n은 n번 전의 커밋으로 돌아간다.

- rebase 히스토리 정리하기  
rebase는 merge와 유사하지만, 커밋 객체들을 다시 생성하는 점에서 다르다. 커밋 객체를 다시 생성하기 때문에, merge대상의 커밋들은 사라지지만 히스토리가 선형으로 예쁘게 정리된다.
```bash
git rebase [option] [tobranch]
```
-i 옵션으로 인터랙티브한 리베이스가 가능하다.

- cherrypick 특정 커밋 가져오기  
다른 브랜치 내의 특정 커밋을 반영하고 싶을 때, cherry-pick을 사용한다.
```bash
git cherry-pick [commit]
```