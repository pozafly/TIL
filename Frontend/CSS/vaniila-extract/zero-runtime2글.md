# zero-runtime2글

브라우저는 HTML, CSS, JS 파일을 통해 화면을 그린다. HTML 파일을 파싱하여 DOM Tree를 그리고, CSS 파일을 파싱하여 CSSOM(또는 CSSOM Tree)를 그린다. 그러면 DOM과 CSSOM를 합쳐 Render Tree를 그리고, layout, paint 과정이 끝난 후에 실제 눈으로 볼 수 있는 화면이 그려지는 것이다. 이 과정은 CRP(Critical Rendering Path)를 아주 간략화 한 것이다.

[CRP]

브라우저는 HTML, CSS, JS 파일 외 다른 파일은 알지 못한다. 하지만, 웹 어플리케이션을 만들면서 다른(superset) 언어를 사용하기도 하며 다른 확장자를 가진 파일을 통해 CSS를 만들어내기도 한다. 개발자가 코드를 작성하고 브라우저에 그려지기까지의 과정을 짚어보면서 어떻게 하면 더 효율적으로 화면을 그릴 수 있는지 알아보자.

<br/>

## CSS 파일이 브라우저에 적용되는 3가지 방법

기초적으로 짚고 넘어가야할 부분이다. CSS는 브라우저에 적용되기 위해 3가지 방법이 있는데, 이는 섞어 사용할 수도 있고 하나의 방법만으로도 사용할 수 있다.

- Inline(인라인) CSS
- Internal(내부) CSS
- External(외부) CSS

### Inline CSS

초기에 HTML은 문서를 전달하기 위해 만들어졌다. HTML로만 문서를 전달하다보니 중요한 콘텐츠에 강조 표시를 하거나, 레이아웃을 다르게 하고 싶은 요구가 생겨났다. 그래서 HTML의 태그에 `style` attribute를 작성하면 글자 색상을 변경시키거나, 폰트 크기를 변경시킬 수 있었다.

```html
<p style="color: blue; font-size: 18px;">
  content here
</p>
```

이런 방식으로 CSS를 입히는 것을 Inline CSS라고 한다.

### Internal CSS

Inline CSS로 스타일을 입히는 방식이 코드 중복이 많아지고 HTML을 읽을 경우 한 눈에 파악하기 어렵다는 단점이 있어 나타난 개념이다.

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

`<head>` 태그에 `<style>` 태그를 작성하고, 그 안에 Selector를 사용해서 스타일을 입힐 수 있다.

### External CSS

CSS 파일을 외부 리소스로 뺄 수 있는 방법이다. Internal CSS 같은 경우는, `<style>` 태그 내에 모든 CSS가 정의되어야 한다. 따라서 어느 섹션의 어느 부분이 스타일이 입혀지고 있는지 파악하기 어렵다. 따라서, 여러 개의 CSS 파일을 만들고 구역 별로 CSS를 여러 개 작성하면 개발자가 코드를 파악하기 쉬워진다.

```html
<head>
  <link rel="stylesheet" type="text/css" href="./main.css">
  <link rel="stylesheet" type="text/css" href="../side/side.css">
</head>
```

`<link>` 태그에 외부 경로를 작성하면 된다. 브라우저가 HTML을 파싱하면서 `<head>`의 `<link>`를 만나면 href 경로를 통해 외부 `.css` 파일을 다운로드 받기 시작한다.

> ※ `<link>` 태그로 외부 CSS 파일을 로드할 경우, HTML을 파싱하는 과정은 멈추지 않는다. 다만 Rendering만 Block 될 뿐이다. [링크](https://www.mo4tech.com/does-css-block-dom-parsing.html)

---

3가지 방법으로 브라우저는 웹 페이지에 스타일을 입힐 수 있는데, 이 부분에서 CRP를 다시 상기시켜보자. 화면이 그려지는 과정은 HTML 파일을 파싱하여 DOM Tree가 그려지고, CSS를 통해 CSSOM이 그려져야 Render Tree를 만들 수 있다는 점이다.

따라서, 어떤 방법을 사용하든 CSS가 모두 파싱 된 후, **CSSOM이 만들어져야 브라우저가 화면을 그릴 수 있다**.

<br/>

## Critical CSS

[Critical CSS](https://web.dev/i18n/ko/extract-critical-css/)란, 사용자에게 가능한 빠르게 콘텐츠를 렌더링하기 위해 스크롤 없이 볼 수 있는 콘텐트에 대한 CSS를 추출하는 것을 말한다.

[Critical-CSS]

출처 : web.dev

웹페이지는 스크롤 없이 볼 수 있는 영역과 스크롤 해야 볼 수 있는 부분으로 나눌 수 있다. Critical CSS에서는 스크롤 없이 볼 수 있는 영역을 `above-the-fold` 라는 용어를 사용한다. 브라우저에서 URL을 입력하고 접속하면, 눈에 바로 보이는 영역이 스크롤 없이 볼 수 있는 영역이고, 스크롤 아래는 스크롤을 내리지 않는 한 아직은 보이지 않는다.

Critical CSS는 눈에 보이는 스크롤 없이 볼 수 있는 영역에 대한 CSS만 미리 추출해서 렌더링 하는 것을 말한다. 사용자가 웹페이지에 접속할 때, 웹페이지끼리 전부 연결되어있는 Global 한 CSS를 모두 가지고 있을 필요가 없고, 가시적인 부분의 CSS만 가지고 저 부분을 그릴 수 있다면 성능이 좋다고 말할 수 있을 것이다.

기본적으로 CSS는 [render-blocking Resourses](https://developer.chrome.com/docs/lighthouse/performance/render-blocking-resources/)다. Render Tree가 만들어져야 브라우저에서 Paint 과정으로 이어지기 때문이다. Critical CSS를 사용하면, [FCP(First Contentful Paint)](https://web.dev/fcp/) 지표를 올릴 수 있다.

그렇다면, 위에서 알아본 CSS를 브라우저에 적용하는 3가지 방법 중 어떤 방법을 통해 Critical CSS를 웹페이지에 적용할 수 있을까? 구현하기에 따라 다르겠지만, 일반적으로는 **Internal CSS** 방법을 사용한다.

- Inline CSS는 우선 코드 중복이 많거나 HTML과 CSS 코드 덩어리가 함께 있기 때문에 가독성이 좋지 않다.
- External CSS는 외부 리소스 파일이므로, 브라우저가 HTML을 다운로드 받은 후, 또다시 네트워크로 CSS 파일을 다운로드 받아와야 하기 때문에 적합하지 않다.

Internal CSS는 HTML 파일에 이미 속해있기 때문에 HTML이 다운로드되는 즉시 사용할 수 있다. 물론 눈에 보이는 부분의 CSS만 Internal CSS로 작성되기 때문에 크기가 크면 안된다. web.dev에 따르면 14kb 미만의 CSS를 권장한다고 한다.

### 최적화

눈에 보이는 부분 외, 스크롤을 내려야만 보이는 영역의 CSS는 어떻게 가져올 수 있을까? 여러 방법이 있겠지만, 일반적으로는 External CSS를 통해 비동기적으로 CSS 파일을 가져와 적용시킬 수 있다.

```html
<link href="styles.css">
```

[non-critical-css]

link 태그에 다른 attribute 없이 css 파일을 가져오면, FCP(First Contentful Paint)가 외부 CSS 파일을 다운로드 한 뒤 발생한다. 하지만, Critical CSS를 Internal CSS로 넣은 후, External CSS를 사용해 비동기로 CSS 파일을 가져오면 FCP 이후에 외부 CSS 파일을 다운로드 할 수 있다.

```html
<style type="text/css">
.accordion-btn {background-color: #ADD8E6;color: #444;cursor: pointer;padding: 18px;width: 100%;border: none;text-align: left;outline: none;font-size: 15px;transition: 0.4s;}.container {padding: 0 18px;display: none;background-color: white;overflow: hidden;}h1 {word-spacing: 5px;color: blue;font-weight: bold;text-align: center;}
</style>

<link rel="preload" as="style" href="styles.css" onload="this.onload=null;this.rel='stylesheet'">
<noscript><link rel="stylesheet" href="styles.css"></noscript>
```

`<style>` 내부는 Internal CSS로서 바로 적용된다. `<link>` 의 attribute로 `rel="preload"`가 사용되었다.

- `rel="preload"`와 `as="style"` 속성은 CSS 파일을 비동기적으로 요청한다.
  - preload : CRP에 필요한 리소스를 미리 로드한다. as와 묶여서 사용된다.
  - as : style, media, fetch, font 등의 리소스 종류를 넣을 수 있으며 중요도를 브라우저가 판단하게 한다. style은 최우선 순위이다. 종류는 [mdn](https://developer.mozilla.org/ko/docs/Web/HTML/Element/link#%ED%8A%B9%EC%84%B1)에서 확인할 수 있고, as의 중요도는 [Chrome의 프리로드, 프리페치 우선순위 블로그](https://medium.com/reloading/preload-prefetch-and-priorities-in-chrome-776165961bbf)에서 확인할 수 있다.
  - 또한, CSS를 적용하기 위해서는 `rel="preload"`가 아닌, `rel="stylesheet"`가 되어야한다. 따라서, 초기에는 stylesheet로 인식하지 않기 때문에 로드해와도 CSS로서 역할을 하지 못한다.
- `onload` : 외부 CSS 파일이 다운로드가 끝나면 실행된다
  - 이때, `rel="preload"` attribute를 `rel="stylesheet"`로 바꿔주어 CSS로서 동작할 수 있게 한다.
  - 또한, `onload=null`로 바꾸어 버리는데, `rel="stylesheet"` attribute가 바뀌면서 다시 onload 함수가 실행될 수 있기 때문에 `null`로 치환해주는 과정이다.

`<noscript>` 태그는 JavaScript를 실행하지 않는 브라우저에서 대체 수단으로 사용하기 위해 명시한 것이다.

[critical-css-external-css]

FCP와 동시에 외부 CSS 파일 로딩이 시작되는 것을 볼 수 있다.

### SSR 환경에서 CSS-in-JS의 Critical CSS

CSS-in-JS의 자세한 부분은 하단에서 설명할 것이다. 웹어플리케이션을 만들 때 흔히 사용하는 CSS-in-JS에서는 Critical CSS를 어떻게 사용하고 있을까? [andreipfeiffer/css-in-js](https://github.com/andreipfeiffer/css-in-js/blob/main/README.md#-critical-css-extraction)에 따르면, CSS-in-JS에서 말하는 Critical CSS는 바로 위에서 사용되고 있는 Critical CSS와는 조금 다르게 사용되고 있다.

- SSR에서는, 웹페이지 전체의 가시적인 부분(visible)의 CSS를 생성한다.
- 동적 렌더링 또는 지연 로딩(lazy loaded) 되는 element에 CSS를 생성하지 않는다.

위 두 개념으로 사용된다. 즉, 스크롤 없이 볼 수 있는 영역(above-the-fold)을 고려하지 않고 static한 전체 페이지 자체의 CSS만을 생성한다는 뜻이다. 대부분의 CSS-in-JS 라이브러리에서 개발 모드에서는 `<style>` 태그를 통해 스타일을 주입하고 있으며(Internal CSS), 프로덕션 모드에서는 라이브러리마다 다른 전략으로 스타일을 생성한다. `<style>` 태그를 사용해 스타일을 입히기도 하고, 외부 CSS 파일을 네트워크 요청으로 가져오는 방법(External CSS)을 사용하기도 한다.

> 📍 번들러 환경에서 개발 모드에서 `<style>` 태그를 사용하는 이유는 무엇일까?
>
> [Webpack5 JavaScript 보일러플레이트 만들기](https://pozafly.github.io/environment/webpack-boilerplate/#Mode%ED%99%98%EA%B2%BD-%EB%B6%84%EB%A6%AC)에서 살펴봤듯, webpack의 production 모드에서는 `MiniCssExtractPlugin`을 사용해 외부 CSS 파일로 만들어서 사용하며, dev 모드에서는 `style-loader`를 통해 `<style>` 태그를 통해 스타일을 입힌다.
>
> 이유는 효율성에 있다. webpack과 같은 번들러는 dev 모드에서 HMR(Hot Module Replacement)를 지원한다. 소스 코드를 변경했을 때, 외부 CSS 파일을 사용한다면 브라우저에서 매번 새로고침을 해주어야 하지만, style 태그는 HTML에 적용되어 있기 때문에 HTML만 Replacement 해준다면 새로고침하지 않아도 자동으로 변경된 화면을 볼 수 있기 때문이다.

어쨌든, SSR 환경의 CSS-in-JS에서는, 구글의 web.dev에서 말하는 Critical CSS 개념을 사용하지는 않고(above-the-fold 영역 고려하지 않음) 현재 웹 페이지에서 사용될 스타일 전체를 Critical CSS로 말하고 있다.

<br/>

## CSS-in-JS 스타일 적용 방법

CSS-in-JS는 크게 스타일을 적용하는 방법이 2가지다.

- 런타임 스타일시트(Runtime stylesheets)
- 정적 CSS 추출(Static CSS extraction)

### 런타임 스타일시트

CSS-in-JS는 번들링 된 JavaScript 파일이 브라우저에서 실행되면서 스타일이 입혀진다. 대부분의 CSS-in-JS 라이브러리가 채택하는 방식이다. HTML 파일이 브라우저에 로드되었을 때는 `<style>` 태그 및 외부 CSS 링크가 없는 상태로 로드된다. 그리고, JavaScript 파일안에 스타일 관련 소스파일이 있고, 일련의 과정을 거쳐 스타일을 입히게 된다.

런타임 스타일시트 방법은 또다시 2가지 방법으로 스타일을 줄 수 있다.

- `<style>` 태그를 추가한 후, Element에 스타일을 집어넣는 방식.
- CSSOM에 직접 스타일 rule을 추가하는 방식. ([CSSStyleSheet.insertRule()](https://developer.mozilla.org/en-US/docs/Web/API/CSSStyleSheet/insertRule))

코드로 표현하면 아래와 같다.

#### `<style>` 태그를 추가한 후, Element에 스타일을 집어넣는 방식

```js
const css = 'h1 { background: red; }',
const head = document.head || document.getElementsByTagName('head')[0],
const style = document.createElement('style');

head.appendChild(style);

style.type = 'text/css';
style.styleSheet.cssText = css;
```

[style-tag-append]

CSSOM에 직접 스타일 rule을 추가하는 방식보다는 성능이 좋지 않다. 하지만, 디버깅 측면에서 더 선호하는 방식이다.

#### CSSOM에 직접 스타일 rule을 추가하는 방식

```javascript
const sheet = document.styleSheets[0];
sheet.insertRule(
  'body{ border-bottom: 1px solid #CCC; color: #FF0000; }',
  0
);
```

`CSSStyleSheet.insertRule()` 메서드를 사용하면 CSSOM에 바로 스타일을 넣을 수 있다. 그러면 style 태그에 따로 추가되지는 않지만, CSS가 적용되는 것을 볼 수 있다. 즉, CSSOM 트리에 해당 규칙을 새로 생성하여 넣어준 것이다.

이는 **stitches.js** 와 같은 zero-runtime라이브러리에서 동적으로 스타일 규칙을 삽입할 때 사용하는 방법이다. 이 방법은 DOM에 transition이 걸려있다면 단순 규칙만 추가되는 것이므로 CSS-in-JS에서도 자연스럽게 잘 동작한다.

CSSOM에 rule을 추가하는 방법은, `<style>` 태그를 조작하는 방법보다 성능이 더 뛰어난 방법이다. 따라서 production 모드에서 더 선호되는 방법이다.

---

런타임 스타일시트는 어쨌든 JavaScript가 모두 로드된 후 런타임 환경에서 스타일을 생성해내기 때문에 번들링 된 JavaScript 파일이 브라우저에서 실행 이후에 스타일 생성이 된다. 따라서 스타일을 생성하는 **런타임 스타일 코드**가 필요하며, 이는 번들 파일에 포함이 된다.

런타임 스타일시트는 SSR 환경에서는 중복된 스타일 코드를 가질 수 있다. 우리는 CSS-in-JS 환경에서 최적화를 위해 Critical CSS를 사용하는 방법을 짚었고, CSS-in-JS의 Critical CSS의 Critical CSS는 페이지 전체의 CSS를 가져온다고 했다. 그렇다면, CSS-in-JS에 Critical CSS가 HTML을 통해 전달이 되었고, 런타임 스타일시트는 JavaScript 파일로 인해 두번 생성된다. 아래 그림과 같다.

[styles-shipped-twice]

### 정적 CSS 추출

CSS-in-JS가 나온 이후, 런타임에서 스타일을 만드는 작업이 오버헤드가 존재하기 때문에 더욱 속도를 높이기 위해서는 **빌드 시점에 CSS를 생성**하고, 생성된 CSS를 브라우저에 전달하여 적용시키는 작업의 개념이 나오게 된다. 이를 **zero-runtime**이라고 한다. 이는 사실 아래에서 살펴볼 Sass에서 이미 사용되는 개념이며 새로운 개념은 아니다. 단지 SPA 시대가 등장하며 CSS-in-JS를 통한 런타임 스타일 시트 방식에서 SSR 개념이 확대되면서 다시 CSS 자체를 서버에서 빌드하는 필요성이 생겨 zero-runtime이라는 용어가 등장했을 뿐이다.

정적 CSS 추출은 말 그대로 서버에서 빌드될 때 CSS 파일이 생성되어 브라우저에 전달된다. 생성된 CSS 파일은 `<link>` 태그를 통해 브라우저로 다운로드 된다.

Critical CSS를 사용한다면, 런타임 스타일시트 방식에서는 두 번 스타일을 입히게 되는 작업이 발생해야 하지만, 정적 CSS 추출은, [Critical](https://github.com/addyosmani/critical)과 같은 라이브러리를 통해 above-the-fold 영역의 Critical CSS를 따로 추출하여 사용할 수 있다.

아래는 SSR 환경에서 런타임 스타일시트 방식과 정적 CSS 추출이 가지는 파일의 사이즈다.

[file-size]

정적 CSS 추출은 보다시피 가장 기초적인 HTML, CSS, JS를 웹서버에서 브라우저로 서빙하는 형태를 그대로 갖추었다. 그래서 정적 CSS 추출 방식은 런타임 비용이 전혀 들지 않는다.

<br/>





## Tools

### Sass

CSS로만 웹 페이지 화면에 스타일을 입히다보니, 불편한 점이 많았기 때문에 Sass가 등장했다. 퓨어한 CSS 파일에 다양한 기능을 추가해 더욱 쉽고 유용하게 CSS를 작성할 수 있다. 대부분은 Sass 문법보다는 SCSS 문법을 사용할 것이다.

> Sass is the most mature, stable, and powerful professional grade CSS extension language in the world. ([Sass](https://sass-lang.com/))

CSS 확장(extension) 언어(language)로 소개하고 있다. 조금 뜯어보자.

#### 확장

CSS에는 중첩(nesting) 구문을 사용할 수 없다. 하지만, Sass는 중첩 구문을 사용할 수 있다. 중첩 구문을 사용하므로서 선택자 구조 파악이 훨씬 쉬워지고, 코드량 자체도 훨씬 줄어들었다. 또한, 모듈을 지원한다. CSS를 하나의 파일에만 전부 넣는 것이 아니라, 여러 파일에 모듈화하여 작성하고 나중에 하나로 합칠 수 있다. 믹스인으로 코드 뭉치를 하나로 사용할 수 있는 기능도 있다.

따라서 Sass는 CSS의 superset이라고 볼 수 있다.

#### 언어

Sass는 전처리기(Preprocessor)다. 브라우저는 CSS 파일밖에 알지 못하기 때문에 `.sass` 또는, `.scss` 확장자를 가진 파일 자체를 브라우저가 해석할 수 있도록 `.css` 파일로 만들어주어야 한다. 전처리기가 그 역할을 하고 있다.

처음 Sass가 등장했을 때는, Ruby 언어로 웹을 만드는 일이 많았기 때문에 [Ruby Sass](https://github.com/sass/ruby-sass) 라이브러리를 통해 Ruby 언어를 통해 SCSS를 CSS로 변환하여 브라우저에 전달했다. 후에 [LibSass](https://github.com/sass/libsass)가 나오게 된다. LibSass는 C/C++로 SCSS를 CSS 파일로 변환시키는 라이브러리다. LibSass는 JavaScript나, Go, Python, Java 등의 언어로 한번 감싸서 다양한 환경에서 컴파일 할 수 있는 core 로직이다.

JavaScript로 Sass를 컴파일 하고 싶을 때, LibSass 라이브러리의 한 종류인 [node-sass](https://github.com/sass/node-sass)가 있다. node-sass 라이브러리를 까보면 package.json 파일에 `lib/index.js` 파일이 진입점이며, `src` 디렉토리 내부에는 `.cpp`, `.hpp` 등의 파일이 존재한다. 하지만, Ruby Sass와 LibSass 라이브러리는 2019년에 deprecated 되었다. dart 언어로 동작하는 [Dart Sass](https://github.com/sass/dart-sass)가 등장했고, 앞으로는 dart를 사용해 컴파일 하기 때문이다.

신기하게 2023년 7월 Sass Blog에 [Sass in the Browser](https://sass-lang.com/blog/sass-in-the-browser/)라는 글이 올라왔다. Sass는 브라우저에서 컴파일하는 것이 아니라, 서버에서 컴파일해주어 CSS 파일이 브라우저로 전달되는 구조를 갖고 있었지만, 이제 브라우저에서도 Sass를 컴파일 할 수 있다.

```js
const sass = await import('https://jspm.dev/sass');
sass.compileString('body { background: #000; a { color: red; } }');
```

[sass-browser-compile]

브라우저 Dev Tool에 위 소스코드를 입력해보면, 정상적으로 컴파일된 CSS 문자열을 받아볼 수 있다. 따라서, Sass 공식 문서에 [playground](https://sass-lang.com/playground/) 페이지를 넣을 수 있게 되었다. 즉, 컴파일러 자체를 ESM으로 만드는데 성공한 것이다. 라이브러리 자체는 아직 공개되지는 않은 것 같다.

어쨌든, Sass는 한 시대를 풍미했던 강력한 라이브러리이자 언어다. 

Vue.js를 

<br/>

### PostCSS



## Styled-Components, Emotion





<br/>



> 참고
>
> - [render-blocking-resources](https://sia.codes/posts/render-blocking-resources/)
> - [defer-non-critical-css](https://web.dev/defer-non-critical-css/)
> - [Sass Guidelines](https://sass-guidelin.es/ko/#section)
> - [css-in-js/critical-css-extraction](https://github.com/andreipfeiffer/css-in-js/blob/main/README.md#-critical-css-extraction)