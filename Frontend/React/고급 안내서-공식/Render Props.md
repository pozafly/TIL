# Render Props

render prop이란, React 컴포넌트 간 코드를 공유하기 위해 함수 props를 이용한 간단한 테크닉임.

render props패턴으로 구현된 컴포넌트는 자체적으로 렌더링 로직을 구현하는 대신, react 엘리먼트 요소를 반환하고 이를 호출하는 함수를 사용한다.

```jsx
<DataProvider render={data => (
  <h1>Hello {data.target}</h1>
)}
```

render props 를 사용하는 라이브러리는 React Router, Downshift, Formik 등이 있다.

<br/>

## 횡단 관심사(Cross-Cutting Concerns)를 위한 render props 사용법

컴포넌트는 React에서 코드의 재사용성을 위해 사용하는 주요 단위. 하지만 컴포넌트에서 캡슐화된 상태나 동작을 같은 상태를 가진 다른 컴포넌트와 공유하는 방법이 항상 명확하지는 않다.

아래는 마우스 위치를 추척하는 로직

```jsx
import React, { useState } from 'react';

const MouseTracker = () => {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  const handleMouseMove = (event) => {
    setPosition({
      ...position,
      x: event.clientX,
      y: event.clientY,
    });
  };

  return (
    <div style={{ height: '100vh' }} onMouseMove={handleMouseMove}>
      <h1>Mouse the mouse around!</h1>
      <p>
        The current mouse position is ({position.x}, {position.y})
      </p>
    </div>
  );
};

export default MouseTracker;
```

스크린 주위로 커서를 움직이면, 컴포넌트가 마우스의 (x, y) 좌표를 `<p>` 에 나타냄.

다른 컴포넌트에서 이걸 재사용하려면 어떻게 해야 할까? 캡슐화 가능한가? `<Mouse>` 컴포넌트로 캡슐화 해보자.

```jsx
import React, { useState } from 'react';

// 캡슐화 한다.
const Mouse = () => {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  const handleMouseMove = (event) => {
    setPosition({
      ...position,
      x: event.clientX,
      y: event.clientY,
    });
  };
  return (
    <div style={{ height: '100vh' }} onMouseMove={handleMouseMove}>
      <p>
        The current mouse position is ({position.x}, {position.y})
      </p>
    </div>
  );
};

const MouseTracker = () => {
  return (
    <>
      {/* ...하지만 <p>가 아닌 다른것을 렌더링하려면 어떻게 해야 하나? */}
      <h1>Mouse the mouse around!</h1>
      <Mouse />
    </>
  );
};

export default MouseTracker;
```

캡슐화 했음. 아직 완벽하게 사용할 수 있는 것은 아님. 예를 들어, 마우스 주위에 고양이 그림을 보여주는 `<Cat>` 컴포넌트를 생각해보자. `<Cat mouse={{x, y}}>` prop을 통해 Cat 컴포넌트에 마우스 좌표를 전달해주고 화면에 어떤 위치에 이미지를 보여줄지 알려 주고자 한다.

첫번째 방법으로는, `<Mouse>` 컴포넌트에 `<Cat>` 컴포넌트를 넣어 렌더링 하는 방법이 있다.

```jsx
import React, { useState } from 'react';

const Cat = ({ mouse }) => {
  return (
    <img
      src='./고양이.png'
      style={{ position: 'absolute', left: mouse.x, top: mouse.y }}
      alt='고양이'
    />
  );
};

const MouseWithCat = () => {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  const handleMouseMove = (event) => {
    setPosition({
      ...position,
      x: event.clientX,
      y: event.clientY,
    });
  };
  return (
    <div style={{ height: '100vh' }} onMouseMove={handleMouseMove}>
      {/*
        여기서 <p>를 <Cat>으로 바꿀 수 있습니다. ... 그러나 이 경우
        Mouse 컴포넌트를 사용할 때 마다 별도의 <MouseWithSomethingElse>
        컴포넌트를 만들어야 합니다, 그러므로 <MouseWithCat>는
        아직 정말로 재사용이 가능한게 아닙니다.
      */}
      <Cat mouse={position} />
    </div>
  );
};

const MouseTracker = () => {
  return (
    <div>
      <h1>Mouse the mouse around!</h1>
      <MouseWithCat />
    </div>
  );
};

export default MouseTracker;
```

이런 접근 방법은 특정 사례에서는 적용할 수 있지만, 우리가 원하는 행위의 캡슐화(마우스 트래킹)라는 목표는 달성하지 못했음. 다른 사용 예제에서도 언제든지 마우스 위치를 추적할 수 있는 새로운 component(`<MouseWithCat>` 과 근본적으로 다른)를 만들어야 함.

여기에 render prop을 사용할 수 있다. `<Mouse>` 컴포넌트 안에 `<Cat>` 컴포넌트를 하드 코딩(hard-coding) 해서 결과물을 바꾸는 대신, `<Mouse>` 에게 동적으로 렌더링 할 수 있도록 해주는 함수 prop을 제공하는 것이다. -> 이것이 render prop의 개념이다.

```jsx
import React, { useState } from 'react';

const Cat = ({ mouse }) => {
  return (
    <img
      src='./고양이.png'
      style={{ position: 'absolute', left: mouse.x, top: mouse.y }}
      alt='고양이'
    />
  );
};

const Mouse = ({ render }) => {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  const handleMouseMove = (event) => {
    setPosition({
      ...position,
      x: event.clientX,
      y: event.clientY,
    });
  };
  return (
    <div style={{ height: '100vh' }} onMouseMove={handleMouseMove}>
      {/*
        <Mouse>가 무엇을 렌더링하는지에 대해 명확히 코드로 표기하는 대신,
        `render` prop을 사용하여 무엇을 렌더링할지 동적으로 결정할 수 있습니다.
      */}
      {render(position)}
    </div>
  );
};

const MouseTracker = () => {
  return (
    <div>
      <h1>Mouse the mouse around!</h1>
      <Mouse render={(mouse) => <Cat mouse={mouse} />} />
    </div>
  );
};

export default MouseTracker;
```

이제 컴포넌트의 행위를 복제하기 위해 하드 코딩할 필요 없이 render 함수에 prop으로 전달해줌으로써, `<Mouse>` 컴포넌트는 동작으로 트래킹 기능을 가진 컴포넌트들을 렌더링 할 수 있다.

정리하자면, **render prop은 무엇을 렌더링할지 컴포넌트에 알려주는 함수**다.

이 테크닉은 행위(마우스 트래킹 같은)를 매우 쉽게 공유할 수 있도록 만들어준다. 해당 행위를 적용하려면, `<Mouse>` 를 그리고 현재 (x, y) 커서 위치에 무엇을 그릴지에 대한 정보를 prop을 통해 넘겨주기만 하면 된다.

render props에 대해 한가지 흥미로운 점은 대부분의 high-order-components(HOC)에 render props pattern을 이식할 수 있다. 예를 들면, 만약 컴포넌트보다 withMouse HOC를 더 선호한다면 render prop을 이용해 다음과 같이 쉽게 HOC를 만들 수 있음.

```jsx
// 만약 어떤 이유 때문에 HOC를 만들기 원한다면, 쉽게 구현할 수 있다.
// render prop을 이용해서 일반적인 컴포넌트를 만들어라.
function withMouse(Component) {
  return class extends React.Component {
    render() {
      return (
        <Mouse render={mouse => (
          <Component {...this.props} mouse={mouse} />
        )}/>
      );
    }
  }
}
```

따라서 render props를 사용하면 두 가지 패턴 모두 사용 가능함.

<br/>

## render 이외의 Props 사용법

여기서 중요하게 기억해야 할 것은, 'render props pattern'으로 불리는 이유로 꼭 prop name으로 render를 사용할 필요 없음.

위 예제에서는 `render`를 사용했지만, 우리는 `children` prop을 더 쉽게 사용할 수 있다.

```jsx
<Mouse children={mouse => (
  <p>The mouse position is {mouse.x}, {mouse.y}</p>
)}/>
```

실제로 JSX element의 “어트리뷰트” 목록에 하위 어트리뷰트 이름(예를들면 render)을 지정할 필요는 없다. 대신에, element *안에* 직접 꽂아넣을 수 있음.

```jsx
<Mouse>
  {mouse => (
    <p>The mouse position is {mouse.x}, {mouse.y}</p>
  )}
</Mouse>
```

이 테크닉은 자주 사용되지 않기 때문에, API를 디자인할 때 `children`은 함수 타입을 가지도록 `propTypes`를 지정하는 것이 좋다.

```jsx
Mouse.propTypes = {
  children: PropTypes.func.isRequired
};
```

<br/>

## 📌 주의 사항

### React.PureComponent에서 render props pattern을 사용할 땐 주의

render props 패턴을 사용하면 [`React.PureComponent`](https://ko.reactjs.org/docs/react-api.html#reactpurecomponent)를 사용할 때 발생하는 이점이 사라질 수 있다. 얕은 prop 비교는 새로운 prop에 대해 항상 `false`를 반환합니다. 이 경우 `render`마다 render prop으로 넘어온 값을 항상 새로 생성함.

위에서 사용했던 `<Mouse>` 컴포넌트를 이용해서 예를 들어보자. mouse에 `React.Component` 대신에 `React.PureComponent`를 사용하면 다음과 같은 코드가 됨.

```jsx
const Mouse = React.memo(({ render }) => {
  // 위와 같은 구현체.
});

const MouseTracker = () => {
  return (
    <div>
      <h1>Mouse the mouse around!</h1>
      {/*
        이것은 좋지 않음. `render` prop이 가지고 있는 값은
        각각 다른 컴포넌트를 렌더링 할 것임.
      */}
      <Mouse render={(mouse) => <Cat mouse={mouse} />} />
    </div>
  );
};
```

이 예제에서 `<MouseTracker>`가 render 될때마다, `<Mouse render>`의 prop으로 넘어가는 함수가 계속 새로 생성됩니다. 따라서 `React.PureComponent`를 상속받은(함수형 컴포넌트에서는 React.memo를 사용한) `<Mouse>` 컴포넌트 효과가 사라지게 된다.

이 문제를 해결하기 위해서, 다음과 같이 인스턴스 메서드를 사용해서 prop을 정의하자.

```jsx
const MouseTracker = () => {
  const renderTheCat = useCallback((mouse) => <Cat mouse={mouse} />, []);
  return (
    <div>
      <h1>Mouse the mouse around!</h1>
      <Mouse render={renderTheCat} />
    </div>
  );
};
```

난 함수형 컴포넌트를 사용하고 있었기 때문에 `useCallback` 으로 감싸 주었다. 공식 홈피는

```jsx
class MouseTracker extends React.Component {
  // `this.renderTheCat`를 항상 생성하는 매서드를 정의합니다.
  // 이것은 render를 사용할 때 마다 *같은* 함수를 참조합니다.
  renderTheCat(mouse) {
    return <Cat mouse={mouse} />;
  }

  render() {
    return (
      <div>
        <h1>Move the mouse around!</h1>
        <Mouse render={this.renderTheCat} />
      </div>
    );
  }
}
```

이렇게 설명하고 있다.