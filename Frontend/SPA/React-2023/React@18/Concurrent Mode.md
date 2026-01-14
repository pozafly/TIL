# Concurrent Mode

> [출처](https://tecoble.techcourse.co.kr/post/2021-07-24-concurrent-mode/)

JavaScript는 싱글 스레드 언어다. 하나의 작업 중 다른 작업을 동시에 수행할 수 없다. 하지만, React에서 concurrent mode를 사용하면 여러 작업을 동시에 처리할 수 있음.

리액트가 정말 한번에 하나의 작업만 처리하는가? UI렌더링 이외 모든 작업을 차단함. 입력값을 입력할 때마다 5000개의 DOM Element를 생성한다고 가정해보자. 어떻게 이걸 효율적으로 그려줄까? 바로 동시성이다.

동시성은 여러 작업을 작은 단위로 나눈 뒤, 그들 간 우선순위를 정하고 그에 따라 작업을 번갈아 수행하는 방법이다. 서로 다른 작업들이 실제로 동시에 수행되는 것은 아니지만, 작업 간 전환이 매우 빠르게 이루어지면서 동시에 수행되는 것처럼 보이는 것임.

<br/>

## 여러 작업을 동시에 처리하는게 왜 중요한가?

concurrent mode는 사용자 경험과 굉장히 밀접한 관계가 있다.

### 1. 기존 디바운스와 쓰로틀 한계

input을 입력할 때마다 `무거운 작업` 을 수행하는 경우 버벅임. 디바운스와 쓰로틀을 사용할 수 있다. 하지만 한계도 있음.

- 디바운스: 마지막 입력이 끝난 뒤 일정 시간이 지나 수행하는 방법. 무조건 `일정 시간` 을 기다려야 함.
- 쓰로틀: 주기적으로 무거운 작업을 수행. 쓰로틀 주기를 짧게 가져갈 수록 성능 좋은 기기에서 버벅임이 심해지기 때문.

react 동시성 모드는 위 한계를 동시성으로 해결함. 작업 처리 속도는 개발자가 설정한 delay에 의존되는 것이 아니라 사용자의 기기 성능에 좌우된다.

### 2. 충분히 렌더링이 빠름에도 의미 없는 로딩을 보여주는 경우

suspense로만 구현된 로딩은 이전 페이지를 유저로부터 `차단(block)` 하ㄷ고 다음 페이지의 전체 로딩 화면으로 대체하므로 사용자는 답답함을 느끼게 되었음.

예제에서 페이지 방문하기 버튼을 누르면 이전 페이지를 모두 차단하고 다음 페이지의 전체 로딩 화면을 띄운다. 짧은 로딩은 사용자에게 피로감을 줄 수 있는 불필요한 로딩이 있다. 최대한 줄이는 것이 좋음.

이를 concurrent mode는 일정 시간 동안 현재 페이지와 기능들을 유지하고 다음 페이지에 대한 렌더링을 동시에 진행함으로써 해결한다. 그리고 다음 **페이지의 렌더링 단계가 특정 조건에 부합하면** 해당 페이지를 렌더링한다. 그러면 조건은 뭔가?

<br/>

## Concurrent mode의 동작 원리

> 특정 state가 변경되었을 때 `현 UI를 유지` 하고 해당 변경에 따른 UI 업데이트를 동시에 준비. 준비 중인 UI의 `렌더링 단계` 가 `특정 조건` 에 부합하면 실제 DOM에 반영한다.

현 UI 상태를 유지한다는 것은 크게 어렵지 않으리라 생각함. 그럼 렌더링 단계와 특정 조건은?

![[assets/images/1b1a3f0842e0c587f2d5e974b3cea34f_MD5.png]]

Transition, Loading, Done 총 3개의 렌더링 단계가 있다. 일반적으로 UI 업데이트는 state의 변경에 의해 발생하므로 각 단계는 특정 state 변경 관점에서 보는 렌더링 단계다. 오른쪽으로 진행할 수록 더 최신 렌더링 단계다.

1. `Transition`: state 변경 직후에 일어날 수 있는 UI 렌더링 단계.
   - Pending 상태: 리액트에서 제공하는 `useTransition` 훅을 사용하면 state 변경 직후에도 UI를 업데이트 하지 않고 현 UI를 잠시 유지할 수 있는데, 이를 Pending 상태라고 한다.
   - Receded 상태: useTransition 훅을 사용하지 않은 기본 상태. state 변경 직후 UI가 변경된다. 전체 페이지에 대한 로딩 화면이라고 생각하면 이해가 쉽다. pending 상태에서도 receded 상태로 넘어갈 수 있다.pending 상태의 시간이 useTransition 옵션으로 지정된 timeoutMs를 넘으면 강제로 Receded 상태로 넘어간다.
2. Loading 단계
   - Skeleton 상태 페이지의 일부만을 로딩하는 상태. 전체 화면을 모두 로딩으로 대체해버리는 Receded와는 다르다.
3. Done 단계
   - Complete 상태: 로딩 UI 없이 모든 정보가 사용자에게 보이는 상태.

### 특정 조건

concurrent mode의 원리 중 하나인 `현재 UI를 유지한다` 에 암시된 의미가 특정 조건이란 다음과 같다고 볼 수 있다.

> 특정 state 변경에 대한 현 화면의 UI 렌더링 단계보다 더 `최신 단계` 로 진행하여야 실제 DOM에 반영한다.

예시로, `페이지 방문하기` 버튼을 누르면 즉시 이동하지 않고 잠깐 현 화면을 유지하다 다음 페이지의 Skeleton 상태로 이동한다.

먼저 버튼을 누르면 resource라는 state를 변경하고 Pending 상태에 진입. 이 때 백그라운드에서는 state에 대한 다음 페이지를 준비하는 Receded 상태에 머물러 있을 것이다.

잠시 뒤 다음 페이지의 Receded 상태가 끝나고 Skeleton 상태에 진입하면 그때서야 실제 DOM에 적용한다. Skeleton 상태는 현 UI 상태인 Pending 상태보다 더 최신 단계이기 때문이다.

조금 헷갈릴 수 있는 상황이 있다. Complete 상태로 넘어가고 다른 버튼을 누르면 UI가 멈춰있는 Pending 상태에 진입하고 잠시 뒤 Skeleton 상태로 넘어갈 것 같지만 Skeleton은 등장하지 않고 바로 다른 정보가 렌더링 되는 Complete 상태로 넘어간다.

<br/>

## 도입하기

`useTransition` 훅.

```jsx
// App.js
const [isPending, startTransition] = useTransition({
  timeoutMs: 3000,
}); // 최대 3초 동안 다음 화면의 렌더링을 백그라운드에서 준비하겠다는 의미. 3초가 지나면 강제로 렌더된다.

const handleChange = event => {
  setInput(event.target.value);
  startTransition(() => {
    // 지연할 UI와 관계된 state를 리액트에게 알려줌
    const newArray = array.map((_, index) => TECOBLE[index % TECOBLE.length] + Math.random());

    setArray(newArray);
  });
};
```

setInput은 startTransition 바깥에서 실행되고, setArray는 내부에서 실행되고 있다.

이 차이는 state 우선순위를 결정한다. setArry를 startTransition으로 감싸주는 의지는 'array state' 변화는 지연 시켜도 돼 라고 리액트에게 알려주는 것이기 때문. input state는 array state 보다 높은 우선순위로 결정된다.

이렇게 해주는 이유는 사용자의 입력이 5,000개의 DOM element를 렌더링하는 것보다 우선시 되어야 사용자가 입력할 때의 버벅임을 줄일 수 있기 때문이다. 결과를 확인해보면, 이전 blocking mode로 구현했던 예제보다 훨씬 버벅임이 줄어들었을 것이다.
