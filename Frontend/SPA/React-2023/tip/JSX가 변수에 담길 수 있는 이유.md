# JSX가 변수에 담길 수 있는 이유

JSX는 변수에 할당할 수 있다. JSX는 JavaScript 확장 문법이기 때문에 식(expression)으로 취급된다.

```jsx
const myJSX = <div>Hello, JSX</div>;

const App = () => {
  return (
  	<>
    	<h1>Hi</h1>
    	{myJSX}
    </>
  );
}
```

`{myJSX}`에 할당한 변수 JSX가 들어간다. 따라서 JSX 코드를 재사용하거나 동적으로 생성할 수 있다.

JSX 코드가 변수에 할당될 수 있는 이유는 JSX가 Babel과 같은 트랜스파일러를 통해 JavaScript로 변환되기 때문이다. JSX는 React 컴포넌트와 함께 사용되는 JavaScript의 확장 문법이지만, 브라우저에서 직접 실행되지는 않는다. 대신, Babel과 같은 도구를 사용하여 JSX 코드를 일반적인 JavaScript 코드로 변환한다.

JSX 코드를 변수에 할당하는 경우, 할당된 JSX 코드는 일반적인 JavaScript 식(expression)으로 취급됩니다. JSX 코드는 React.createElement() 함수 호출로 변환되어 실행된다. React.createElement() 함수는 해당 JSX 요소를 표현하는 JavaScript 객체를 반환하며, 이 객체는 React가 가상 DOM을 구성하고 렌더링하는 데 사용된다.

```js
// version 16 이하

"use strict";
var myJSX = /*#__PURE__*/ React.createElement("div", null, "Hello, JSX");
var App = function App() {
  return /*#__PURE__*/ React.createElement(
    React.Fragment,
    null,
    /*#__PURE__*/ React.createElement("h1", null, "Hi"),
    myJSX // here
  );
};
```

[링크](https://babeljs.io/repl#?browsers=defaults%2C%20not%20ie%2011%2C%20not%20ie_mob%2011&build=&builtIns=false&corejs=3.21&spec=false&loose=false&code_lz=MYewdgzgLgBAtgTwFIGUAaMC8MA8ATASwDcA-ACQFMAbKkAGhlTRwHpDSBuAKC9ElgCCAByFYYACgCUWEjADeXGDABOFKAFdlYCYpgBIHCV1KDACwCM5AqwtGlJuYiYBfY7hZ2Yk7s6A&debug=false&forceAllTransforms=true&modules=false&shippedProposals=false&circleciRepo=&evaluate=false&fileSize=false&timeTravel=false&sourceType=module&lineWrap=true&presets=es2015%2Creact%2Cstage-2&prettier=true&targets=&version=7.21.8&externalPlugins=%40babel%2Fplugin-syntax-jsx%407.21.4&assumptions=%7B%7D)

JSX 코드를 babel로 변환한 결과다. `React.createElement` 함수가 '실행'된다. 그 결과가 변수에 들어가게 되는 것임. 여기서 반환하는 결과는 React 요소(element)다.

## React.createElement

```js
React.createElement(type, props, children);
```

- `type`: React 컴포넌트 또는 HTML 태그 문자열
- `props`: React 요소에 전달할 속성(props)을 나타내는 객체
- `children`: React 요소 내부에 포함될 자식 요소

이후 변수를 JSX로 사용할 때 React는 해당 변수에 할당된 JavaScript 객체를 가상 DOM으로 해석하여 렌더링함.

간단히 말해, JSX를 변수에 할당하면 JSX 코드가 JavaScript 객체로 변환되고, 이후에는 해당 변수를 JSX로 사용할 때 React가 해당 JavaScript 객체를 해석하여 화면에 표시한다.

## react 17 version 이상

```js
import { jsx as _jsx } from "react/jsx-runtime";
import { Fragment as _Fragment } from "react/jsx-runtime";
import { jsxs as _jsxs } from "react/jsx-runtime";
const myJSX = /*#__PURE__*/ _jsx("div", {
  children: "Hello, JSX"
});
const App = () => {
  return /*#__PURE__*/ _jsxs(_Fragment, {
    children: [
      /*#__PURE__*/ _jsx("h1", {
        children: "Hi"
      }),
      myJSX
    ]
  });
};
```

17부터는 `React.createElement` 를 사용하지 않는다. `_jsx` 함수를 사용한다.

이유는, 파일마다 `import React from 'react';` 를 넣어주어야 하는데, 귀찮기 때문에 이를 하지 않아도 컴파일 시 위와 같은 _jsx 함수를 그냥 사용하기 위함이다.

> ※ 단, useState 같은 Hook을 사용하기 위해서는 import를 사용해주어야 함.

[이곳](https://github.com/reactjs/rfcs/blob/createlement-rfc/text/0000-create-element-changes.md#detailed-design)에 자세한 설명이 되어 있다.