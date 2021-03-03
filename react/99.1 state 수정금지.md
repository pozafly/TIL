# 동작원리

리액트는 변경사항이 한가지의 방향으로만 흘러간다.

- 데이터가 변경 되면 -> UI가 업데이트 된다.
- 데이터(State)가 변경 되면 -> 리액트가 render() 함수를 호출해서 UI가 업데이트 된다.

```jsx
class App extends Component {
  state = {
    count: 0,
  };
  render() {
    return (
      <>
        <span>{this.state.count}</span>
        <button
          onClick={() => {
            // 클릭이 되면 count를 하나씩 증가 한다
          }}
        >
          Click
        </button>
      </>
    )
  }
}
```

클래스 컴포넌트이고, this.state에는 숫자를 담을 수 있는 count가 들어있는 오브젝트가 있다. 그리고 button이 클릭되면 onClick 콜백이 실행될 것이고, 우리가 State를 업데이트 시켜줄 수 있다.



## 1. State를 바로 수정하는 경우(좋지않음)

```jsx
<button
  onClick={() => {
    this.state.count++;   // 이곳
  }}
>
  Click
</button>
```

this.state.cout++는 this.state를 직접적으로 수정하고 있으므로 동작하지 않음. 리액트에서는 상태를 제대로 업데이트 하기 위해서 setState 함수를 호출해야 함.

## 2. setState 사용하기

```jsx
<button
  onClick={() => {
    this.setState({ count: this.state.count + 1 });   // 이곳
  }}
>
  Click
</button>
```

setState 함수를 이용해 새로운 상태 오브젝트(업데이트 하고자 하는 상태 데이터)를 인자로 전달했다. 리액트가 업데이트가 되어야 한다고 알아 차리게 하기 위해서 이렇게 setState 함수를 호출해주어야 함.

이제 UI를 업데이트 하기 위해서 render 함수를 호출 해줘야지! 하고 알아차리게 됨.

setState 함수가 호출이 되면 리액트는 **현재 가지고 있는 상태**(this.state)와, **업데이트 해야하는 새로운상태**(setState 함수의 인자로 전달된 새로운 오브젝트) 두 가지를 비교해서 업데이트가 필요한 경우 해당 컴포넌트의 render 함수를 호출해준다.

---

현재 state와 새로운 state를 비교하는 방식

- PureComponent 인 경우 두 가지를 얉게 비교해서 (제일 상위 reference 만 비교) 달라진게 있다면 컴포넌트를 업데이트 함.
- 일반 Component라면 따로 라이프사이클 메서드인 shouldComponentUpdate를 구현하지 않았다면 setState가 호출될 때마다 무조건 render 함수가 호출된다.

※ setState는 비동기 API다.

WebAPIs 중에 setTimeout, setInterval과 같은 비동기 함수처럼, setState도 마찬가지. 따라서, setState를 호출한다고 해서 무조건 render 함수가 호출되는 것이 아니라 리액트에 업데이트 요청을 하기만 하고 다시 뒤에 이어지는 코드가 실행되어진다. 비동기 이기 때문에 리액트가 동시 다발적으로 요청된 여러가지 setState를 효율적으로 처리할 수 있다.

<br/>

## 🔥 **정말 중요**

그리고, `state를 업데이트 할때 이전 state 값에서 어떤 계산이 되어지는 경우`라면, 컴포넌트 내의 state 값에 의존해서 계산한 값은 setState(updated) 로 설정하기 보다는, `setState(prevState => newState)` 이렇게 이전 state 값을 받아 그걸로 업데이트 되는  state 값을 만드는 arrow 함수를 전달할 수 있는 함수를 호출 하는게 좋다.

리액트에서 제공하는 setSate 함수는 두가지 종류가 있다.

- setState(newState) : 새로운 state **오브젝트를 인자**로 바로 받는 함수
- setState(prevState => { return newState; }) : 이전 state를 받아서 그걸로 계산해 새로운 state를 리턴하는 **함수를 인자로 받는 함수**.

```jsx
<button
  onClick={() => {
    this.setState(state => ({
      count: state.count + 1,
    }));
  }}
>
  Click
</button>
```

이런 식으로 말이다.

<br/>

## state를 수정하면 안되는 이유

리액트에서는 상태를 직접적으로 절대! 수정하면 좋지 않다.

다른 프로그래밍에서도 Object를 직접적으로 변경하는 것은 좋지 않다. 예상치 못한 오류가 발생하는 것을 피하기 위해서 이미 만들어진 오브젝트는 항상 불변성(Immutability)을 유지하는게 좋다.

다만, 좋지 않다는 뜻은, 절대적으로 state 오브젝트를 수정하지 못하도록 방지하거나, 또 수정한다고 해서 바로 심각한 오류가 눈에 보이는 것은 아니기 때문이다.

```jsx
<button
  onClick={() => {
    this.state.count++;
    this.setState(this.state);
  }}
>
  Click
</button>
```

this.state가 가리키고 있는 오브젝트의 count를 바로 직접적으로 수정하고, 수정된 this.state 자체를 setState의 인자로 전달해주었다. 실행 시 나름 잘 동작하는 것을 볼 수 있다.

<br/>

### <u>state를 직접적으로 수정하는게 좋지 않은 이유</u>

`1. setState는 비동기적으로 동작한다`

- state를 직접 수정하면서 여러번 상태를 업데이트 하는 경우 **이전 업데이트 내용이 다음 업데이트 내용으로 덮어 쓰여질 수 있고,** 비동기 특성으로 인해 예상치 못한 곳에서, 예상치 못한 순간에 버그 발생 위험이 있다.

`2. PureComponent 에서 정상적으로 동작하지 않는다`

- 위 컴포넌트를 PureComponent로 변경해보면 업데이트가 되지 않는다.
- PureComponent는 현재 컴포넌트가 가지고 있는 상태(this.state), 업데이트 해야 하는 새로운 상태(setState 함수의 인자로 전달된 새로운 오브젝트)의 레퍼런스를 비교해서 업데이트가 필요한 경우 해당 컴포넌트의 render 함수를 호출한다.
- 지금 경우는 this.state 오브젝트를 직접적으로 수정해서 setState 함수에 동일한 오브젝트를 전달하므로, 비교해야 하는 대상의 레퍼런스가 동일하므로 리액트는 업데이트 할 필요가 없다 판단해서 render 함수를 호출해주지 않는다.

따라서 리액트의 state를 직접적으로 수정하는 것은 예상치 못한 문제가 발생할 수 있기 때문에, 반드시 불변성을 유지 하는게 좋다.

