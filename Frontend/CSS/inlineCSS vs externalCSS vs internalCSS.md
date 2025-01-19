# inlineCSS vs externalCSS vs internalCSS

> [출처](https://www.hostinger.com/tutorials/difference-between-inline-external-and-internal-css)

## 용법

### Inline CSS

`style` attribute를 태그에 직접 사용해 CSS 규칙 지정. 별도의 CSS 파일을 만들지 않고 요소에 빠르고 간단한 스타일 입힘.

```html
<p style="color: blue; font-size: 18px;">
  content here
</p>
```

특정 요소에만 적용됨.

### Internal CSS

HTML문서의 `<head>` 태그 내부에 `<style>` 태그로 스타일을 입힘.

```html
<head>
  <style>
    body {
      color: blue;
      font-size: 18px;      
    }
  </style>  
</head>
```

해당 HTML 문서에만 적용됨. 다른 페이지나 웹 사이트에서 사용하지 못함.

### External CSS

CSS 파일을 별도의 파일로 만들어, HTML `<head>` 태그의 `<link>` 태그로 `.css` 파일을 불러옴.

```html
<head>
  <link rel="stylesheet" type="text/css" href="style.css">
</head>
```

<br/>

## 우선순위

inline CSS > internal CSS > external CSS

하지만, external CSS 같은 경우 빠른 페이지 로드를 위해 캐시될 수 있다.

<br/>

## Critical CSS

Critical CSS는, 주로 Internal CSS로 작성한다. 스크롤을 내리지 않아도 처음 접속할 때, 스타일이 보이고 후에 나머지 스타일이 비동기적으로 로드되어야 하기 때문에 Internal CSS로 작성하는 것이 좋다.

HTML에서 `<style>` 을 만나면, HTML은 파싱을 멈추고, CSSOM을 먼저 만들기 때문이다.

inline으로 할 수도 있지만, CSS는 렌더링 차단 리소스이므로, 개별 DOM을 inlineCSS가 적용되어 있다면, 모든 CSS 구문을 분석하고 이해하는데 시간이 오래 걸릴 수 있기 때문이다.

inline은 CSS를 한땀한땀 넣어주어야 한다. 따라서 코드 중복이 생길 수 있다. inline CSS는 코드 중복 때문에 CSS 량이 늘어나고, 그러면 오히려 시간이 더 걸릴 수 있다. (HTML은 점진적으로 파싱되지만, CSS는 선택자 중복이 있을 수 있으므로 브라우저에서 한번에 모든 CSS를 읽어들여 한번에 파싱한다. [링크](https://nitropack.io/blog/post/critical-rendering-path-optimization))

오히려 First Paint 시간이 오래 걸리기 때문에 inline CSS는 일반적으로 Critical CSS 를 설정하는데 사용하지 않는다. [링크](https://blog.logrocket.com/improve-site-performance-inlining-css/)

또한, 인라인으로 되어있는 CSS를 변경하고자 할 때, 우선순위 때문에 `!important`를 사용해야하는 경우가 생긴다. 따라서, Internal CSS를 사용하는 것이 좋다.

---

그러면 external CSS를 비동기로 로드 작업은 어떻게 하는가?

```html
<link rel="preload" href="styles.css" as="style" onload="this.onload=null;this.rel='stylesheet'">
<noscript><link rel="stylesheet" href="styles.css"></noscript>
```

- `link rel="preload" as="style"`은 스타일시트를 비동기적으로 요청합니다. [중요 자산 사전 로드 가이드](https://web.dev/preload-critical-assets)에서 `preload`에 대해 자세히 알 수 있다.
  - `rel`이 `stylesheet`가 아니면, 실제로 CSS로서 역할을 하지 못한다. 단순 preload만 해옴. 그리고 load가 완료된 시점에 rel을 stylesheet로 바꿔주면서 스타일이 실제로 입혀지게 되는 것이다.
- `link`의 `onload` 속성을 사용하면 로드가 완료될 때 CSS를 처리할 수 있다.
- `onload` 핸들러를 사용한 후 이를 "nulling"하면 rel 속성을 전환할 때(preload -> stylesheet) 일부 브라우저가 핸들러를 다시 호출하지 않도록 할 수 있다.
- `noscript` 요소 내부의 스타일시트에 대한 참조는 JavaScript를 실행하지 않는 브라우저의 대체 수단으로 작동함.

[링크](https://web.dev/i18n/ko/defer-non-critical-css/#%EC%B5%9C%EC%A0%81%ED%99%94)
