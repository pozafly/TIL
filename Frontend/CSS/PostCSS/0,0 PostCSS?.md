# PostCSS

css 전처리기 중 하나이다. 

- 전처리기를 사용하는 이유
  - 기본적인 CSS 만으로 중복적 작성 & 브라우저 호환성 맞춰 주어야 함.
- PostCSS와 다른 전처리기와 다른점
  - LESS, SASS 등 다른 전처리기는 문법과 프레임워크가 따로 있다. 이 녀석들은 css로 변환 되어 표현됨.
  - 이녀석들이 제공하는 것만 맞춰서 쓸 수 밖에 없다.
  - PostCSS 는 플러그인이 많아 선택해서 사용할 수 있음.
  - create react-app 에서도 PostCSS를 채택하여 사용하고 있음.

<br/>

## 장점

- 플러그인 제공

- 브라우저 호환성

  ```css
  :fullscreen {}
  ```

  PostCSS 에서 위에 녀석을 사용하면,

  ```css
  :-webkit-full-screen {}   // 사파리
  :-ms-fullscreen {}        // 익스플로러
  :fullscreen {}
  ```

  일반 css, scss 에서 prefix로 사용하고 있는 이런 브라우저 호환성에 대해 걱정할 필요가 없다.

- 최신적 문법 적용.(컬러나 그리드, 사용할 수 있게 해줌.)

- 모듈화 가능(_ 로 이어지는 class 명을 길게 쓰지 않아도 됨.)

  ```css
  // button1.css
  .button {
    background-color: aquamarine;
  }
  
  // button2.css
  .button {
    background-color: sandybrown;
  }
  ```

  만약 이렇게 한 프로젝트 내 다른 css 파일에 동일한 선택자로 css를 사용한다면 밑에 있는 녀석이 최 우선이기 때문에 sandybrown이 적용되는걸 볼 수 있다. 따라서 button1__button 이런식으로 class를 나누어서 사용하곤 했다.

<br/>

## 모듈화 방법

button1.css 확장자 명을, `button1.module.css` 로 바꾸어 준다.

```jsx
import styles from './button1.module.css';
...
<div className={styles.button}>
  <span className={styles.text}></span>
</div>
```

이렇게 하면 모듈화 된 채로 가져와진다. 브라우저에서 element를 확인해보면,

```html
<span class="button1_button__1VGtG"></span>
```

이렇게 이름이 다른 컴포넌트에서 동일한 이름을 썼는지 안썼는지 확인하지 않아도 알아서 PostCSS가 해줌.



<br/>

[PostCSS](https://postcss.org/)

[PostCSS Plugins](https://www.postcss.parts/)

[Plugins Github 페이지](https://github.com/postcss/postcss/blob/master/docs/plugins.md)