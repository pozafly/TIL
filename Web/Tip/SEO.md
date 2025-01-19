# SEO(검색엔진 최적화)

> [출처1](https://opentutorials.org/course/2039/10995), [출처2](https://velog.io/@jerrynim_/SEO%EB%A5%BC%EC%9C%84%ED%95%9C-%EB%85%B8%EC%98%A4%EC%98%A4%EB%A0%A5)

## SEO란?

SEO(Search Engine Optimization)는 검색어 최적화로써, 사이트가 검색자에게 적절하게 노출되도록 하는 것을 말함. 사이트를 검색어 최상단에 위치시키는 것, 사용자의 니즈에 맞춰 사이트를 노출시키는 것, 검색어에 관련된 사이트를 표시하는 것 또한 SEO라고할 수 있다.

프론트엔드 개발자로써 SEO를 위해 고려할 수 있는 항목은 다음과 같다.

- 제목(\<title>)
- 키워드(\<keyword>)
- url
- 콘텐츠(\<h1>)
- 대체 택스트(alt)

<br/>

<br/>

## 제목(\<title>)

\<head> 안에 \<title> 을 이용해 제목을 설정할 수 있으며, 다음과 같이 SERP에 표시된다.

> 📌 SERP(Search Engine Results Pages)는 검색결과 페이지다.

![1](https://user-images.githubusercontent.com/59427983/120089360-598ff980-c134-11eb-8efc-4fdb4fda33c8.png)

### 제목의 길이

알파벳 기준으로 제목의 길이는 60자 미만, 한글 기준으로 '가'를 최대 32번 사용할 수 있다.

![2](https://user-images.githubusercontent.com/59427983/120089366-66145200-c134-11eb-9db2-6decdbe4ef4e.png)

길어진 제목은 잘려져 `…` 으로 표시되기 때문에 적절한 제목을 표시해 사용자의 클릭을 유도해야 한다.

### 중복 타이틀 방지

모든 페이지에 고유한 제목을 지정하여 Google이 중복 된 콘텐츠로 판단하는 것을 방지하고, 검색자에게 원하는 정보를 클릭할 수 있도록 하여야 한다.

<br/>

<br/>

## 키워드(\<keyword>)

키워드는 콘텐츠의 내용을 정의하는 아이디어와 주제다. SEO 측면에서는 검색자가 검색 엔진에 입력하는 단어와 구문으로 '검색 쿼리' 라고도 함. 콘텐츠에 적합한 키워드를 사용하여 사용자에게 노출시키고, 키워드와 관련된 콘텐츠를 제공하여 사용자의 클릭률을 높여 검색 순위를 올릴 수 있다.

### 개념 타겟팅

키워드를 사용할 때 다음과 같은 방법은 좋지 않다.

> Buy Widgets, Best Widgets, Cheap Widgets, Widgets for Sale

키워드 목록에 불과한 제목이나 동일한 키워드의 변형을 반복해서 사용하지 마십시오. 이러한 제목은 검색 사용자에게 좋지 않으며 검색 엔진에 문제를 일으킬 수 있습니다. 구글에서는 개념을 공유하여 하나의 키워드로 충분히 대치할 수 있습니다.
[Tactical Keyword Research in a RankBrain World](https://moz.com/blog/tactical-keyword-research-in-a-rankbrain-world)

### 키워드 순서

Moz의 테스트 및 경험에 따르면 키워드 앞쪽의 키워드가 검색 순위에 더 많은 영향을 미친다고 한다. 또 사용자들은 `처음 두 키워드` 만 스캔할 수 있어 가장 독특한 키워드가 먼저 나타나는 것을 권장한다고 한다.

<br/>

<br/>

## 메타 설명(meta description)

메타 설명은 웹 페이지에 대한 간략한 요약을 제공하는 HTML 속성이다. \<head> 안에 \<meta> 를 이용해 다음과 같이 사용한다.

```html
<head>
  <meta name="description" content="This is description">
</head>
```

![3](https://user-images.githubusercontent.com/59427983/120089536-6dd4f600-c136-11eb-9e2f-f7444f024cfa.png)

결과적으로 메타 설명은 검색어 순위에 **영향을 주지 않음**. 하지만 검색자에게 유용한 설명을 제공함으로써 클릭률을 높일 수 있다.

> 구글은 2009/9 에 웹 검색을 위한 구글의 순위 알고리즘에 메타 설명이나 메타 키워드가 고려되지 않는다고 발표함.

메타 설명은 충분히 설명할 수 있을만큼 길게 유지하는 것이 가장 좋으므로 50~160자를 권장한다. 이 또한 알파벳 기준이기데 다음 그림과 같이 잘리게 된다.

![4](https://user-images.githubusercontent.com/59427983/120089569-bee4ea00-c136-11eb-865e-6e3f36691bb7.png)

### 주의사항

메타 설명에 큰 따옴표(`"`)를 사용하면 안된다. 큰 따옴표를 사용하게 된다면 SERP에서 큰 따옴표가 있는 부분을 자르게 된다. 대신에 [HTML엔티티](https://www.w3schools.com/html/html_entities.asp)를 사용해 `"` 를 사용할 수 있다.

<br/>

<br/>

## 대체 텍스트(alt text)

[이미지 SEO](https://github.com/pozafly/TIL/blob/main/Frontend/Web/%EC%9D%B4%EB%AF%B8%EC%A7%80%20SEO.md) 에 정리해둠.

<br/>

<br/>

## 중복방지

다음과 같은 이유로 페이지의 중복이 발생할 수 있다.

- 다양한 URL

  URL 매개 변수를 중복 콘텐츠 문제를 일으킬 수 있다.

![5](https://user-images.githubusercontent.com/59427983/120090375-e76fe280-c13c-11eb-9184-91285c6830ad.png)

  - **해결법**: 301 리디렉션

    대부분의 경우 중복 콘텐츠를 방지하는 가장 좋은 방법은 '중복' 페이지에서 원본 콘텐츠 페이지로 301 리디렉션을 설정하는 것이다.

- www와 https

  사이트의 주소가 "www.site.com" 및 "site.com", 혹은 'http://' 및 "https://" 처럼 다른 버전이 있다면 복제본이 생성되게 된다.

  - `Rel="canonical"`
    중복 콘텐츠를 처리하는 또 다른 옵션은 rel = canonical 속성 을 사용하는 것임. 검색엔진이 페이지가 특정 URL의 사본으로 처리됨을 알려줍니다. 또한 이 페이지에 부여된 링크, 콘텐츠, "ranking power" 은 특정 URL에 부여된다.

- 복제된 콘텐츠

  여러 웹 사이트에서 동일한 항목을 판매하고 모두 해당 항목에 대한 제조업체의 설명을 사용하는 경우 동일한 콘텐츠가 웹의 여러 위치에 표시된다.

  - Google Search Console 이용 https://moz.com/learn/seo/duplicate-content

<br/>

<br/>

## robots.txt

robots.txt 파일은 웹 크롤러(user-agents)가 웹 사이트를 크롤링 할 수 있는지 여부를 나타내게 된다.

ex) /robots.txt

```
User-agent: *
Allow: /
Sitemap: https://site.com/sitemap.xml

User-agent: AdsBot-Google
Disallow:
```

예시와 같이 User-agent, Allow, Disallow, Crawl-delay, Sitemap의 구문을 사용하며 user-agent를 지정하여 특정 페이지를 'allow'혹은 'disallow'할 수 있다.

사이트맵(sitemap)은 URL과 관련된 XML 사이트맵의 위치를 호출하는데 사용된다. 이 명령은 Google, Ask, Bing 및 Yahoo 에서만 지원된다.

### robots.txt의 장점

- SERP에 중복 콘텐츠가 표시되지 않도록 방지
- 웹 사이트의 전체 섹션을 비공개로 유지 (예: 개발 dev 사이트)
- 내부 검색 결과 페이지가 공개 SERP에 표시되도록 유지
- 사이트 맵 위치 지정
- 검색 엔진이 웹 사이트의 특정 파일(이미지, PDF 등)을 인덱싱하지 못하도록 방지
- 크롤러가 한 번에 여러 콘텐츠를 로드할 때 서버에 과부하가 걸리지 않도록 크롤링 지연(Crawl-delay) 지정

사이트에 사용자 에이전트 액세스를 제어하려는 영역이 없는 경우 robots.txt 파일이 전혀 필요하지 않을 수 있다.
