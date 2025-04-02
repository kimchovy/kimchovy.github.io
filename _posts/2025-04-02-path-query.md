---
title: Path Variable 그리고 Query String
description: >-
    웹에서 데이터를 전달하는 방식에는 여러 가지가 있는데, 그중 Path Variable과 Query String은 가장 자주 쓰이는 방식이다.
    하지만 초보들은 이 둘을 헷갈리는 경우가 있는데, 이번 글에서 각각의 개념과 실제 사용 예제까지 살펴보자.
date: 2025-04-02 16:24:00 +0900
categories: [CS, Network, Web]
tags: [Web, Parameter, Path, Query, String, REST, API]
pin: true
---

## 들어가기에 앞서 : 쿼리 파라미터(Query Parameter)? 쿼리 스트링(Query String)?

- 다음과 같은 URL이 있다. `https://example.com/search?keyword=post&sort=recent`
- `?` 이후에 나오는 부분이 `쿼리 스트링(Query String)`이고,
- `keyword=post`와 `sort=recent`는 각각 하나의 `쿼리 파라미터(Query Parameter)`이다.
> 여러 개의 쿼리 파라미터를 `&` 연산자로 연결할 수 있습니다.

------
## Path Variable(경로 변수)

- URL 경로 자체에 값을 포함하는 방식입니다.
- 보통 `특정 리소스`를 `식별`할 때 사용합니다.
> 이와 같은 이유로 `Path variable`은 주로 `중복되지 않는` 유일한 값(Unique value)이 들어가게 됩니다.
>> 그러나 비유니크한 값으로 사용할수도 있으며, 이 경우에는 추가적인 방법으로 자원을 식별하는 과정이 필요합니다.
- 이 방식은 리소스 간의 `계층적 관계`를 명확하게 표현할 수 있습니다.
- RESTful API[^rest-api]에서 사용하는 방식입니다.

### Path Variable 예시

```
https://example.com/users/31
```
> 일반적인 Path Variable의 예시입니다.
>> 여기서 `31`은 Path Variable이며, 특정 유저의 ID를 나타냅니다.
>>> 즉, 이 URL은 ID값이 31인 유저에 대한 정보를 요청하는 것입니다.

```
GET /items/58
```
> RESTful API의 요청 예시입니다.
>> 여기서는 `58`이 path variable이 되며, 특정 아이템의 ID를 나타냅니다.
>>> 즉, 위의 예시는 ID값이 58인 아이템의 정보를 요청하는 것입니다.
>>>> 이것을 전체 URL로 표현한다면, `https://example.com/items/58`과 같이 나타낼 수 있습니다.

------
## Query String

- `URL` 뒤에 `?`를 붙여 키=값 쌍으로 데이터를 전달합니다.
> '&' 기호로 키=값 쌍을 추가 및 구분합니다.
- 보통 검색이나 필터링, 옵션 전달을 위해 사용합니다.

### Query String 예시

```
https://example.com/search?query=깃허브&sort=recent
```
> 일반적인 Query String의 예시입니다.
>> 예시 설명: example.com에서 검색(search)을 하는데, 검색어(query)는 깃허브이고(&), 정렬(sort)은 최신순(recent)으로

```
GET /search?keyword=노트북추천&page=3
```
> RESTful API의 요청 예시입니다.
>> 예시 설명은 생략하도록 하겠습니다.
>>> 전체 URL로 표현하면, `https://example.com/search?keyword=노트북추천&page=3`와 같이 나타낼 수 있습니다.


------
## Path Variable과 Query String의 차이점

|구분|Path Variable|Query String|
|------|---|---|
|데이터 위치|URL 경로 자체에 포함|URL의 `?`뒤에 `key=value 형식`으로 추가|
|주 사용 목적|특정 `리소스 식별`|필터링, 정렬, 검색, 페이지네이션[^pagenate]등에 사용|
|예시|`/users/31`|`/search?keyword=노트북추천&page=3`|
|RESTful 설계|O(자주 사용됨)|X(보조 역할)|

------
## 실전 예제: 네이버 웹툰 조회

- 다음은 필자가 좋아하는 웹툰 중 하나인 '입학용병'의 페이지 캡쳐본이다.

- ![webtoon1](/assets/img/posts/path-query/네이버웹툰캡쳐1.png)

- 위의 URL에 주목해보자.

```
https://comic.naver.com/webtoon/list?titleId=758150&tab=sun
```
> 위에서 볼 수 있듯이, `https://comic.naver.com/webtoon/list`뒤의 `?`를 시작으로하는 `Query String`이 보일것이다.
>> `&`를 기준으로 각각 `titleId=758150`, `tab=sun` 총 두개의 `Query Parameter`가 사용되었다.

- 다음은 해당 웹툰의 222화를 클릭했을때의 화면 캡쳐본과 URL이다.

- ![webtoon2](/assets/img/posts/path-query/네이버웹툰캡쳐2.png)

```
https://comic.naver.com/webtoon/detail?titleId=758150&no=223&week=sun
```
> 위의 URL에서 `no=223`의 `값`에 해당하는 `223`을 다른 숫자로 바꿔보자.

- 해당 값을 변경할때마다 그 값에 해당하는 페이지로 이동되는 것을 볼 수 있을 것이다.

    - 만일 존재하지 않는 값(숫자)을 넣을 경우 자동으로 해당 웹툰의 메인홈으로 이동(Redirection)[^redirection]될 것이고, 

    - 숫자가 아닌 문자를 넣었을때는 상태 코드(Status code) 500(Internal Server Error)을 응답받으며, 

    - '죄송합니다, 오류입니다.'라는 텍스트가 포함된 페이지가 뜨는 것을 확인 할 수 있었다.

- 필자는 이후 독자들이 방문하는 여러 웹사이트에서 URL의 형태를 확인하고, 파악하고, 직접 변경해보며 익숙해지길 바라면서 글을 마치겠다.

------
## Reference

<https://fabric0de.tistory.com/55>

<https://growth-msleeffice.tistory.com/65>

<https://achieve-dev.tistory.com/51>

------
[^rest-api]: `RESTful API`는 `Representational State Transfer`의 약자로, 웹 표준을 기반으로 서버와 클라이언트 사이의 `상호작용`을 `정의`하는 방법입니다.
[^pagenate]: `페이지네이션(Pagination)`은 콘텐츠를 `여러 페이지로 나누어 표시하는 기술` 또는 기법을 의미합니다. `페이징(Paging)`이라고도 불립니다. 
[^redirection]: `웹페이지`의 `리다이렉션(redirection)`은 사용자가 `요청한 URL`과 `다른 URL`로 `자동으로 이동`하는 것을 말합니다.