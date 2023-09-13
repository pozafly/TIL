# critical-css

> [출처](https://web.dev/i18n/ko/extract-critical-css/)

critical css 기술로 렌더링 시간을 개선하는 방법.

<br/>

브라우저는 페이지를 표시하기 전에 CSS 파일을 다운로드하고 구문 분석해야 CSS를 렌더링 차단 리소스로 만든다. CSS 파일이 크거나 네트워크 상태가 좋지 않은 경우 CSS 파일을 요청하면 웹 페이지가 렌더링되는 데 걸리는 시간이 크게 늘어날 수 있다.

> Critical CSS는 사용자에게 가능한 한 빨리 콘텐츠를 렌더링하기 위해 스크롤 없이 볼 수 있는 콘텐츠에 대한 CSS를 추출하는 기술이다.

![image](https://github.com/pozafly/TIL/assets/59427983/c414fd2a-1854-41f5-80f6-148b7e3c72ae)

> 스크롤 없이 볼 수 있는 부분은 스크롤 하기 전 페이지 로드 시 뷰어가 보는 모든 콘텐츠다. 무수히 많은 장치와 화면 크기가 있기 때문에 스크롤 없이 볼 수 있는 콘텐츠 이상으로 간주되는 보편적으로 정의된 픽셀 높이는 없다.

`<Head>`에서 추출된 스타일을 인라인하면 이런 스타일을 가져오기 위해 추가로 요청할 필요가 없다. CSS 나머지 부분은 비동기식으로 로드할 수 있다.

![image](https://github.com/pozafly/TIL/assets/59427983/f352bf39-70b1-4ede-8b5e-92b0e5958586)

렌더링 시간을 개선하면 특히 열악한 네트워크 조건에서 [인지된 성능](https://web.dev/rail/#focus-on-the-user)에 큰 차이를 만들 수 있다. 모바일 네트워크에서는 대역폭에 관계없이 높은 대기 시간이 문제다.

![image](https://github.com/pozafly/TIL/assets/59427983/2fc89f5a-076c-4bdb-8ea9-7b02a943abe3)

-> 3G 연결에서 렌더링 차단 CSS가 있는 페이지(상단)와 인라인 중요 CSS가 있는 동일한 페이지(하단) 로드 비교

[FCP(First Contentful Paint)](https://web.dev/fcp/)가 불량하고 Lighthouse 감사에서 "렌더링 차단 리소스 제거" 기회가 표시되는 경우 중요 CSS를 사용하는 것이 좋다.

![image](https://github.com/pozafly/TIL/assets/59427983/25863a05-5ab7-46a5-a4f8-a338cfbcd578)

> 갓차
>
> 많은 양의 CSS를 인라인하면 HTML 문서의 나머지 부분의 전송이 지연된다는 점에 유의하자. 모든 것이 우선시되면 아무것도 아니다. 인라인은 또한 브라우저가 후속 페이지 로드에서 재사용하기 위해 CSS를 캐싱하는 것을 방지한다는 점에서 몇 가지 단점이 있으므로 드물게 사용하는 것이 가장 좋다.

첫 번째 렌더링 왕복 횟수를 최소화 하려면 스크롤 없이 볼 수 있는 콘텐츠를 **14KB** (압축) 미만으로 유지하는 것을 목표로 한다.

> 새로운 [TCP](https://hpbn.co/building-blocks-of-tcp/) 연결은 클라이언트와 서버 사이에서 사용 가능한 전체 대역폭을 즉시 사용할 수 없으며, 모두 연결할 수 있는 것보다 많은 데이터로 연결에 과부하가 걸리는 것을 피하기 위해 [느린 시작](https://hpbn.co/building-blocks-of-tcp/#slow-start)을 거친다. 이 과정에서 서버는 적은 양의 데이터로 전송을 시작하고, 완벽한 상태로 클라이언트에 도달하면 다음 왕복에서 양의 2배가 된다. 대부분의 서버에서 첫 번째 왕복에서 전송할 수 있는 최대 크기는 패킷 10개 또는 약 14KB이다.

이 기술로 달성할 수 있는 성능 영향은 웹사이트 유형에 따라 다르다. 일반적으로 사이트에 CSS가 많을 수록 인라인 CSS 의 영향도 커진다.

<br/>

## 도구 개요

페이지의 중요한 CSS를 자동으로 결정할 수 있는 훌륭한 도구가 많이 있다. 이 작업을 수동으로 수행하는 것은 지루한 프로세스이기 때문에 이것은 좋은 소식이다. 뷰포트의 각 요소에 적용되는 스타일을 결정하려면 전체 DOM을 분석해야 한다.

### Critical

[Critical](https://github.com/addyosmani/critical)은 스크롤 없이 볼 수 있는 CSS를 추출, 축소 및 인라인하여 [npm 모듈](https://www.npmjs.com/package/critical)로 사용할 수 있다. Gulp 또는 Grunt와 함께 사용할 수 있고 [webpack 플러그인](https://github.com/anthonygore/html-critical-webpack-plugin)도 있다.

프로세스에서 많은 생각을 필요로 하는 단간한 도구다. 스타일 시트를 지정할 필요도 없다. Critical은 자동으로 스타일시트를 감지한다. 또한 여러 화면 해상도에 대해 중요한 CSS 추출을 지원한다.

### CriticalCSS

[CriticalCSS](https://github.com/filamentgroup/criticalCSS) 는 스크롤 없이 볼 수 있는 CSS를 추출하는 또 다른 [npm 모듈.](https://www.npmjs.com/package/criticalcss) CLI로도 사용 가능.

중요한 CSS를 인라인하고 축소하는 옵션은 없지만 실제로 중요한 CSS에 속하지 않는 규칙을 강제로 포함할 수 있고 `@font-face` 선언을 포함하는 것에 대해 더 세부적으로 제어할 수 있다.

### Penthouse

[Penthouse](https://github.com/pocketjoso/penthouse)는 사이트나 앱에 DOM에 동적으로 주입되는 많은 수의 스타일이나 스타일이 있는 경우에 좋은 선택이다. (Angular 앱에서 일반적). 내부적으로 [Puppeteer](https://github.com/GoogleChrome/puppeteer)를 사용 하며 [온라인 호스팅 버전](https://jonassebastianohlsson.com/criticalpathcssgenerator/)도 제공한다.

Penthouse는 스타일시트를 자동으로 감지하지 않으므로 중요한 CSS를 생성할 HTML 및 CSS 파일을 지정해야 한다. 장점은 많은 작업을 병렬로 실행하는 데 좋다.
