# globalStyle

> [출처](https://vanilla-extract.style/documentation/global-api/global-style/#globalstyle)

vaniila-extract에서 가장 짜증나는 것은, 하위 요소를 선택할 수 없다는 것이다.

이는 vaniila-extract의 철학 때문이다. sass나, post-css등의 라이브러리는 nesting 을 할 수 있다.

```scss
nav {
  ul {
    margin: 0;
    padding: 0;
    list-style: none;
  }
  li {
    display: inline-block;
  }
  a {
    display: block;
    padding: 6px 12px;
    text-decoration: none;
  }
}
```

이런 문법 구조이다.

하지만, Vanilla-extract는 이런 것을 사용할 수 없고, `globalStyle`을 통해서 하도록 권장된다. vanilla-extract에서 하위 자식 모두를 선택하고 싶을 때 사용할 수 있다.

globalStyle 이라는 네이밍 때문에 모든 전역으로 선택자가 선택 되기는 하지만, 아래 코드와 같이 선택자를 부모 요소의 하위 자식으로 한정시킨다면 전역으로 적용되지 않는다.

```ts
// PageHeader.css.ts
export const ul = style({
  display: 'flex',
  alignItems: 'center',
  height: '100%',
  maxWidth: 800,
  margin: '0 auto',
});

globalStyle(`${ul} li:not(:first-child)`, {
  marginLeft: 30,
  height: '100%',
  lineHeight: '35px',
});
```

위 코드는, `.PageHeader_ul__1ho3ogc2` 라는 곳에 걸린 스타일이다.

<img width="395" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/2d7799d4-32ce-4e36-a57c-3a6e3e02e324">

그리고 그 하위에 `li : list-style: none` 에 걸린 것은, 어플리케이션 전체 코드의 globalStyle에 걸린 것이다.

```ts
// global.css.ts
globalStyle('ul, li', {
  listStyle: 'none',
});
```

따라서, globalStyle을 명확한 parent를 지정해준다면 걱정하지 않고 사용할 수 있다.

---

단, globalStyle에서는 의사 코드 및 복합 선택기를 할 수 없다. 예를 들어 아래와 같은 코드를 작성할 수 없다는 것이다.

```css
ul li:first-child, a > span.
```

