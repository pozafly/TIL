# className, htmlFor



## 1. JSX에서 class가 아니라, className을 사용한다.

**react에서는 jsx element에 class를 할당할 때 왜 className을 사용할까?**

react에서 className을 사용하는 이유는, JavaScript의 class 예약어와 겹치기 때문에 className을 사용한다. 따라서 JavaScript의 class 예약어와 충돌을 피하기 위해 사용한 것인데, Vue의 템플릿 코드에서는 class를 사용할 수 있다.

Vue 컴포넌트 템플릿에서 class를 직접 사용하는 경우는 JavaScript class 예약어와 충돌하지 않는다. Vue는 템플릿을 파싱하고, Virtual DOM을 생성한다. 'class' 속성을 감지하면, 해당 요소에 클래스를 바인딩하는 코드를 생성한다.

```vue
<template>
  <div class="container">
    <p class="text">Hello, Vue!</p>
  </div>
</template>
```

 위 템플릿 코드는 아래와 같이 변환된다.

```js
render() {
  return this._c(
    'div',
    {
      staticClass: 'container'
    },
    [
      this._c(
        'p',
        {
          staticClass: 'text'
        },
        'Hello, Vue!'
      )
    ]
  );
}
```

변환된 JavaScript 코드에서 `staticClass` 라는 속성이 사용되는데 이는, class 속성을 나타내는 것이다.

react에서는 JSX 코드는 react 컴파일러에 의해 JavaScript 코드로 변환되고, className은 일반적인 JavaScript 객체의 속성으로 처리된다.

```jsx
<div className="container">
  <p className="text">Hello, React!</p>
</div>
```

아래와 같이 변환됨

```js
React.createElement(
  'div',
  { className: 'container' },
  React.createElement(
    'p',
    { className: 'text' },
    'Hello, React!'
  )
);
```

<br/>

## 2. label에서 for 어트리뷰트가 아니라, htmlFor을 사용해야 한다.

마찬가지로, JavaScript의 `for` 예약어와 충돌하기 때문이다.

<br/>

이걸 봤을 때, react는 최대한 JavaScript 예약어를 헤치지 않도록 방향성을 잡은 것 같다. 이유는, vue에서도 마찬가지로 코드를 파싱후 class or for 구문을 사용하는데 예약어와 관계없이 자유롭게 사용할 수 있다. 왜냐면 실행될 코드가 아니라 string 형태로 object에 박히기 때문에.

react도 마찬가지로 파싱된 후  `staticClass` 라는 것으로 변환되는데, 이는 객체의 key 값으로 들어가기 때문에 JavaScript 예약어와 상관 없이 사용할 수 있기 때문에 충돌을 피할 수 있다.

그런 이유로 react는 의도적으로 예약어를 피하려고 다른 이름을 선택했다고 추론할 수 있다.