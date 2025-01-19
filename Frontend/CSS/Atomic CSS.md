# Atomic CSS

> [출처](https://tech.kakao.com/2022/05/20/on-demand-atomic-css-library/)

최근 주목 받고 있는 TailwindCSS. 혹은 Utility-first CSS, Atomic CSS, Functional CSS라는 CSS 방법론이 있다.

Bootstrap, Sass, BEM 혹은 CSS-in-JS 와는 달리 비주류이고, TailwindCSS를 통해 조금씩 알려지고 있는 AtomicCSS가 있다.

<br/>

## Atomic CSS를 이해하기 위한 CSS의 역사 이야기

### HTML 탄생 1990

문서를 공유하기 위해 만들어짐. 콘텐츠에 의미를 부여하는 태그를 붙여주어 서식을 꾸미는 방식으로 만들어짐.

### Inline-style

조금 더 다른 형태의 서식으로 꾸미기를 원했고 이를 위해 style이라는 것이 추가 되었다. 태그에 직접 style을 지정하는 방식을 line-style이라고 한다.

문제점은 중복 서식을 표현할 때 코드가 너무 비대해지면서 가독성 저하, 유지 보수 어려워짐.

![image](https://github.com/pozafly/TIL/assets/59427983/f557da4b-d9ac-4917-be1f-a9b835134162)

따라서 inline-style은 나쁘다는 원칙이 만들어짐.

<br/>

## CSS 탄생 1996

inline-style 중복을 제거하기 위한 방법. inline-style을 별도의 Rule로 선언, 서식이 필요한 곳을 선택해 반복 적용하는 방식으로 만들어짐.

```html
<style>
strong { color: red; text-decoration: underline }
</style>


<strong>This is Red Underline Title</strong>
.... <strong>Other Red Unerline</strong>
... <strong>Underline Title 333<strong>
```

<br/>

## 콘텐츠와 서식 분리

콘텐츠와 스타일을 분리하고 보니 콘텐츠와 서식을 1:1 매칭하지 않고 하나의 콘텐츠에 여러 가지 스타일을 선택적으로 적용할 수 있다는 것을 알게된다.

```html
<strong>This is Red Underline Title</strong>
.... <strong>Other Red Unerline</strong>
... <strong>Underline Title 333<strong>
```

```css
/* 동일한 콘텐츠로 이러한 서식을 적용하거나 */
/* style1.css */
strong { color: red; text-decoration: underline }
```

```css
/* 저러한 서식을 선택적으로 사용할 수 있다. */
/* style2.css */
strong { color: blue; font-weight:bold }
</style>
```

즉, `.css` 파일을 나누어 적용할 수 있다. 관심사가 분리된 것.

콘텐츠가 수정되면 HTML을 수정하고, 서식이 변경되면 CSS를 수정하자.

<br/>

## Selector가 복잡해진다. Sass의 등장, 2006

스타일을 하나하나 원하는 엘리먼트를 선택, 스타일을 적용하는 것이 쉬운일은 아니다. 복잡한 Selector를 써야 했고, Selector도 세분화가 필요했다.

![image](https://github.com/pozafly/TIL/assets/59427983/5017e36f-951c-4711-914a-8a7ed9103474)

Sass의 등장.

<br/>

## Ajax의 등장과 HTML 편집권의 변화

Ajax가 보편화 되면서 백엔드에서 HTML을 제외한 데이터만 전달할 수 있게 되면서 이제는 HTML 편집권은 백엔드가 아니라 프론트엔드로 내려오게 된다.

HTML, CSS를 동시에 편집할 수 있게 되면서 복잡하게 Selector를 쓸 필요가 없다는 것을 깨닫게 된다. 이제는 HTML을 수정할 수 있으니 CSS를 복잡하게 하는 것보다 HTML을 복잡하게 쓰는 것이 더 낫다는 것을 알게 된다.

> CSS에서 복잡한 Selector를 잘 쓰기 보다는 HTML에서 미리 잘 지어둔 class 이름이 더 좋다.

이후에는 어떻게 Selector를 잘 쓰는지보다 어떻게 class이름을 더 잘 지을 수 있는지 고민하는 식으로 변하게 된다.

### 의미 있는 이름 vs 시각적인 이름

복잡한 Selector 보다는 단순 Class Selector를 사용하는 것이 보편화 되면서 어떻게 이름을 잘 지을지 고민하게 됨.

2가지 방식. 의미를 기반으로 하는 `.error`, `.title`과 같은 이름이나, 기능이나 시각적 요소를 기반으로 하는 `.red`, `.bold` 와 같은 방식이다.

> *어떻게 이름을 짓는 것이 좋을까?*
>
> *`.error` vs `.red`*
> *`.description` vs `.text-right`*
> *`.title` vs `.bold`*

스타일 변경 시 CSS를 수정한다는 원칙에 의거, 디자인에 맞게 CSS를 변경하다 보니 시각적 이름으로 짓게 될 경우 `.red { color: orange }` 라는 이상한 형태가 만들어진다. 따라서, 유지 보수와 유연한 대응을 위해 의미 기반으로 짓는 것이 좋다.

즉, 시각적인 아름다움보다 시멘틱한 이름이 좋다.

<br/>

## 웹 문서가 아니라 웹 어플리케이션 시대

구글 지도나 구글 오피스 같이 웹으로도 어플리케이션을 만들게 되고 CMS와 같은 데이터를 다루는 백오피스 등의 웹 페이지의 요구사항이 커지면서 웹은 점차 어플리케이션의 형태로 발전.

CSS는 웹 문서를 잘 꾸미기 위해 설계되었다. 그래서 초기에 문서를 만들기 위해 설계된 CSS는 웹 어플리케이션 개발 패러다임에서 과도기를 맞이하게 된다.

### CSS가 문제가 생기기 시작했다

1. HTML을 복 붙 했는데 CSS가 제대로 적용안됨.
2. CSS를 수정했더니 엉뚱한 곳이 틀어짐.
3. Selector가 안먹어!important로 덮어써야 하는 문제.
4. 안 쓰이는 코드 찾기가 진짜 어려운 문제 (특히 JS와 결합된 class)
5. 도움지 CSS가 재사용되지 않는 문제

### 어플리케이션에서 CSS 사용시 결정적 2가지 문제점

JS는 Component와 Frammework 기반 개발방식으로 변해가고 있었음. 당시 flexbox와 같은 어플리케이션을 위한 레이아웃 스펙은 존재하지 않았기에 float 등을 억지로 사용하고 문서 간격을 만들기 위한 margin을 통해 레이아웃을 하는 스펙의 부재도 있었지만, 결정적으로 큰 2가지 설계상 문제 존재.

1. Global Scope: 전역 변수
2. Specificity: 작성된 순서대로 적용되지 않음.
   - HTML에 지정된 class 순서가 아니라, CSS의 순서에 의해 서식 우선순위가 결정된다.
   - Selector가 복잡할 수록 서식이 나중에 적용된다.
   - 다른 서식을 덮어쓰기 위해서는 Selector를 더 복잡하게 작성해야 한다.
   - 간단한 Selector로 덮어쓰려면 `!important`를 해야 함.
   - `!important` 한 속성을 덮어쓰려면 복잡한 Selector에 `!important` 해야 한다.

<br/>

## 방법론 (BEM) 2013

처음부터 CSS를 작성하는 규칙을 잘 짜면 문제 해결 가능하다는 것이 CSS 방법론이다. SMACSS, OOCSS, BEM, ITCSS, ATOMIC CSS 등

![image](https://github.com/pozafly/TIL/assets/59427983/934886d6-3772-4b50-b1ae-a82027f8c94f)

BEM이 이겼음. 이런 방법론으로 복잡성을 줄이거나 생산성을 가져오는 것은 아니었고, CSS의 본질적 문제를 해결해주지 않았다.

어쩌면 CSS로 태생적으로 해결할 수 없는게 아닐까? 그러면 JS의 도움을 받아보자. 라는 생각으로 CSS 자체보다는 JS의 도움을 받는 식으로 해결 하고자 하는 움직임.

<br/>

## CSS Modules, 2015

Global Scope 문제의 해결책.

![image](https://github.com/pozafly/TIL/assets/59427983/2eb307a5-e51e-4545-abd4-e552ae2c50b1)

CSS 문제인 Global Scope를 막기 위해 Component 단위에서 사용하는 CSS에 hash를 추가해 CSS가 더이상 Global 하지 않도록 하는 방식을 통해 해결하고자 하는 방법이 만들어짐.

<br/>

## CSS-in-JS

React가 대세가 되면서 React에서 Style을 다루는 불편함을 해소하기 위해 JS에서 HTML을 쓰기 위해 JSX를 만든 것 처럼 JS에서 Style을 쓰기 위한 방법으로 CSS-in-JS 라는 방법이 등장한다.

### CSS-in-JS의 시작(Vjeux), 2014

CSS는 Global Scope, Specificity 말고도 사실 문제가 많다. JS에서 CSS를 해보면 어떨까?

![image](https://github.com/pozafly/TIL/assets/59427983/203638a8-10c0-4341-b216-2326bfc9caae)

### CSS-in-JS Styled-Components, 2016

![image](https://github.com/pozafly/TIL/assets/59427983/35d418ac-b024-42b8-95cc-9346643e990f)

Styled-Components 탄생.

![image](https://github.com/pozafly/TIL/assets/59427983/aab9c704-d068-42d6-86fd-da37df1ce506)

<br/>

## 발전 정리

> *복잡한 Selector → 간단한 Selector → No Selector*
> *Global Scope → Local Scope*
> *HTML과 CSS의 분리 → 컴포넌트로 결합*

(간단한 Selector는 Sass임.)

<br/>

## TailwindCSS 2017

> *"Best practices" don't actually work.*
> ***Utility-First CSS Framework***

지금까지의 Best practices는 사실 **제대로 동작하지 않는다**며 만들어진 새로운 시각의 CSS Framework가 Tailwind CSS다.

하지만, 많은 Hater를 가지고 있음.

> Utility-First, Atomic CSS, Function CSS는 같은 것을 의미함. TailwindCSS는 Utility-First를 밀고 있지만, Atomic CSS라고 하자.

<br/>

## Atomic CSS

> 위의 원칙들
>
> 1. inline-style은 나쁘다.
> 2. 콘텐츠와 서식을 분리.
> 3. class 이름은 시멘틱한 게 좋다.

아직도 위 원칙이 유효한가?

CSS가 만들어지고, `.bold`. `.hidden`, `.text-right`, `.mt10`, `.red` 과 같은 기능 단위 class 이름을 만들어 분명히 편리하게 사용했던 경험이 있음.

이 방식이 편리한 방식이라면 일부만 하는게 아니라 전부 다 이런 방식으로 만들어 보는 것은 어떤가?

![image](https://github.com/pozafly/TIL/assets/59427983/bee031ff-e25a-4b59-aa3b-2fb946158852)

Atomic CSS. 의미 기반이 아닌 시각적 기능에 기반한 이름을 지은 변하지 않는 단일 클래스를 이용하자. 변하지 않는 CSS를 만들어두고 HTML에서 필요한 스타일을 적용하는 방식으로 개발하는 것이 Atomic CSS다.

### 왜 이렇게 작성하는가?

![image](https://github.com/pozafly/TIL/assets/59427983/5ed6e3b7-75bd-4054-97ca-d628ab2e38b4)

의미론적 이름 짓기가 어렵기 때문임.

```
.inner-main` `.wrap_cont` `.cp_more_wrap` `.inner_item` `.wrap_title
```

와 같은 길어지면.

### 의미론적 이름 짓기의 한계

다른 프ㅡ로젝트 내에서 중복해 사용할 수 없기 때문에 새로운 이름을 지을 때면 항상 기존에 만든 CSS를 피해가야 함.

### Traditional CSS vs Atomic CSS

#### Traditional CSS

이름 짓기 어려운 CSS 만들어야 함. 만들어 놓은 CSS를 피해야 하기 때문에 복잡해져 파일 크기가 점점 커지지만, 재사용 불가능. 매번 계속 작성해야 함.

#### Atomic CSS

시각적 기능 기반 쉬운 이름으ㅡㄹ 가지며 미리 만들어두기 때문에 외워두면 계속 재사용이 가능, 일정 크기 이상 파일이 커지지 않는다.

![image](https://github.com/pozafly/TIL/assets/59427983/1d61cfa0-42cd-4d83-8c04-5d057584aa2c)

무엇보다 필요없는 이름 짓기에서 해방된다.

누구나 동일 형태의 코드 스타일을 작성.

- 협업시 컨벤션을 맞추기 위한 노력 줄어듦.
- 컨벤션과 방법론의 역할을 한다.

<br/>

## 비주류

주류가 되지 못함. TailwindCSS는 왜 Atomic CSS가 아니라 Utility-First라고 이름 지었을까?

AtomicCSS는 Tailwind CSS가 시초가 아님. TailwindCSS가 유명해진 것은 커뮤티니를 통해 인식 전환을 위한 노력을 해왔다. 지금도 여전히 안티는 많지만.

### AtomicCSS를 싫어하는 이유

- HTML에 스타일 코드가 반복되는 것 해결 안됨.
- 의미론적 구분 없으면 전체적 콘텐츠 파악 어려움.
- 콘텐츠와 서식의 분리를 할 수 없는데, 다양한 테마 서식 적용은 어떻게?
- HTML이 너무 지저분해지고 코드 가독성 떨어짐.
- HTML기반의 문서에 전부 AtomicCSS 사용해야 함?
- CSS를 쓰지 않으면 CSS를 모르게 된다.
- CSS가 문제 있는 것은 맞는데, CSS 잘하면 되는거아님?

### Atomic CSS 써도 되는 이유

WebFramework를 사용한 Component 기반 시대인 지금, Web Framework의 컴포넌트가 시멘틱과 중복 방지 역할을 대신 해주고 있음.

하나의 콘텐츠에 여러 가지 디자인을 할 필요성이 줄어들었다. 디자인이 곧 아이덴티티인 시대이기 때문에 더 이상 콘텐츠와 서식을 분리할 필요 없음. 시멘틱 하지 않아도 된다.
