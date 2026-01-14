# 합성(composition)

> 출처: [유데미](https://www.udemy.com/course/best-react/learn/lecture/28517205#overview)

![[assets/images/d299e9e5de79dc4226f0270b44b7e20b_MD5.png]]

이런 react UI가 있다고 가정하자. 그럼 가장 큰 영역과, 내부에 들어가는 4개의 요소는 현대 UI에서 많이 사용하는 card ui다. 동일한 스타일을 가진다. 아래와 같은 css 속성을 가진다.
```css
border-radius: 12px;
box-shadow: 0 1px 8px rgba(0, 0, 0, 0.25);
```
또한 위 둘은 상위, 하위 계층구조로 나누어져 css를 서로 공유하지 않고 있다고 가정하자. 그럼 이 속성을 적용할 수 있는 컴포넌트를 제작해서 스타일을 줄 수 있다.

<br/>

## Composition(합성)이란

작은 빌딩 블러으로부터 사용자 인터페이스를 구축하는 접근 방법이다.

먼저, Card.js 컴포넌트를 제작하자.
```jsx
// Card.js
import './Card.css';

export default function Card(props) {
  const classes = 'card ' + props.className;
  return <div className={classes}>{props.children}</div>;
}
``````css
// Card.css
.card {
  border-radius: 12px;
  box-shadow: 0 1px 8px rgba(0, 0, 0, 0.25);
}
```
Card.js를 적용해보자.
```jsx
// Expense.js (상위 컴포넌트)
import Card from './Card';
import ExpenseItem from './ExpenseItem';
import './Expenses.css';

export default function Expenses({ expenses }) {
  return (
    <Card className="expenses">  // 🔥
      {expenses.map((expense) => (
        <ExpenseItem expense={expense} key={expense.id} />
      ))}
    </Card>
  );
}
``````jsx
// ExpenseItem.js (하위 컴포넌트)
import Card from './Card';
import './ExpenseItem.css';

export default function ExpenseItem(props) {
  return (
    <Card className="expense-item">  // 🔥
      <ExpenseDate date={props.expense.date} />
			(...)
    </Card>
  );
}
```
🔥 부분으로 보면, Card 컴포넌트가 사용되었다. 즉, 다른 위계에 있는 컴포넌트에서, 작은 코드 조각인 Card UI를 사용했다. 이렇게 공통 로직을 하나의 컴포넌트로 정의해 서로 다른 곳에서 사용함.

그리고 Card.js의 공통 스타일을 적용했지만, 각 컴포넌트에서 className을 적용이 되지 않았다. className으로 각 컴포넌트의 스타일을 적용해주기 위해 작성된 코드를 보자.
```jsx
export default function Card(props) {
  const classes = 'card ' + props.className;
  return <div className={classes}>{props.children}</div>;
}
```
props로 받은 className을 `card` string과 함께 div 요소에 바인딩 시켜주었다. 그러면, card … 이렇게 두 개의 className을 가진 채 서로 다른 컴포넌트에서 서로 다른 요구사항을 만족하는 className을 쓸 수 있게 되었다. 합성의 개념이다.

합성은, 작은 코드 조각으로 다른 여러 곳에서 사용될 수 있도록 유연하게 설계된 하나의 모듈이라고 생각하면 된다.
