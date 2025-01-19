# Safari blur event

> [출처](https://itnext.io/fixing-focus-for-safari-b5916fef1064)

safari에서는 특정 Element에 마우스로 클릭 시 focus 및 blur 이벤트가 발생하지 않는다.

## 코드

```tsx
export default function Page() {
  const handleFocus = () => console.log('focus');
  const handleBlur = () => console.log('blur');

  return (
    <>
      <button
        onFocus={handleFocus}
        onBlur={handleBlur}
      >
        asdfa
      </button>
      <div
        tabIndex={0}
        onFocus={handleFocus}
        onBlur={handleBlur}
      >
        abc
      </div>
    </>
  );
}

```

이걸 잘 보자.

1. button Element
   - Focus, Blur 이벤트가 걸려있다.
2. div Element
   - 마찬가지로 Focus, Blur 이벤트가 걸려있다.
   - 추가로 클릭시 이벤트가 원래 걸리는 요소가 아니기 때문에 `tabIndex`를 넣어주었다. tabIndex를 주게 되면 Focus 이벤트가 발생한다. 포커스를 줄 수 있는 Element가 되기 때문이다.

## 결과

### Chrome

Chrome에서는 모든 이벤트가 동작한다. Element를 클릭하면 Focus 이벤트가 동작하고, 외부를 클릭해 포커스를 뺐을 경우 Blur 이벤트가 일어난다.

### Safari

Safari에서는 `button` Element에서 Focus 이벤트 및 Blur 이벤트가 일어나지 않는다. 이유는 Click 이벤트가 일어날 경우, Safari에서는 Element에 Focus 자체를 주지 않는다. 이는 버그다. 하지만 10년 넘게 이 버그는 고쳐지지 않았다. 의도적인 것이다.

하지만 반면 `div` Element는 Focus 이벤트가 일어난다. [MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/button#clicking_and_focus)에 따르면 `<button>`, `<input>` 요소는 클릭했을 때 기본적으로 초점이 맞춰지지 않기 때문이다.

📌 따라서, 이는 button에도 `tabIndex`를 주면 해결이 된다. 단, Wrapper에도 focus 이벤트가 걸려있을 경우 relatedTarget 및 target이 변할 가능성이 생긴다.

[추가 링크](https://bugs.webkit.org/show_bug.cgi?id=22261)
