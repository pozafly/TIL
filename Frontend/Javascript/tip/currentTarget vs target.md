# curretTarget vs target

> 출처 : https://velog.io/@edie_ko/JavaScript-event-target%EA%B3%BC-currentTarget%EC%9D%98-%EC%B0%A8%EC%9D%B4%EC%A0%90

event.target과 event.currentTarget의 차이점에 대해 알아보자.

- currentTarget : 이벤트 핸들러가 부착된 것을 가리킨다. 이벤트가 부착된 `부모` 의 위치를 반환.
- target : 부모로부터 이벤트가 위임되어 발생하는 자식의 위치, 내가 클릭한 `자식 요소` 를 반환한다.

```jsx
const App = () => {
  const onClick = (event) => {
    console.log(event.target);
    console.log(event.currentTarget);
  };
  return (
    <div>
      <li>
        <button onClick={onClick}>
          <span>무야호</span>
        </button>
      </li>
    </div>
  );
};
```

이렇다고 해보자. 클릭했을 때는 이렇게 나온다.

<img width="141" alt="스크린샷 2021-03-25 오후 2 34 04" src="https://user-images.githubusercontent.com/59427983/112424123-35501f00-8d77-11eb-96d4-a30336a1426b.png">

- `<span>무야호</span>` 는 event.target 이다.
- `<button><span>무야호</span></button>` 은 event.currentTarget 이다.

즉, event.target은 자식 요소인 span을 리턴하고, event.currentTarget은 부모 요소인 button을 반환한다.

다시 말하면, `currentTarget` 은 이벤트가 걸려있는 위치를 반환(`this` 가 가리키는 것과 같다)하고, `target` 은 실제 이벤트가 발생하는 위치, 내가 클릭한 요소를 반환한다.