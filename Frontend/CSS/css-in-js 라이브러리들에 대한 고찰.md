# Css-in-js 라이브러리들에 대한 고찰

> [출처](https://bepyan.github.io/blog/2022/css-in-js)

## 들어가면서

- 다양한 CSS-in-JS 라이브러리가 있는데 이들은 어떤 차이점이 있을까?
- 더 나아가 어떤 상황에서 어떤 라이브러리를 사용하면 좋을까?

### CSS-inJS가 대세인 이유

- 중복되지 않는 class 이름을 고려할 필요가 없다.
- JS 코드와 CSS가 상태 값을 공유할 수 있다.
- 컴포넌트와 스타일 코드를 쉽게 오갈 수 있다.
- 자동으로 [vendor-prefix](https://developer.mozilla.org/en-US/docs/Glossary/Vendor_Prefix)을 붙여준다.

개발 친화적 DX다.

<br/>

2020, 2021에 들어서서 많은 CSS-in-JS 라이브러리 등장함.

- https://2021.stateofcss.com/en-US/technologies/css-in-js
- https://risingstars.js.org/2021/en#section-css-in-js

css-in-js 동작 방식은 크게 **runtime**, **zero-runtime** 으로 나뉜다. runtime이 반드시 성능 저하를 발생시키진 않고 프로젝트 규모와 상황에 따라 달라질 수 있음을 염두하고 살펴보도록 하자.

<br/>

## Runtime

### JavaScript runtime에서 필요한 CSS를 동적으로 만들어 적용한다

대표적으로 알려진 styled-component, emotion이 있다.

- 개발 모드에서는 `<style>` 태그에 style을 추가하는 방식을 사용한다.
  - 디버깅에 이점이 있다고 한다.
- 배포 모드에서는 styleSheet를 [CSSStylesSheet.insertRule](https://developer.mozilla.org/en-US/docs/Web/API/CSSStyleSheet/insertRule) 통해 바로[CSSOM](https://dkmqflx.github.io/frontend/2020/09/14/jscssom/)에 주입한다.
  - 성능상의 이점이 있다고 한다.

### Css-loader가 필요 없다

css 파일을 생성하지 않기에 webpack에서 css-loader가 필요 없다.

### 런타임 오버헤드가 발생할 수 있다

- 런타임에서 동적으로 스타일을 생성하기에 스타일이 수시로 변경된다면 오버헤드 발생.
  - ex) 스크롤, 드래그 앤 드랍 관련 복잡한 에니메이션

<br/>

## Zero-runtime

런타임에 css를 생성하지 않으면서 **페이지를 더 빨리 로드할 수 있다**. JS 번들에서 styles 코드를 모두 실행 되어야 페이지가 로드된다.

![image](https://github.com/pozafly/TIL/assets/59427983/61fb9f45-7cfa-4083-abe3-2525e675b8fd)

runtime에서 스타일이 생성되지 않는다. props 변화에 따른 동적 스타일은 css 변수를 통해 적용한다.

빌드 타임에 css를 생성해야하기에 webpack 설정을 해야 한다.

- React CRA를 사용한다면 eject 해서 webpack 설정해야하는데 굉장히 번거롭다.
- runtime에서 css polyfill을 사용할 수 없어 브라우저 버전 이슈가 있을 수 있다.

### 첫 load는 빠르지만, 첫 paint는 느릴 수 있다

![image](https://github.com/pozafly/TIL/assets/59427983/abfe3a5c-34c1-44e6-8397-5265955163ee)

css styles까지 모두 로드 되어야 첫 paint를 시작한다. 반면 runtime에서는 style을 로드하면서 첫 paint를 시작할 수 있다.(로딩 화면)

대표적 라이브러리

- linaria
  - styled-component 문법 그대로 사용해 러닝커브가 없을 것 같다.
  - [styled-components와 속도 비교](https://pustelto.com/blog/css-vs-css-in-js-perf/)
  - mini-css-extract-plugin에 의해 critical css를 판단할 수 없는 경우 linaria의 collect를 사용가능하다.
