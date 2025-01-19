# 더 최신의 CSS Reset

> [출처](https://ykss.netlify.app/translation/a_more_modern_css_reset/)

## 전체 Reset 코드

```css
/* box-sizing 규칙을 명시합니다. */
*,
*::before,
*::after {
  box-sizing: border-box;
}

/* 폰트 크기의 팽창을 방지합니다. */
html {
  -moz-text-size-adjust: none;
  -webkit-text-size-adjust: none;
  text-size-adjust: none;
}

/* 기본 여백을 제거하여 작성된 CSS를 더 잘 제어할 수 있습니다. */
body,
h1,
h2,
h3,
h4,
p,
figure,
blockquote,
dl,
dd {
  margin-block-end: 0;
}

/* list를 role값으로 갖는 ul, ol 요소의 기본 목록 스타일을 제거합니다. */
ul[role='list'],
ol[role='list'] {
  list-style: none;
}

/* 핵심 body의 기본값을 설정합니다. */
body {
  min-height: 100vh;
  line-height: 1.5;
}

/* 제목 요소와 상호작용하는 요소에 대해 line-height를 더 짧게 설정합니다. */
h1,
h2,
h3,
h4,
button,
input,
label {
  line-height: 1.1;
}

/* 제목에 대한 text-wrap을 balance로 설정합니다. */
h1,
h2,
h3,
h4 {
  text-wrap: balance;
}

/* 클래스가 없는 기본 a 태그 요소는 기본 스타일을 가져옵니다. */
a:not([class]) {
  text-decoration-skip-ink: auto;
  color: currentColor;
}

/* 이미지 관련 작업을 더 쉽게 합니다. */
img,
picture {
  max-width: 100%;
  display: block;
}

/* input 및 button 항목들이 글꼴을 상속하도록 합니다. */
input,
button,
textarea,
select {
  font: inherit;
}

/* 행 속성이 없는 textarea가 너무 작지 않도록 합니다. */
textarea:not([rows]) {
  min-height: 10em;
}

/* 고정된 모든 항목에는 여분의 스크롤 여백이 있어야 합니다. */
:target {
  scroll-margin-block: 5ex;
}
```

<br/>

## 분석

```css
/* box-sizing 규칙을 명시합니다. */
*,
*::before,
*::after {
  box-sizing: border-box;
}
```

모든 요소와 의사 요소가 기본 `context-box` 가 아닌 `border-box` 를 사용해 크기를 조정. [유동적인 타입과 공간](https://utopia.fyi/)을 가진 [유연한 레이아웃](https://every-layout.dev/)으로 [브라우저가 더 많은 작업을 수행](https://buildexcellentwebsit.es/)하도록 하는 데 더 중점을 두고 있기 때문에 이 규칙은 예전만큼 유용하지 않다. 하지만 플젝 어딘가 명시적 크기 조정이 없는 경우는 드물기 때문에 리셋에 포할될 수 있다.

```css
/* 폰트 크기의 팽창을 방지합니다. */
html {
  -moz-text-size-adjust: none;
  -webkit-text-size-adjust: none;
  text-size-adjust: none;
}
```

[이에 대한 가장 좋은 설명은 Kilian의 글.](https://kilianvalkhof.com/2022/css-html/your-css-reset-needs-text-size-adjust-probably/) 이 글에서 그는 그 못생긴 접두사가 왜 여전히 필요한지도 설명한다.

```css
/* 기본 여백을 제거하여 작성된 CSS를 더 잘 제어할 수 있습니다. */
body,
h1,
h2,
h3,
h4,
p,
figure,
blockquote,
dl,
dd {
  margin-block-end: 0;
}
```

[보다 거시적인 수준에서 흐름과 공간을 정의](https://andy-bell.co.uk/my-favourite-3-lines-of-css/)한다는 의미에서 여백에 대한 사용자 에이전트 스타일을 제거하는 것을 선호합니다. 이전 reset에서는 모든 측면을 제거했으나 논리적 속성을 사용해 이제는 끝 여백만을 제거할 수 있으며, 이는 프로덕션 환경에서도 잘 작동하는 것 같다.

```css
/* 목록 역할이 있는 ul, ol 요소에서 기본 목록 스타일을 제거합니다. */
ul[role='list'],
ol[role='list'] {
  list-style: none;
}
```

Safari는 여러 이상 기능을 제공한다. [그중 하나](https://bugs.webkit.org/show_bug.cgi?id=170179)는 목록 스타일을 제거하면 보이스 오버의 의미도 제거한다는 것이다. 어떤 사람은 이걸 기능이라 하고, 어떤 사람은 버그라고 한다. role이 추가 되었는지 확인하기 위해 약간의 보상으로 목록 스타일을 기본적으로 제거 했다.

```css
/* 핵심 body의 기본값을 설정합니다. */
body {
  min-height: 100vh;
  line-height: 1.5;
}
```

가독성 좋은 line-height를 좋아하고, 그것을 상속받는 것이 좋다. body의 최소 높이를 `100vh`로 설정하면, 특히 장식적인 요소를 설정할 때 매우 편리하다. `dvh`와 같은 새로운 단위를 사용하고 싶을 수도 있지만, [Ahmad처럼 깊이 파고들어보면](https://ishadeed.com/article/new-viewport-units/#:~:text=Be careful with the dvh,is scrolling up or down.) 그것이 해결책이 되기보다는, 더 많은 문제를 일으킬 수 있다는 것을 알 수 있다. 그리고 그것은 리셋의 목적에 부합하지 않다!

[`svh`단위가 `dvh`보다 나은 것 같지만](https://mastodon.social/@simevidas/111088262361593466), 저는 `vh`로도 만족합니다. 새로운 단위에 몰두하고, 그것을 진리처럼 추천하기 전에 항상 새로운 단위에 대해 충분히 이해하고, 그것에 대해 심층적으로 파고들어 보는 것이 좋다. 이미 잘 작동하고 있는 것이 있다면 굳이 바꿀 필요는 없다!

```css
/* 제목 요소와 상호작용하는 요소에 대해 line-height를 더 짧게 설정합니다. */
h1,
h2,
h3,
h4,
button,
input,
label {
  line-height: 1.1;
}
```

전체적으로 `line-height`를 넉넉하게 설정하는 것이 편리한 것과 마찬가지로, 제목과 버튼 등의 `line-height`를 짧게 설정하는 것도 편리하다. 하지만 글꼴에 큰 어센더(ascender)와 디센더(descender)가 있는 경우, 이 규칙을 제거하거나 수정하는 것이 좋다. 글꼴이 서로 충돌하여 접근성 문제를 일으키는 것은 피해야 할 상황이기 때문.

```css
/* 제목에 대한 text-wrap을 balance로 설정합니다. */
h1,
h2,
h3,
h4 {
  text-wrap: balance;
}
```

이 규칙은 우리의 프로젝트에만 국한될 수도 있는 규칙이지만, 최근에 나온 `text-wrap` 속성을 사용하면 제목을 아름답게 표시할 수 있다. 그러나 이 규칙이 적절하지 않다고 생각하는 사람도 있을 것이라 생각. 그렇다면 이 규칙을 삭제해도 좋음.

```css
/* 클래스가 없는 기본 a 태그 요소는 기본 스타일을 가져옵니다. */
a:not([class]) {
  text-decoration-skip-ink: auto;
  color: currentColor;
}
```

이 규칙은 먼저 텍스트 장식이 어센더와 디센더를 간섭하지 않는지 확인. 현재 대부분의 브라우저에서 이 기능이 기본값으로 설정되어 있다고 생각하지만, 이것을 설정하는 것도 좋은 보험 정책이다. [Set Studio](https://set.studio/)에서는 링크가 텍스트의 `currentColor`를 기본적으로 상속하도록 설정하는 것을 선호하지만, 원하지 않는다면 이를 제거하는 것이 좋다.

```css
/* input 및 button 항목들이 글꼴을 상속하도록 합니다. */
input,
button,
textarea,
select {
  font: inherit;
}
```

`font: inherit` 단축 속성은 정말 유용하며, 특히 input 및 form 요소에서 사용할 때 효과적. 주로 `<textarea>` 요소에 영향을 주지만, 다른 form 요소에도 적용해도 나쁠 것은 없다. 이렇게 함으로써 프로젝트 후반에 일부 CSS를 절약할 수 있다.

```css
/* 행 속성이 없는 textarea가 너무 작지 않도록 합니다. */
textarea:not([rows]) {
  min-height: 10em;
}
```

`<textarea>` 요소에 대한 이 규칙은 유용하다. 기본적으로 `rows` 속성을 추가하지 않으면 요소가 매우 작아질 수 있다. 이는 손가락과 같은 더 거친 포인터에는 이상적이지 않다. 그리고 `<textarea>` 요소는 여러 줄의 텍스트에 사용되기 때문에 이를 더 쉽게 만드는 것이 합리적이다.

```css
/* 고정된 모든 항목에는 여분의 스크롤 여백이 있어야 합니다. */
:target {
  scroll-margin-block: 5ex;
}
```

마지막으로 요소가 고정되어 있는 경우, 해당 요소가 타겟팅된 경우에만 고려되는 `scroll-margin`을 사용하여 그 위에 약간의 여유 공간을 추가하는 것이 좋다. 이 약간의 조정으로 사용자 경험을 크게 개선할 수 있다! 하지만 고정 헤더가 있는 경우라면 이를 조정해도 좋다.
