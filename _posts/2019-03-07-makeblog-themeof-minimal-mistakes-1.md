---
published: true
layout: single
title: "깃허브 블로그 만들기: minimal-mistakes 테마-1"
category: ETC
comments: true
---

깃허브 블로그를 만들어볼까 했다. 공짜 호스팅에다 *.github.io url, git으로 관리할 수 있으니까. 그동안 구경해둔 블로그들은 있었으므로, 원하는 형태는 있었다.

내가 원하던 포맷은
- 카테고리가 있을 것
- bio같은 것보다, 콘텐츠 내용에 적합한 레이아웃일 것
- 내부 검색이 가능할 것


적절해 보이는게 minimal-mistakes 테마였다. [공식사이트](https://github.com/mmistakes/minimal-mistakes)

구글에 minimal-mistakes 검색하면 많은 블로그에 정보글이 나온다.  
아쉬웠던 점은 아예 아무것도 모르는 사람은 쓰여진 대로만 따라해야 한다. 따라하고 나면 설명한 것과 똑같은 결과가 나온다. 스스로가 원하는 것은 어디에서 어떻게 만져야 할지 모른다는 것이다.~~나만 모를 수도.~~  
내 수준에 맞게(초보) 설명을 남기고, 다음 포스트에 이어서 과정을 남겨본다.

--------
<br>
  
깃허브 페이지는 **jekyll**(*지킬이라고 읽는다. 지킬박사와 하이드의 그 지킬 맞다*)을 이용해서 서비스된다. jekyll은 웹 프레임워크의 일종이다. html을 동적으로 생성하고 스타일시트, 레이아웃 등을 쉽게 관리하기 위한 도구다.

따라서 깃허브에 사이트를 생성, 운영하려면 **jekyll**의 체계에 맞게 내용을 구성해야 한다.

# jekyll
jekyll은 루비 기반 프레임워크다. [jekyll 공식사이트](https://jekyllrb-ko.github.io/) 공식페이지에 가보면, 친절히 한글로 설치부터 운영까지 알려준다. 
내가 이해하기로 jekyll은 다음과 같은 특징이 있다.
- 정적 사이트 생성기
- 데이터베이스 필요 없음
- 마크다운 문법 지원

말로 풀어 본다.  
사이트 구성에 노력할 필요 없습니다! 모듈로 관리되는 테마들을 사용해 보세요!   
복잡하게 데이터베이스 연동, 소스관리를 하지 않으셔도 됩니다!  
당신은 손쉬운 마크다운으로 콘텐츠에만 집중하면 근사한 웹 페이지가 만들어져요!

## 구조
jekyll의 디렉토리 구조는 다음과 같다. 공식사이트의 내용에 나름대로 주석을 추가했다.
```ruby
.
├── _config.yml ## 사이트 설정. 사이트 url, 플러그인, 테마 등을 설정한다.
├── _data ## 사이트 공통 데이터. 다국어, 글로벌 메뉴 등이 있다.
|   └── members.yml
├── _drafts ## 퍼블리시 하기 전 포맷들이다.
|   ├── begin-with-the-crazy-ideas.md
|   └── on-simplicity-in-technology.md
├── _includes ## 메뉴, 사이드바.. 와 같은 사이트 요소들.
|   ├── footer.html
|   └── header.html
├── _layouts ## 레이아웃 정보. _includes의 내용들을 조합한다.
|   ├── default.html
|   └── post.html
├── _posts ## 포스팅들. 마크다운으로 저장하면 웹페이지로 생성한다.
|   ├── 2007-10-29-why-every-programmer-should-play-nethack.md
|   └── 2009-04-26-barcamp-boston-4-roundup.md
├── _sass ## 사이트 스타일시트. scss로 관리된다.
|   ├── _base.scss
|   └── _layout.scss
├── _site ## 실제 웹 페이지에 게시되는 정보들.
├── .jekyll-metadata
└── index.html
```

이 디렉토리 구조를 보면, 원하는 것을 위해 어디서 무엇을 찾아야 할지 알 수 있다.

- 사이트 제목을 바꾸려면 _config.yml에서 title을 뒤진다.
- 사이트 테마를 바꾸려면 _config.yml에서 theme 관련을 뒤진다.
- 사이드바를 없애려면 _layout폴더 아래 default나, _config.yml에서 설정된 테마 파일을 뒤진다.
- 사이드바를 수정하려면 _include폴더 아래 sidebar를 뒤진다.
- 사이트의 margin, padding을 수정하려면 _sass폴더 아래를 뒤진다.
- 새로운 글을 쓰려면 _post 폴더 아래에 새로운 markdown 파일을 생성한다.
- ...

또하나 눈여겨볼 것은, _site 폴더다. **_site폴더에 있는 것이 최종 결과물이다**. 나머지는 _site를 만들기 위한 자료에 불과하다.


다시 말하면 jekyll은 시작될 때, _site를 제외한 폴더에서 사이트 구성에 필요한 자료를 가져온다. _include에서 사이트 모듈 정보를 불러오고, _layout에서 모듈을 조합하고, _post에서 사이트 본문을 불러온다. ~~ 마지막으로 _site폴더에 **진짜 사이트 내용**, html코드를 생성한다. 이후에 사이트에 요청이 들어올 때, _site폴더에서 내용을 찾아 응답하는 방식이다.  

minimal-mistakes 테마 설치와 적용, 커스텀은 다음 포스팅에서 이어 진행한다.