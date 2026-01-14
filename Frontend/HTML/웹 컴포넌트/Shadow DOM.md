# Shadow DOM

> [Toast UI](https://ui.toast.com/posts/ko_20170721)

Shadow DOM에서 배워야 하는 API는 `element.attachShadow()` 함수 하나 뿐임.

<br/>

## 분리: #shadow-root

HTML, CSS에 스코프를 줄 수 었다.

```js
// 어디까지 녹색으로 보이려나?
document.body.appendChild(document.createElement('span')).innerHTML
  = '<style>div { background-color: #82b74b; }</style><div>야호!</div>';
```

그러면 모든 div는 녹색으로 변경된다. 글로벌이기 때문이다.

```js
// 이번에야 말로 `야호!`만 녹색으로!
document.body.appendChild(document.createElement('span'))
  .attachShadow({mode: 'open'})
  .innerHTML = '<style>div { background-color: #82b74b; }</style><div>야호!</div>';
```

이번엔 `attachShadow({ mode: 'open' });` 을 넣어주었다.

![[assets/images/72daa70d42f2c2ff8ad9560d021db151_MD5.png]]

개발자 도구에 `#shadow-root` 라는게 생겼고, 그 밑에 있는 `style` 은 밖으로 새어나가지 않는다. 반대로 글로벌에 존재하는 스타일 역시 `#shadow-root (open)` 안에 있는 엘리먼트에는 영향을 주지 않는다.

```js
document.body.appendChild(document.createElement('span')).innerHTML
  = '<style>div { background-color: #82b74b; }</style><div id="non-shadow">야호!</div>';
document.body.appendChild(document.createElement('span'))
  .attachShadow({mode: 'open'})
  .innerHTML = '<div id="shadow">야호!</div>';
```

이렇게 넣어주면 가장 위에 있는 shadow-root style에 있는 div 녹색은 아래의 shadow-root에 영향을 주지 않는다.

![[assets/images/d69b0a9d9a6fabed7a836588433b3c42_MD5.png]]

글로벌 스타일이 적용되지 않는것이다. 쉐도우 돔은 돔 자체의 분리 역할을 함. 루트를 기준으로 `id` 를 중복으로 써도 되고, 루트 안팎의 동일 이름의 `class` 역시 전혀 다른 클래스의 역할을 수행한다. 쉐도우 루트 밖에서 쉐도우 돔의 엘리먼트를 셀렉트할 수도 없음.

하나의 HTML에 수천 개의 엘리먼트의 스타일을 한 번에 모두 관리하기 위해 `class` 이름을 고민할 필요도, `id` 중복이 무서워 쓰지 못하는 경우도 없다. 큰 기능 없는 페이지에도 스타일 수천 줄은 우습게 나온다. 과연 잘 관리하고 있는 걸까?

<br/>

## 조합: `<slot>`

쉐도우 돔을 사용하지 않더라도 `iframe` 을 사용하면 비슷하게 수행할 수 있음. 하지만 iframe을 사용한 DOM 분리는 다음과 같은 단점이 있음.

- http 요청이 한 번 더 일어난다.
- 별도 페이지이기 때문에 소비되는 리소스도 높고 느리다.
- iframe 주소가 같은 도메인이 아닌 경우 접근 불가능하다.

이와 같인 이유로 트위터는 `iframe` 형식으로 지원하는 기능을 브라우저가 지원하는 경우 쉐도우 돔 형식으로 전환함.

> What does this change mean for you? Much lower memory utilization in the browser, and much faster render times. Tweets will appear faster and pages will scroll more smoothly, even when displaying multiple Tweets on the same page. - *[Upcoming Change to Embedded Tweet Display on Web](https://twittercommunity.com/t/upcoming-change-to-embedded-tweet-display-on-web/66215)*

iframe으로 절대 할 수 없지만, 쉐도우 돔이 할 수 있는 일로 **슬롯 조합**이 있다. **슬롯**은 HTML 조합을 지어 나타나 특별한 기능을 수행하는 경우에서 발견할 수 있다. `ol` + `li`, `select` + `item`, `form` + `input` 등이 그 예다.

우리가 쉐도우 돔과 함께 사용할 슬롯역시 위와 동일한 개념이다. `ol` 은 `li` 자식 노드에 숫자를 부여한다거나 하는 식이다. 특별한 마크업을 부여할 수도 있고, 스타일을 달리하거나, 동작을 수행하는 기능을 부여할 수도 있다. 아래 예제는 자식 노드(라이트 돔)을 블럭 요소로 감싸는 일을 하는 셈이다.

개발자 도구에서 편집하여 사용하거나, 작은 HTML 파일을 만들어 테스트해 볼 수 있다. [whatwg - 쉐도우 트리](https://dom.spec.whatwg.org/#shadow-trees)

- **쉐도우 돔**: 아래의 코드에서 h1, p 등 **쉐도우 루트**에 붙어있는 DOM
- **쉐도우 루트**: `#shadow-root`
- **쉐도우 호스트**: 쉐도우 루트의 부모. 아래의 코드에서 `div#slot-test`
- **라이트 돔**: 도큐먼트의 쉐도우 호스트에 붙어있는 노드들. `span`

아래 코드를 정의로 풀이해보면 '쉐도우 돔의 슬롯이 가진 이름에 맞는 라이트 돔의 노드가 각 슬롯에 삽입된다' 라고 ㅅ할 수 있겠다.

```html
<body>
...
<div id="slot-test">
  <!-- Light DOM -->
  <span slot="title">Hello</span>
  <span slot="desc">world</span>
</div>
...
</body>
```

```js
// Shadow DOM
document.querySelector('#slot-test')
  .attachShadow({mode: 'open'})
  .innerHTML = `
  <h1>
    <slot name="title"></slot>
  </h1>
  <p>
    <slot name="desc"></slot>
  </p>
  `;
```

![[assets/images/b9405affe413c695ef5664f14490b2c9_MD5.png]]

슬롯의 이름에 맞는 라이트 돔이 자리를 찾아간다.

<br/>

## 컴포넌트: 커스텀 엘리먼트 + 쉐도우 돔 = DOM OOP

쉐도우 돔을 사용하지 않고 돔의 분리를 할 수 있는 방법으로 `iframe` 을 사용할 수 있다. 실제 쉐도움 돔 `mode: close` 의 polyfill은 `iframe` 으로 작성되었다. 위에서 작성했던 쉐도우 돔을 예제로 아래처럼 쉐도우 돔의 엘리먼트에 외부에서 접근해보자.

```js
document.querySelector('#non-shadow'); // <div id="non-shadow">야호!</div>
document.querySelector('#shadow'); // null
```

알아본 바와 같이 쉐도우 돔에 있는 노드는 `id` 를 통해 가져올 수 없음. 쉐도우 돔에 존재하는 엘리먼트를 쉐도우 돔 밖에서 얻어오기 위해서는 아래와 같이 조금 더 복잡한 방법을 통해 가능하다.

```js
document.querySelector('span').shadowRoot.querySelector('#shadow'); // <div id="shadow">야호!</div>
```

오히려 이것은 좋은 일임. 커스텀 엘리먼트는 HTML 엘리먼트를 js 오브젝트로서 관리할 수 있도록 해준다.

```js
// 자바스크립트와 HTML 엘리먼트를 한몸으로 만들어 준다
class MyElement extends HTMLElement {
  yey() {
    console.log('yey');
  }
}
document.querySelector('my-element').yey() // 'yey'
```

그럼 이번에 커스텀 엘리먼트와 쉐도우 돔을 묶은 코드를 보자.

```js
class MyElement extends HTMLElement {
    static get observedAttributes() {return ['lang']; }

    constructor() {
      super();

      // add shadow root in constructor
      const shadowRoot = this.attachShadow({mode: 'open'});
      shadowRoot.innerHTML = `
        <style>div { background-color: #82b74b; }</style>
        <div>yey</div>
      `;
      this._yey = shadowRoot.querySelector('div');
    }

    attributeChangedCallback(attr, oldValue, newValue) {
      if (attr == 'lang') {
        let yey;
        switch (newValue) {
          case 'ko':
            yey = '만세!';
          break;
          case 'es':
            yey = 'hoora!';
          break;
          case 'jp';
            yey = '万歳!';
          break;
          default:
            yey = 'yey!';
        }

        this._yey.innerText = yey;
      }
    }

    yell() {
      alert(this._yey.innerText);
    }
  }

  window.customElements.define('my-element', MyElement);
```

아래 그림과 위 코드를 서로 대입해보기로 하자. 아래 그림은 커스텀 엘리먼트, 그리고 쉐도우 돔을 OOP 오브젝트 개념도와 비슷하게 그려놓은 것임. 커스텀 엘리먼트는 HTML 엘리먼트를 확장해 오브젝트로 만들어주고, 쉐도우 돔은 그 오브젝트에 스코프를 제공해준다. 다시 이야기하면 **커스텀 엘리먼트와 쉐도우 돔은 DOM을 OOP의 대상으로 바라볼 수 있게 해준다**.

![[assets/images/d2b7e733c8161272b33e702fe03e96b9_MD5.png]]

커스텀 엘리먼트가 가지고 있는 쉐도우 돔 트리의 엘리먼트들은 OOP 내부 구현에 해당함. 외부에서 어떤 오브젝트의 private 속성을 변경하고 싶다면 그것은 어떤 상황인가? private을 수정한다는 시도가 먼저 잘못되었고, 오브젝트의 정체성에 맞게 필요하다면 메서드를 새로 추가해야 한다.

웹 컴포넌트(커스텀 엘리먼트 + 쉐도우 돔)에서도 마찬가지다. 내부 돔을 직접 수정하려 하는 시도가 잘못된 것이고, 웹 컴포넌트의 정체성에 맞게 필요하다면 메서드를 추가해야 한다. 우리가 알아보고 있는 기술은 **웹 컴포넌트**다. 그 자체로 독립적이고 완결성 있는 것이어야 한다는 뜻이다.

위에서 언급한 불편한 쿼리문(querySelector)은 우리가 기다리는 JS private 속성 만큼이나 반갑고 유용한 존재다. 추후 직접 프로젝트를 시작할 때, 아래의 나쁜 예처럼 셀렉터를 작성하고 있다면 위의 그림을 다시 떠올리기 바란다.

### 예제 1

```html
<!-- GOOD! DOM OOP! -->
<my-element lang="ko"></my-element>
// BAD IDEA!
  document.querySelector('my-element')
    .shadowRoot
    .querySelector('div')
    .innerText = '만세!'
```

### 예제 2

```js
  // GOOD! DOM OOP!
  const myElement = document.querySelector('my-element');
  myElement.yell();
  // BAD IDEA!
  const yey = document.querySelector('my-element')
    .shadowRoot
    .querySelector('div')
    .innerText;
  alert(yey);
```

### DOM을 인터페이스로

구글 폴리머 팀의 Rob Dodson은 위 그림에서 DOM에 해당하는 **events**, **attributes** 로 인터페이스를 구성하라고 권고함. 엘리먼트는 결국 어떤 상태를 나타내고 있기에, method를 실행하는 것보다 attributes 값을 할당하는 것이 맞다는 말임. 그 다음, 커스텀 엘리먼트는 attributes 값이 변경될 때, 그에 맞는 동작을 수행하면 된다. 또한 중요한 attributes 값이나 상태가 있다면 그것은 **events** 로 내보내, 필요한 곳에서 수행해야 한다고 제안함.

좋은 방법이다. 하지만, attributes 값은 문자열, 존재 유무로 boolean 값만 처리할 수 밖에 없다. 실제 컴포넌트를 구성할 때 얼만큼 효용이 있을지는 모르겠다. 위에서 말한 바와 같이 DOM 스코프만 잘 지키면 충분히 좋은 컴포넌트 인터페이스가 만들어질 것이라 생각함.

<br/>

## 쉐도우 돔 자세한 내용들

- `textarea`, `input`, `image` 와 같은 엘리먼트들은 쉐도우 돔을 가질 수 없다. (가질 수 있는게 이상함)
- 쉐도우 돔은 여러 번 중첩될 수 있다. `slot` 도 마찬가지다.
- 슬롯에 배포된 엘리먼트는 `slot.assignedNodes()` 를 통해 접근할 수 있다.
- 슬롯 안의 엘리먼트가 변경될 때 `slotchange` 이벤트를 `slot` 엘리먼트에 리스터늘 달아 받아오자.
- 쉐도우 호스트 스타일은 `:host` 로 변경하다.
- 쉐도우 호스트의 클래스에 따른 스타일은 `:host-context(.classname)`으로 가능하다.
- 슬롯 스타일은 `::sloted(h1)` 방식으로 한다.
- `attachShadow({mode: 'closed'})`로 쉐도우 루트를 생성하면, 쉐도우 돔에 접근이 불가능해진다.
- 쉐도우 돔 내부에서 발생한 이벤트의 `target`은 외부에서 쉐도우 호스트로 변경된다.
