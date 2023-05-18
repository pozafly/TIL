# TIC-TAC-TOE

> [링크](https://react-ko.dev/learn/tutorial-tic-tac-toe)

코드는 여기에 -> https://github.com/pozafly/react-tic-tac-toe

### 상태값으로 선언되지 않아도 되는 값은 일반 변수로 선언하자

`xIsNext` 변수가 원래는 state 값이었다가, 일반 변수로 변경했다. 왜냐하면 상태값이 변경되면 컴포넌트 리랜더링(함수 재호출)이 일어나는데, 변수는 상태값을 기반으로 다시 계산되기 때문이다.

따라서, 리랜더링이 일어나도 반드시 기억해야하는 값이 아니라면, 혹은 상태값 기반으로 계산될 수 있는 값이라면 일반 변수로 치환하여 나타낼 수 있다. 마치 Vue.js의 computed 옵션과 비슷하다.

