# zero-runtime

> [출처1](https://velog.io/@jhlee910609/sass-%EA%B1%B0%EB%91%AC%EB%82%B4%EA%B3%A0-css-in-js-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0), [출처2](https://blog.logrocket.com/comparing-the-top-zero-runtime-css-in-js-libraries/)

## sass-loader 단점

- build 속도가 너무 느리다.
- scope className이 없다.

## CSS-in-JS의 장점

- webpack의 css-loader를 사용하지 않는다. -> build가 없다.
- 따라서 브라우저 JavaScript 코드로 CSS 구문이 적힌 text가 파싱되어 런타임에 동작한다.
- 라이브러리 자체적으로 vender-prefix를 지원한다. ([링크](https://css-tricks.com/a-thorough-analysis-of-css-in-js/#aa-automatic-vendor-prefixes))

## CSS-in-JS의 단점

- 스타일은 JavaScript에 정의되어 있으므로 JavaScript가 비활성화 된 경우 스타일에 영향을 줄 수 있음.
- 스타일은 라이브러리에서 한번, 스타일이 삽입될 때 브라우저에서 한번 이중 구문 분석(parsing)된다.
- 전통적으로 웹페이지를 로드할 때 브라우저는 CSS를 읽고 적용한다. CSS-in-JS를 사용하면 브라우저는 CSS 스타일 태그를 동적으로 생성한 다음 이를 읽고 웹 페이지에 적용한다. 이를 읽고 생성하면 성능 시간이 소요된다.

<br/>

## 런타임이 없는(zero-runtime) CSS-in-JS 사용

이중 파싱으로 인한 성능 손실을 개선하는 솔루션 중 하나는 라이브러리가 먼저 CSS-in-JS 블록을 별도의 `.css` 파일로 변환해야 한다. 그런 다음 브라우저는 해당 스타일을 읽고 웹페이지에 적용해 궁극적으로 스타일 태그를 생성하는 동안 일반적으로 낭비되는 런타임을 절약한다. 이를 zero-runtime CSS-in-JS 라고 함. 성능이 중요한 규모가 크거나 복잡한 프로젝트에 특히 유용하다.

> zero-runtime이란, CSS-in-JS 구문으로 스타일을 작성하지만, 다른 전처리기와 마찬가지로 CSS가 생성된다는 의미다.
>
> [출처](https://css-tricks.com/the-unseen-performance-costs-of-modern-css-in-js-libraries/)

몇 가지 라이브러리가 있다.

### linaria

#### 장점

- TypeScript로 작성 되었음.
- 7.1k 이상의 GitHub star, 260개의 GitHub fork 존재.
- 거의 모든 프레임워크와 호환
- Linaria는 프로덕션용 빌드를 생성하는 동안 CSS-in-JS를 별도의 `.css` 파일로 변환한다.
  - 따라서 css 파일로 추출되므로, 사용하지 않은 스타일을 자동으로 제거할 수 있고, CSS 파일은 JS 파일과 다른 주기로 변경될 수 있기 때문에 캐싱에 유용하다.

- 📌 개인적으로 마음에 드는 부분은 sass 문법과 같이 nested 문법을 지원한다는 점임.
- GitHub 등의 문서가 매우 친절함([동작 원리](https://github.com/callstack/linaria/blob/master/docs/BENEFITS.md), [사용 사례](https://medium.com/airbnb-engineering/airbnbs-trip-to-linaria-dc169230bd12) 등)

#### 단점

- 구현의 어려움: Linaria를 올바르게 구현하려면 Babel 설정해야 하므로 혼란스러울 수 있다.
- 번들러 설정 : JS 파일에서 CSS를 추출하려면 Rollup 또는 Webpack과 같은 번들러를 사용해야 하며 설정이 어려울 수 있다.
- 플러그인 지원 : Linaria는 Rust에 대한 고품질 플러그인을 지원하지 않는다.

### vanilla-extract

vanilla-extract는 [treat](https://github.com/seek-oss/treat) 라이브러리에서 파생되었다. 즉 treat는 vanilla-extract로 대체되었다. ([출처](https://github.com/andreipfeiffer/css-in-js#treat)) Air-bnb에서 zero-runtime을 고를 때 검토했던 라이브러리로 Emotion, linaria, treat가 있을 정도로 고려되었음. 결국 linaria를 선택했지만.

[linaria vs vanilla-extract](https://github.com/silviogutierrez/reactivated/discussions/139)

#### 장점

- TypeScript로 정적 분석의 도움을 받을 수 있다. 정의되어 있는 타입으로 타입 추론이 가능하며 자동완성 기능 활용도가 높다.
- 복붙 시 css-converter VSCode 플러그인으로 변환 가능.

#### 단점

- 제약이 많음.
  - 코로케이션을 지원하지 않는다. 즉, `.css.ts` 파일을 새로 생성해야 한다. (스타일 공동 배치 불가능)
  - className에 들어갈 변수 명을 지정해주어야 한다.(인라인 스타일 정의가 불가능하다.) ([링크](https://github.com/vanilla-extract-css/vanilla-extract/discussions/500))
    - esbuild와 같은 기능을 사용하기 위해서는 AST 조작을 사용하면 안되는데, 변수 명을 지정해주지 않으면 AST 조작을 해야 하기 때문에 변수 명 지정이 필수적이다.
  - nested 문법을 사용할 수 없음.

<br/>

> 추가 도움
>
> - [성능 분석 CSS-in-JS](https://css-tricks.com/a-thorough-analysis-of-css-in-js/)
> - [emotion으로 파악해보는 css-in-js 이모저모](https://ideveloper2.dev/blog/2022-01-25--emotion%EC%9C%BC%EB%A1%9C-%ED%8C%8C%EC%95%85%ED%95%B4%EB%B3%B4%EB%8A%94-css-in-js%EC%9D%98-%EC%9D%B4%EB%AA%A8%EC%A0%80%EB%AA%A8/)
