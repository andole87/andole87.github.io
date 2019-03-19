---
published: true
layout: single
title: "깃허브 블로그 만들기: minimal-mistakes 테마-2"
category: WEB
comments: true
---
minimal-mitakes 테마를 적용하고 설정을 변경하거나 일부 커스텀을 진행했다.

## minimal-mistakes 테마  

[공식사이트](https://mmistakes.github.io/minimal-mistakes/)
기본 모듈들이 잘 갖춰져 있고, 카테고리를 쉽게 나눌 수 있다. 

디렉토리 구조는 다음과 같다. 주요한 것만 남긴다.
```python
-.
|-_data         ## 아래에 navigation.yml에서 메뉴를 설정할 수 있다.
|-_includes     ## analytics, comment, 소셜, SEO등 기능적 모듈들이 담겨있다.
|-_layouts      ## home.html, posts.html등 템플릿들이 담겨있다.
|-_posts        ## 포스팅하는곳. "yyyy-mm-dd-titlename.md"형식으로 저장한다.
|-_sass         ## scss파일들이 있다.
|-_site         ## 지킬이 빌드될때 이 폴더 아래에 html파일을 생성한다.
|-.sass-cache   ## 컴파일된 scss파일들이 캐시 형식으로 담겨있다.
|-assets        ## 이미지, 동영상 등
|-_config.yml   ## 사이트 설정 파일
|-banner.js     ## 배너 인터액션에 관한 js
|-index.html    ## 메인 페이지
```

터미널에서 프로젝트 폴더로 이동한 다음, jekyll serve 또는 bundle exec jekyll serve를 명령하면 로컬환경에서 블로그를 볼 수 있다.
```bash
$>jekyll serve
$>bundle exec jekyll serve
```
둘의 차이점은 가상환경을 이용하지 않느냐 이용하느냐다. 공식사이트에서는 bundler를 이용한 프로젝트 환경을 추천하고 있다.  

## 사이트 설정하기  
프로젝트 루트에서 _config.yml을 열면 많은 항목들이 나온다. 각 항목에서 원하는 부분을 설정해주면 된다.
나는 기본 프로필, 기본url, 댓글시스템, 애널리틱스, SEO를 설정했다.
영어로 친절한 설명이 있다. 찬찬히 읽어보면서 나에게 맞게 바꿨다.

```ruby
# Thme Settings
minimal_mistakes_skin   : "mint" # minimal-mistakes 테마에는 여러 하위 테마가 있다. air, aqua, mint ... 
                                # 사이트 전체 색감을 변경할 수 있다. 나는 민트를 선택했다.

# Site Settings
locale                  : "ko-KR"
title                   : # 사이트 제목
name                    : # 사이트 소유자 이름
url                     : # 사이트 홈 url. 깃헙기준 "사용자이름.github.io"
comments:
  provider              : "disqus" # 주석에 여러개 지원이 가능하다고 씌여 있다.
  disqus:
    shortname           : # disqus에서 가입후 절차를 밟으면 shortname을 부여해준다.
                        : # 주석에는 full-url로 되어있지만 shortname만 넣으면 된다.
                        : # disqus에서 jekyll에 대해 기능을 추가한 것 같다..

# Analytics
analytics:
  provider              : "google" # 기본값은 false. 구글 애널리틱스를 사용할 것이므로 "google"을 입력.
  google                :
    tracking_id         : # 애널리틱스에서 부여받은 코드를 입력 "코드"

# Defaults
defaults:
  - scope:
  values:
    author_profile      : false # 기본값은 true. 프로필 노출이 싫어 껐다.
```

## 사이트 너비 조정
기본 테마는 two column 형식이다. 프로필이 좌측 작게 있고 본문이 오른쪽 넓게 들어간다. 나는 프로필을 꺼두었으니, 본문영역을 더 넓히고 싶었다.
_sass 폴더의 page.scss 파일을 연다. 전체 경로는 [/_sass/minimal-mistakes/_page.scss]

```sass
@include breakpoint($large) {
    float: right;
    width: calc(100% - #{$right-sidebar-width-narrow});
    padding-right: $right-sidebar-width-narrow;         //기존

    padding-right: calc($right-sidebar-width-narrow);   //변경
  }

@include breakpoint($x-large) {
    width: calc(100% - #{$right-sidebar-width});        //기존
    padding-right: $right-sidebar-width;                //기존

    width: calc(100% - #{$right-sidebar-width}+100px);  //변경
    padding-right: $right-sidebar-width - 100px;        //변경
````
100px씩 왼쪽, 오른쪽으로 넓혔다.

## 민트 테마 커스텀
위 _config.yml에서 선택한 mint테마는 링크 버튼만 민트색으로 바꿔준다. 포스트 제목도 민트색으로 바꾸고 싶었다.
_sass폴더 아래 mint.scss를 열고 다음 내용을 추가했다. 전체경로는 [_sass/minimal-mistakes/skins/_mint.scss]이다.
```sass
#page-title{
  color: $link-color;
}
```

## 포스팅
_posts폴더 아래에 'yyyy-mm-dd-포스트제목.md'파일을 생성한다.
jekyll은 각 포스트마다 맨 처음에 **프론트매터**라는 것을 넣어줘야 한다. 이유는 jekyll이 사이트를 빌드할 때, 프론트매터를 기준으로 포스트들을 빌드하기 때문이다. 그래서 데이터베이스가 없어도 페이지간 링크나 카테고라이징, 태깅, 인터랙션이 가능한 것이다.
minimal-mistakes의 프론트매터는 다음과 같다.
```ruby
---
published: true
layout:         ## 여러 레이아웃이 있다. "single", "splash" ...
title:          ## 포스트 제목
category:       ## 포스트 카테고리
tags:           ## 포스트 태그
comments:       ## 댓글 기능
---
```

위 프론트매터 다음부터, 마크다운 문법에 따라 [(마크다운 치트시트)](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet) 맘대로 글을 쓰고, 깃헙에 푸시하면 된다. 깃허브 pages기능이 온 되어 있으면 깃허브가 jekyll을 빌드해 요청에 응답하게 된다.

