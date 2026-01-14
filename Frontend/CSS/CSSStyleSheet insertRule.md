# CSSStyleSheet insertRule

> [MDN](https://developer.mozilla.org/en-US/docs/Web/API/CSSStyleSheet/insertRule)

CSSStyleSheet에서 insertRule() 메서드가 존재한다.

이것은 JavaScript를 이용해서 스타일을 수정할 수 있도록 하는 메서드다.

처음 생각했을 때는 style 태그를 `docuement.creatElement` 로 만들어서 그냥 스타일 태그를 넣어주면 되는 게 아닌가 생각했다.

따라서 아래와 같은 코드가 나오게 된다.
```js
const styleTag = document.createElement('style');
styleTag.type = 'text/css';
styleTag.textContent = 'body { border: 20px solid red }';
document.head.appendChild(styleTag);
```
이 코드를 실행하면, HTML 상에 새로운 style 태그가 삽입된다. 그리고 CSS도 정상적으로 동작하는 것을 볼 수 있다.

![[assets/images/3d043253af06ca087a4d3c236158e4a6_MD5.png]]

잘 동작한다. 하지만, CSSStyleSheet insertRule() 메서드를 사용하면, style 태그를 새로 생성하는 것이 아니라, CSSOM에 바로 넣을 수 있다.
```js
const sheet = document.styleSheets[0];
sheet.insertRule(
  'body{ border-bottom: 1px solid #CCC; color: #FF0000; }',
  0
);
```
그러면 style 태그에 따로 추가되지는 않지만, CSS가 적용되는 것을 볼 수 있다. 즉, CSSOM 트리에 해당 규칙을 새로 생성하여 넣어준 것이다.

이는 **stitches.js** 와 같은 zero-runtime라이브러리에서 동적으로 스타일 규칙을 삽입할 때 사용하는 방법이다. ([링크](https://so-so.dev/web/css-in-js-whats-the-defference/))

transition이 걸려있다면 rule이 추가되는 것이므로 CSS-in-JS에서도 자연스럽게 잘 동작한다.
