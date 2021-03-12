# 코드 분할(code splitting)

코드 분할을 이용하면 사용자에게 필요한 양의 코드만 내려 줄 수 있다. 코드 분할을 사용하지 않으면 전체 코드를 한 번에 내려 주기 때문에 첫 페이지가 뜨는 시간이 오래 걸린다. 코드를 분할하는 한 가지 방법은 동적 임포트를 이용하는 것.

Todo를 만들어보자.

```jsx
// Todo.js
import React from 'react';

const Todo = ({ title }) => {
  return <div>{title}</div>;
};

export default Todo;
```

```jsx
// TodoList.js
import React, { useState } from 'react';

const TodoList = () => {
  const [todos, setTodos] = useState([]);
  const onClick = () => {
    import('./Todo.js').then(({ Todo }) => {   // 📌 1
      const position = todos.length + 1;
      const newTodo = <Todo key={position} title={`할 일 ${position}`} />;      // 📌 2
      setTodos([...todos, newTodo]);
    });
  };
  return (
    <div>
      <button onClick={onClick}>할 일 추가</button>
      {todos}
    </div>
  );
};

export default TodoList;
```

- 📌 1 : onClick 함수가 호출되면 비동기로 Todo 모듈을 가져온다. 동적 임포트는 프로미스를 반환하기 때문에 then 메서드를 이용해 이후 동작을 정의할 수 있음.
- 📌 2 : 비동기로 가져온 Todo 컴포넌트를 이용해서 새로운 할 일을 만든다.

결국 Todo.js 파일은 별도의 JS 파일로 분리 되었고, 필요한 경우에만 내려받도록 구현되었다. SPA를 만들기 위해 react-router-dom 패키지를 이용하는 경우에는 react-router-dom 에서 지원하는 기능을 이용해 페이지 단위로 코드 분할을 적용할 수 있다.