# CSS-in-JS 스타일 출력 방법

> [출처](https://andreipfeiffer.dev/blog/2021/css-in-js-styles-output)

서로 다른 두 가지 방법

- 대부분의 CSS-in-JS 라이브러리에 사용되는 런타임 스타일 시트.
- 소수의 라이브러리에 사용되는 정적 CSS 추출.

## 런타임 스타일 시트

`.js` 사용해 스타일 정의는 어플리케이션 로직과 함께 bundle에 포함된다.

```html
<!-- styles get bundled along with the components & app logic -->
<script src="bundle.js"></script>
```

브라우저는 JavaScript 파일에 포함된 스타일을 처리하는 방법을 모르기 때문에 런타임 라이브러리가 필요하다. 런타임 코드는 기본적으로 빌드 시 번들에 포함되고, 목적은 아래와 같다.

- JavaScript 번들에서 스타일을 읽는다.
- DOM에 스타일을 삽입한다.
- 이벤트로 인해 변경이 발생할 때마다 스타일을 업뎃 한다.

스타일을 삽입하고 업뎃하는 방법도 있다.

- 일반적으로 DOM API를 사용하여 문서에 하나 이상의 `<style>` 태그를 추가한다. `<head>` 에 추가하는 방법은 디버깅을 쉽게 하므로 개발 모드에서 사용된다. -> 소스맵이 필요 없어서?
  - 소스맵이 필요 없어서?
  - **Hot Reloading 및 코드 수정 편의성**: 개발 중에 CSS를 `<style>` 태그로 작성하면 코드 수정 후 즉시 변경 내용을 볼 수 있다. CSS 파일을 외부에서 로드할 경우, 파일을 저장하고 페이지를 새로고침해야 변경 사항이 적용되기 때문. -> 이게 가장 큰 이유. external CSS 코드로 link 태그로 불러오면 새로고침을 반드시 해주어야 한다.
- API를 사용해 CSSOM에서 직접 스타일 시트를 관리한다. `CSSStyleSheet`. 이는 DOM 내에서 태그를 조작하는 것보다(`<style>` 태그 조작) 성능이 더 뛰어나기 때문에 프로덕션 빌드에 선호되는 방법이다.

하지만, 두 가지 문제가 있다.

<br/>

### ⛔️ 런타임 라이브러리 오버헤드

런타임에 스타일 시트를 삽입하고 업뎃 하려면 추가 코드를 브라우저에 제공해야 함. 이 코드 크기는 `1k` 일 수도 있고, `18k`일 수도 있다.

CSR, SSR 에도 런타임 코드가 필요하다는 점을 기억하자.

![[assets/images/09b5d9b5666aaab7f44d2229947a51b6_MD5.png]]

### ⛔️ SSR을 사용한 중복된 스타일

SSR에서 CSS-in-JS는 렌더링 된 HTML 콘텐츠와 함께 소위 [Critical CSS 도 포함된다](https://github.com/andreipfeiffer/css-in-js/blob/main/README.md#-critical-css-extraction).

> 위 링크에 따르면, 스크롤 없이 볼 수 있는 부분의 Critical CSS 추출을 의미하지는 않는다.
>
> 실제로 하는 일:
>
> - SSR 중에, 그 페이지에서 초기에 보이는 요소에 대한 스타일만 생성한다. (즉, 페이지 above-the-fold가 아닌, 페이지 전체 CSS를 만든다.)
> - 동적으로 렌더링되거나 나중에 로드되는 요소에 대한 CSS는 생성하지 않는다.
>
> 100% 정적 CSS를 사용하면 실제로 이점이 없습니다. 서버에서 매우 적은 요소를 렌더링하는 동적 페이지와 대부분의 구성 요소가 클라이언트에서 동적으로 렌더링되면 이점이 증가합니다. -> 즉 실제로 Critical CSS를 사용하는 것이 아니기 때문에 이점이 제대로 없다는 뜻임.

즉, Critical CSS는 서버에서 생성된 정적 HTML 페이지에 필요한 모든 스타일을 나타낸다.

주의할 점은 Critical CSS 스타일이 브라우저에 **두 번 전달된다**는 것이다. 첫 번째는 HTML 파일이고, 두 번째는 re-hydration되는 JS 번들이다.

![[assets/images/0787301ed05a30bf65c9d3eb24eb72a4_MD5.png]]

<br/>

## 정적 CSS 추출

정적 추출은, JS 파일에 정의된 모든 스타일을 추출하고 프로덕션용으로 빌드할 때 `.css` 일반 파일을 생성하는 것이다. 이 방법을 사용하면 문서에 일반 CSS 스타일 시트로 스타일을 포함할 수 있다.

```html
<!-- styles are extracted as static .css files -->
<link rel="stylesheet" href="styles.css" />
```

스타일이 자체 파일에서 추출되었으므로, JavaScript 번들에는 구성요소와 어플리케이션 로직만 포함된다. 또한, 브라우저가 기본적으로 파일 작업을 수행할 수 있으므로 추가 런타임 라이브러리가 필요하지 않는다.

### ✅ 더 작은 리소스

더 작은 번들이 생성되므로 브라우저에 전달되는 바이트 수가 줄어든다. 또한 SSR을 사용하면 추가적인 Critical CSS도 필요하지 않으므로 더 많은 바이트를 절약할 수 있다. (위에서 살펴본 runtime Critical CSS 말하는 것임)

추가 최적화 기술로 정적 CSS 추출과 함께 Critical CSS를 사용할 수도 있다. [Critters는](https://github.com/GoogleChromeLabs/critters) 런타임 스타일시트가 있는 CSS-in-JS 라이브러리와 유사하게 중요한 CSS를 추출하는 반면 [Critical은](https://github.com/addyosmani/critical) 중요한 경로(above-the-fold) CSS를 추출한다.

![[assets/images/f02193a9562677f8d88b9ed93a0dc098_MD5.png]]

정적 CSS 추출은 동적 스타일을 처리하기 위해 일부 런타임 코드가 필요하지 않는 한 일반적으로 **런타임 비용이 전혀 들지 않는다**. 최종 결과는 CSS 스타일 시트의 모든 장점과 단점을 공유하는 일반 CSS, 전처리기 또는 CSS 모듈과 같은 비 CSS-in-JS 솔루션을 사용하는 것과 유사하다.

<br/>

## 로드 시간은 어떠한가?

**크기**는 우리가 분석할 수 있는 지표 중 하나일 뿐이며, **시간**도 다른 지표다. 그렇다면?

> 페이지 크기가 작을 수록 로딩 시간도 빨라지나?

이 질문에 답하려면 브라우저가 페이지 리소스를 가져오기 위해 서버에서 보내는 **HTTP 요청**을 살펴봐야 한다. 각 HTTP 요청은 여러 [단계를](https://developer.chrome.com/docs/devtools/network/reference/#timing-preview) 거쳐야 하지만, 그 중 두 단계는 페이지 로드 성능을 연구할 때 관련성이 있고 중요한 역할을 한다.

1. 네트워킹 조건(DNS, TCP, SSL), 대기 시간 및 서버 응답에 따라 달라지는[**TTFB(Time to First Byte)**](https://developer.mozilla.org/en-US/docs/Glossary/time_to_first_byte)
2. **콘텐츠 다운로드** 시간은 리소스 크기와 인터넷 연결 대역폭에 따라 달라진다.

![[assets/images/9f2eb7c8fac015b5727b39cd4f95b480_MD5.png]]

개발자로서 통제할 수 없는 요소 중 하나는 사용자의 대역폭이다. 그들 중 일부는 빠른 Wi-Fi를 통해 페이지를 방문할 수 있고, 다른 일부는 느린 3G 모바일 연결을 사용할 수도 있다.

![[assets/images/fbf37ee6f563c7445fa9312f2e7818f9_MD5.png]]

---

이제 위에서 언급한 두 가지 방법을 사용해 로딩 폭포 차트가 웹페이지를 찾는 방법을 살펴보자. 일반적으로 정적 리소스에 대한 HTTP 요청은 비슷한 TTFB를 가지며 파일 크기에 따라 콘텐츠 다운로드 단계에 큰 차이가 반영된다.

![[assets/images/ab783941fc6caa2a8aa8e5773d45cbee_MD5.png]]

런타임 스타일 시트는 일반적으로 파일크기가 더 크지만, 정적 CSS 추출보다 더 빠른 First Paint 측정 항목을 제공할 수 있다.

**정적 CSS 추출**은 일반 `.css` 파일이 문서의 다른 스타일 시트도 포함된다는 것을 의미하므로 `<head>` 로딩 폭포는 친숙해보일 수 있다.

1. 브라우저는 HTML 파일을 다운로드 하고 구문 분석한다.
2. 구문 분석 중 브라우저는 파일 참조(`styles.css`)를 발견하게 되므로 `.css` 파일을 가져오기 위한 또 다른 요청, 즉 **렌더링을 차단하는** 작업을 수행한다.
3. 파일을 다운로드하고 구문 분석한 후 `.css` 에서 참조된 다른 리소스가 없으면 `<head>` 브라우저는 페이지 렌더링을 시작할 수 있다.

문서의 끝에 `.js` 파일을 포함한다고 하면, 브라우저는 `.css` 파일을 다운하고 구문 분석할 때까지 페이지 `<body>` 를 그릴 수 없음. 따라서 정적 CSS는 [렌더링 차단 CSS 리소스를](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/render-blocking-css) 도입한다.

반면 **런타임 스타일시트**는 로딩 폭포가 약간 다르다.

1. 브라우저는 HTML 파일을 다운로드 하고 구문분석한다.
2. 그런다음, JavaScript 파일 즉, `bundle.js` 및 `library_runtime.js` 를 로드한다.

body 문서의 끝에.js 파일이 있고, `<head>` 섹션에 다른 리소스가 없다면 HTML이 구문 분석 되자마자 화면에 **페이지를 그리기 시작할 수 있다**.

---

### ✅ 런타임 스타일시트로 페인트 시간 단축

다른 최적화 기술을 제외하면 런타임 스타일 시트는 외부 CSS 파일에 대한 추가 HTTP 요청이 필요하지 않기 때문에 더 빠른 [First Paint](https://developer.mozilla.org/en-US/docs/Glossary/First_paint) 측정 항목을 제공해야 함.

대용량 `.css` 파일은 지속적으로 증가하고 시간이 지남에 따라 First Paint 메트릭이 증가하는 경향이 있기 때문에 대규모 어플리케이션에서 문제가 될 수 있다. 이 문제를 극복하기 위한 다양한 기술을 가지고 있는 이유다.

- [Critical CSS를 인라인 처리](https://web.dev/extract-critical-css/) 하면 추가 HTTP 요청이 제거되고 전체 스타일시트의 일부만 미리 로드된다.
- [Atomic CSS](https://sebastienlorber.com/atomic-css-in-js)는 선형 곡선 대신 로그 증가 곡선을 사용해 CSS 코드 량을 제한함.

> NOTE:
>
> [TTI(Time to Interactive)는](https://web.dev/interactive/) 두 가지 방법 중 하나를 사용해 더 빠르든 느리든 비결정적이라는 점을 언급할 가치가 있음. 관련된 변수가 너무 많아 명확한 결론을 내릴 수는 없음.

<br/>

## 결론

런타임 스타일시트와 정적 CSS 추출은 모두 CSS-in-JS 라이브러리를 선택할 때 고려해야 할 유효한 옵션이다. 자연스럽게 서로 다른 목적에 적합하다 생각한다.

**런타임 스타일시트**는 일반적으로 CSR을 사용하는 SPA와 같은 매우 동적 솔루션에 더 잘 맞는 것 같다.

- Critical CSS는 필요하지 않으므로, 스타일의 일부를 브라우저에 두 번 전달하는 것을 피할 수 있음.
- 이런 어플리케이션에서는 일반적으로 데이터를 클라 측에서 가져오기 때문에 로딩 스피너를 표시하는 페인트 시간이 빨라지는 이점이 있음.
- SPA에서 일반적으로 사용되는 대부분 JavaScript 프레임워크는 해당 스타일도 포함하는 구성 요소를 지연 로드하는 간단한 방법을 제공한다.

반면 정적 CSS 추출은 SSG 및 SSR과 같은 보다 정적인 솔루션에 더 잘 맞는 것 같다.

- 관련 오버헤드가 없기 때문에 더 적은 바이트가 사용자에게 전달된다.
- CSS 스타일을 그대로 유지하면서 JavaScript 코드를 변경하면 캐싱이 향상되는 이점이 있음.

---

위 결론은 단지 일반적인 지침일 뿐이다. 일부 어플리케이션에서는 SSR/CSR/SSG를 결합한 하이브리드 렌더링이 필요하다.

특히 대규모 어플리케이션의 경우 고려해야 할 다른 고려 사항이 있다.

- 최초 사용자 또는 재방문자를 위해 최적화 해야하나?
- 새 빌드를 얼마나 자주 릴리즈 해 캐시를 무효화 하나?
- 스타일이나 구성 요소/어플리케이션 코드 중 무엇을 더 자주 변경하는가?
- (저사양) 모바일 장치에 맞게 최적화 해야 하나?
