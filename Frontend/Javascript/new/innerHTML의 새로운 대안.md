# innerHTML의 새로운 대안

> [출처](https://velog.io/@typo/new-alternatives-to-innerHTML?utm_source=substack&utm_medium=email)

`setHTMLUnsafe` 는 [모든 브라우저](https://caniuse.com/?search=sethtmlunsafe)에서 지원된다. `setHTML` 은 아직 표준화 작업이 진행 중이며, 현재는 파이어폭스에서 플래그를 활성화해야만 사용할 수 있다. [getHTML](https://caniuse.com/mdn-api_element_gethtml)은 버전 125부터 크롬과 엣지에서 지원된다.

최근 브라우저에 `setHTMLUnsafe` 메서드가 새롭게 추가되었음. 안전하지 않다(unsafe)는 것은 `innerHTML` 과 마찬가지로 입력 값 검증(sanitization)을 수행하지 않음을 의미한다. 이 명칭은 기존 브라우저 API의 네이밍 방식과 차이가 있음. 예를 들어, `innerHTML` 을 굳이 `innerHTMLUnsafe` 라고 하지 않았고, `eval` 을 `evalUnsafe()` 라고 하지 않았다. 그렇다고 해서 `setHTMLUnafe` 라고 하지 않았고, `eval` 을 `evalUnsafe()` 라고 하지 않았다. 그렇다고 해서 `setHTMLUnsafe` 는 기존 메서드들보다 더 위험한 건 아니다. 오래된 메서드들과 다르게 안전한 메서드(`setHTML`) 과 안전하지 않은 메서드(`setHTMLUnsafe`) 가 있기 때문에 메서드 이름에 'unsafe'를 명시하는 방식을 사용한 것이다.

[Sanitizer API 명세](https://wicg.github.io/sanitizer-api/)에는 다음과 같은 구절이 있다.

> "안전한" 메서드는 스크립트를 실행하는 마크업을 생성하지 않습니다. 즉, XSS(Cross-Site Scripting)로부터 안전해야 합니다.

텍스트 `<input>` 과 사용자가 입력 값에 따라 DOM을 변경하는 js 코드를 갖는 form이 있다해보자.

```js
form.addEventListener('submit', function(event) {
  event.preventDefault();
  const markup = `<h2>${input.value}</h2>`;
  div.innerHTML = markup;
});
```

만약 사용자가 input에 `<img src="dosenotexist">` 와 같이 입력했다면 js 코드는 브라우저에서 작동하게 되고 `.setHTMLUnsafe()` 메서드도 동일한 위험을 지니고 있다.

간단한 코드는 브라우저에서만 실행되지만 DB에 저장되어 다른 사용자에게 동적 콘텐츠로 보여지면 브라우저에서 임의의 잠재적 악성 js가 살행될 수 있다.

`setHTML` 을 사용하면 DOM에 삽입되는 것은 `<img src="doesnotexist">` 뿐이다. 이미지는 페이지에 삽입되지만 js는 제거된다.

Sanitizer API는 아직 개발 중이지만, 이런 배경지식은 `setHTMLUnsafe` 의 이름의 맥락을 이해하는데 도움이 된다.

<br/>

## setHTMLUnsafe

이미 `innerHTML` 이 있고, 다행히도 `setHTML` 이 개발되고 있는데 왜 `setHTMLUnsafe` 가 필요할까? 바로 선언적 쉐도우 DOM(declarative shadow DOM) 때문이다.

HTML `<template>` 요소는 다음 두 가지 방식으로 사용될 수 있다.

- 렌더링 되지는 않지만 추후 js로 사용할 수 있는 HTML 조각(fragment)를 보관한다.
- 쉐도우 DOM을 즉시 생성한다. `<template>` 이 `shadowrootmode` 속성을 포함하는 경우, 쉐도우 루트에서 해당 요소는 DOM 내의 요소 콘텐츠로 대체된다.

`innerHTML` 은 전자의 방식에서는 잘 작동하지만 후자의 방식을 처리하진 못한다.

```js
const main = document.querySelector('main');
main.innerHTML = `
    <h2>I am in the Light DOM</h2>
    <div>
      <template shadowrootmode="open">
          <style>
          	h2 { color: blue; }
          </style>
          <h2>Shadow DOM</h2>
      </template>
    </div>`
```

innerHTML은 페이지에 `<template>` 을 삽입할 순 있지만 여전히 `<template>` 요소로 남아있다. 이 `<template>` 요소는 내용도 렌더링 되지 않고, `shadowrootmode` 속성 여부에 관계 없이 쉐도우 DOM으로 변환되지도 않는다.

`setHTML` 은 `<template>` 과 그 콘텐츠를 의도적으로 제거한다.

```js
const main = document.querySelector('main');
main.setHTML(`
    <h2>I am in the Light DOM</h2>
    <div>
      <template shadowrootmode="open">
          <style>
          	h2 { color: blue; }
          </style>
          <h2>Shadow DOM</h2>
      </template>
    </div>`);
```

위 예제에서 `main` 컨텐츠는 `h2` 와 빈 `div` 뿐이다. `template` 은 '안전하지 않은 노드'로 취급된다.

이것이 바로 브라우저가 `setHTMLUnsafe` 메서드를 추가한 이유다. 페이지에 선언적 쉐도우 DOM을 동적으로 추가하기 위함이다.

```js
main.setHTMLUnsafe(`
    <h2>I am in the Light DOM</h2>
    <div>
      <template shadowrootmode="open">
          <style>
            h2 { color: blue; }
          </style>
          <h2>Shadow DOM</h2>
      </template>
    </div>
`);
```

`setHTMLUnsafe` 를 사용하면 `<template>` 의 콘텐츠는 쉐도우 DOM 내부에 렌더링 된다.

<br/>

## getHTML

`setHTML` 과 `setHTMLUnsafe` 는 `innerHTML` 을 완전히 대체할 수는 없다. `innerHTML` 은 HTML을 설정(set)하고 가져올(get) 수 있기 때문이다. `setHTML` 과 `setHTMLUnsafe` 를 보완하는 함수가 [`getHTML`](https://html.spec.whatwg.org/multipage/dynamic-markup-insertion.html#html-serialization-methods) 이다(안전하지 않음).

```js
const main = document.querySelector('main');
const html = main.getHTML();
```

`serializableShadowRoots` 프로퍼티를 true로 설정하면 직렬화가 가능한 모든 쉐도우 DOM 트리를 직렬화한다.

`shadowrootserializable` 속성을 사용하여 `template` 요소의 직렬화를 가능하게 만들 수 있다.

```html
<template shadowrootmode="open" shadowrootserializable>
```

이와 비슷하게 js `attachShadow` 메서드에는 `serializable` boolean 옵션이 있다.

```js
this.attachShadow({ mode: "open", serializable: true });
```

쉐도우 루트 배열을 전달하여 특정 쉐도우 DOM 트리만 직렬화할 수도 있다.

```js
const markup= main.getHTML({
    shadowRoots: [document.querySelector('.example').shadowRoot]
});
```

배열의 모든 쉐도우 루트는 직렬화가 가능하다고 명시적으로 표시되어 있지 않더라도 직렬화된다.
