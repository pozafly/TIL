# setState 비동기

## React의 state update는 동기(synchronous) 동작인가요, 비동기(asynchronous) 동작인가? 그렇게 했을 때의 장점은 무엇인가?


setState는 비동기 동작이다. setState 함수 자체는 동기 함수이지만, setState가 비동기 적으로 호출된다. 

이유는, 효율성이기 때문이다. 만약, 동기적으로 일어나게 되면, 한 함수에 여러 setState가 있을 때, VDOM의 변화를 계산하는데 매번 다시 계산(즉, re-render) 해야한다. 하지만, 비동기적으로 묶어 계산하게 되면, 변화된 부분을 한 번만 계산해 렌더링 하면 되기 때문이다.

즉, 비동기적으로 일어나야 batch update(한 번에 모아 업데이트)가 가능하고, 렌더링의 우선순위를 설정할 수 있기 때문이다.

---

또, 내부 일관성을 보장할 수 있다. 만약, 상위 컴포넌트의 state를 현재 props로 가져왔다하면, props는 스냅샷으로 관리되기 때문에 props 업데이트를 따로 해주어야 한다.
즉, 현재(하위) 컴포넌트에서 동기적으로 setState가 일어난다면 일괄 처리 자체가 불가능해진다.

또, 동시 업데이트 활성화의 이유 때문이다. 리액트는 이벤트 핸들러, 네트워크 응답, 애니메이션등의 출처에 따라 setState() 호출에 각각 다른 우선순위를 할당할 수 있다. 만약, 동기라면 위의 상황에 맞는 우선순위를 할당할 수 없게 된다. 비동기 콜백을 모아두고, 어떤 부분을 먼저 실행할지 스케줄링 할 수 있기 때문이다.

---

출처1. https://legacy.reactjs.org/docs/faq-state.html#why-is-setstate-giving-me-the-wrong-value
출처2. https://velog.io/@jay/setStateisnotasync
출처3. https://velog.io/@hyunjine/%EC%99%9C-setState%EB%8A%94-%EB%B9%84%EB%8F%99%EA%B8%B0%EC%A0%81%EC%9C%BC%EB%A1%9C-%EB%8F%99%EC%9E%91%ED%95%98%EB%8A%94%EA%B0%80