# getBoundingClientRect

> [MDN](https://developer.mozilla.org/ko/docs/Web/API/Element/getBoundingClientRect)

`Element.getBoundingClientRect()` 메서드는 엘리먼트의 크기와 뷰포트에 상대적 위치 정보를 제공하는 DOMRect 객체를 반환한다.

```js
const domRect = element.getBoundingClientReat();
```

## 값

반환 값은 padding과 border-width를 포함해 전체 엘리먼트가 들어 있는 가장 작은 사각형인 DOMRect 객체다.

`left`, `top`, `right`, `bottom`, `x`, `y`, `width`, `height` 프로퍼티는 전반적인 사각형의 위치와 크기를 픽셀 단위로 나타낸다. width, height가 아닌 다른 프로퍼티는 뷰포트의 top-left에 상대적이다.

![element-box-diagram](https://developer.mozilla.org/ko/docs/Web/API/Element/getBoundingClientRect/element-box-diagram.png)

메서드가 반환하는 DOMReact 객체의 width, height 프로퍼니틑 콘텐츠의 width/height 뿐 아니라 padding과 border-width도 포함한다. 표준 박스 모델에서, 이는 엘리먼트 + padding + border-width의 width 또는 height 프로퍼티와 동일하다. 하지만, `box-sizing: border-box` 가 새당 엘리먼트에 설정되어 있으면, 이는 width, height와 직접적으로 동일하다.

반환 값은 엘리먼트의 getClientRects() 가 반환한 사각형들의 결합으로 생각해볼 수 있다. 해당 엘리먼트에 관계된 CSS border-box들을 예로 들어볼 수 있다.

빈 border-box들은 완전히 무시한다. 모든 엘리먼트의 border-box가 비어있다면 width, height가 0인 사각형을 반환하며 top, left는 해당엘리먼트의 첫 번째 CSS 박스에 대한 border-boxdml top-left 이다.

뷰포트 영영(또는 스크롤 가능한 엘리먼트)에서 수행된 스크롤의 정도는 경계 사각형을 계산할 때 고려된다. 이는 사각형의 경계 모서리(top, right, bottom, left)는 스크롤링 위치가 변경될 때마다 그 값이 변경됨을 의미한다.(이 값은 절대적인 것이 아니라 뷰포트에 상대적이기 때문이다.)