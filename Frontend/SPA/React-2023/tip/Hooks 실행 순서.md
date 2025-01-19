# Hooks 실행 순서

> [출처](https://careerly.co.kr/comments/87690?utm_campaign=user-share)

리액트의 각 Hook이 언제 실행되는지 이해하고 적용하는 것은 컴포넌트의 동작을 예측 가능하게 만드는데 중요하다. 예를 들어, 'useEffect' Hook이 렌더링 후 비동기적으로 실행된다는 사실을 이해하면, 이 Hook 내부에서 상태를 변경하는 작업 시 발생할 수 있는 잠재적인 문제를 미리 방지할 수 있다.

그렇다면, useState, useMemo, useEffect, useLayoutEffect는 어떤 순서로 실행될까?

1. useState, useMemo
2. useLayoutEffect
3. useEffect

useState와 useMemo, 이 두 Hook은 컴포넌트 렌더링 단계에서 호출된다. 렌더링 단계에서 호출되어야, Linked List로 구현된 Hook의 특성을 활용해 호출 순서를 기억하고, 상태와 관련 메모이제이션 된 값을 올바르게 유지할 수 있다.

useLayoutEffect는 렌더링이 일어난 직후, 즉 DOM 업데이트가 일어나고 브라우저가 '레이아웃 단계'를 수행하기 전 동기적으로 실행된다. 이는 Critical Rendering Path에서 'Paint' 단계 이전에 실행되는 것을 의미한다. 동기적으로 실행되므로, 사용이 과도하면 UI 블로킹을 일으킬 수 있다.

마지막으로, useEffect는 브라우저가 Paint 단계 완료 이후에 비동기적으로 실행된다. 화면이 완전히 그려진 후에 비동기적으로 실행되므로, useEffect 내의 작업이 렌더링에 영향을 주지 않는다.
