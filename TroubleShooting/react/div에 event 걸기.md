# Div에 event 걸기

react의 on… 이벤트는 일반적으로 input 태그에 걸리는 경우가 많다. 아마도 react에서 사용하는 event 객체는 일반 객체가 아니라 react에서 따로 만든 합성 이벤트(SyntheticEvent)를 사용한다고 한다. 이것 때문에 걸리지 않는건지 onKeyPress 종류의 keyboard 종류 이벤트만 걸리지 않는지는 잘 모르겠다.

어쨌든, div 태그에 key 관련 이벤트를 먹여보자.

<img width="906" alt="스크린샷 2021-04-01 오후 8 18 30" src="https://user-images.githubusercontent.com/59427983/113287081-06fab280-9328-11eb-8a12-629e8a368c87.png">

포트폴리오로 만들기 시작한 RetroFM 첫 화면이다. 여기 전체(div 태그)에 onKeyPress 이벤트를 걸고 싶다.

```jsx
function IntroDisplay() {
  const blockRef = useRef();
  const onKeyPress = (e) => {
    console.log(e.key);
    if (e.key === 'Enter' || e.key === ' ') {
      console.log('눌림');
    }
  };

  useEffect(() => {
    blockRef.current.focus();
  }, []);
  
  return (
    <IntroDisplayBlock onKeyPress={onKeyPress} tabIndex={0} ref={blockRef}>
      ...
    </IntroDisplayBlock>
  );
}
```

요렇게 걸어주면 된다. useEffect로 처음 화면이 불러와진 후, focus를 주고, tabIndex를 첫번째(0)로 주면 포커싱 된 상태이다.

포커싱 된 상태라면 화면 전체에 파란색 줄이 쳐져서 포커싱 됨을 알리는데, 이게 보기가 싫으면,

```css
&:focus {
  outline: none;
}
```

css 를 요렇게 먹여주도록 하자.
