# key의 id는 useRef, props 함수는 useCallback

<br/>

## key의 id는 useRef

useRef는 key의 id 값으로도 많이 사용된다. useState로 key의 id 값을 하면 props, state가 변할 때마다 렌더링 되는데, useRef를 사용해서 만들면 랜더링이 안됨.

즉, id 값은 렌더링 되는 정보가 아님. 예를 들어 이 값은 화면에도 보이지 않고, 이 값이 바뀐다고 해서 컴포넌트가 리렌더링될 필요도 없음. 단순히 새로운 항목을 만들 때 참조되는 값일 뿐.

<br/>

## props 함수는 useCallback

습관을 들여야 함. props로 전달해야 할 함수를 만들 때는 useCallback을 사용하여 함수를 감싸는 것을 습관화 하자.