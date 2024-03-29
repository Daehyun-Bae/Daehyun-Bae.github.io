---
layout: post
title:  Github.io blog guideline
excerpt: 블로그 작성할 때 유의사항
date:   2021-03-10
categories: [Tutorial]
comments: false
---



# Introduction

Github.io 블로그를 만들어야겠다는 생각은 오래전부터 있었으나, 매번 테마만 적당히 바꿔보고 포스팅도 제대로 하지 못한 채 흐지부지되었었다. (이 블로그도 마찬가지)

지금 테마도 바꾸고 싶은 점들이 많지만 여기에 집중하다보면 또 흥미를 잃고 방치할 것 같아서 레이아웃 수정은 차차 시간이 날 때 하고 내용부터 채우려고한다.

아직 markdown 문법이 익숙하지 않아서 자주 까먹는 내용이나 문서 작성 시 유의해야할 점을 기록하기 위해 작성하였다.



틈나는대로 계속 업데이트하기





## Post 작성

### 1. 메타 데이터 작성하기

마크다운 파일 (`yyyy-mm-dd-title.md` )을 작성할 때 메타데이터 정보를 작성하자.

```
---
layout:		post
title:		title
excerpt:	exerpt
date:		yyyy-mm-dd
categories:	[category1, category2, ...]
comments: false
---
```

특히 `cateogories` 항목을 잘 채워넣어야 나중에 포스팅 분류할 때 편하게 정리할 수 있다.

`excerpt` 를 작성하면 메인 화면에서 본문 대신 해당 내용을 보여준다. 
해당 post를 설명하는 내용을 작성한다.



### 2. 수식 작성

수식 작성 할 때 `$$x = 1 $$` 와 같이 `$` 기호를 두 개씩 붙여서 감싸줘야 제대로 인식한다.

#### `$$ x = 1 $$` &rarr; $$ x = 1 $$ (O)

#### `$ x = 1 $` &rarr; $ x = 1 $ (X) 





## Layout / Style 수정

스타일 수정은 `_sass/pages/` 폴더의 `_layout.scss` 또는  `_post.scss`  파일을 수정하면 된다.

페이지 레이아웃은 `_layouts/`  폴더 참고

[Resume](https://daehyun-bae.github.io/resume) : 데이터는 `_data/index`, 레이아웃은 `_includes/sections` 에서 수정

* \<About> section: `_includes/sections/about.html`

[Tags](https://daehyun-bae.github.io/tags) : [Categories](https://daehyun-bae.github.io/categories) 탭과 큰 차이점은 잘 모르겠지만 태그를 사용하고 싶으면 메타데이터 정보를 작성할 때 `tags: tag1 `와 같이 태그 정보를 입려해주면 된다.



## Github blog 미리보기

```shell
# root_dir of github.io
$ bundle exec jekyll serve
```



## To Do

### Layout

Sidebar 크기 줄이기

~~본문 가로 길이 줄이기~~

~~아이콘 바꾸기~~

~~폰트 (크기, 스타일)~~



### Posting

~~네이버 블로그 포스팅 가져오기~~

~~어느정도 익숙해지면 Sample post 삭제~~

~~메인 화면 페이지 (home) 만들지 고민~~ &rarr; 일단은 생략

~~Resume 채우기~~