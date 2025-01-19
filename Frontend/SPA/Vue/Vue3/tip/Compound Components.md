# Compound Components

> [출처](https://itchallenger.tistory.com/entry/Vue3-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8-%EB%94%94%EC%9E%90%EC%9D%B8-%ED%8C%A8%ED%84%B4-Renderess-Component-Compound-Component)
>
> [볼것](https://books.alexvipond.dev/free), [2](https://adamwathan.me/renderless-components-in-vuejs/), [3](https://velog.io/@bokdol11859/%EB%A6%AC%EC%95%A1%ED%8A%B8-%EA%B0%9C%EB%B0%9C%EC%9E%90%EB%93%A4%EC%9D%80-%EB%AA%A8%EB%A5%B4%EB%8A%94-Renderless-Component)

## 요약

- renderless component는 마크업이 없는 컴포넌트다.
- 상태와 액션만 추상화하는 컴포넌트다.

<br/>

## 좋은 컴포넌트란?

- 결합도가 낮고 응집도가 높은 확장성 있는 컴포넌트
- SOLID 원칙을 잘 지키는 컴포넌트
  - 개발 중 의식해야 하는 부분: 응집도와 결합도

## 좋은 컴포넌트 설계 방법론

- 나쁜 UI 컴포넌트는 너무 다양한 관심사가 섞여있음
  - 비동기 데이터 페치 로직
  - UI 업데이트
  - 상태 관리
  - 마크업 만들기
  - 스타일 적용하기
  - …
- 좋은 컴포넌트란?
  - 하나의 일만 잘하는 컴포넌트
    - 상속 보단 합성
    - 단일 책임 원칙

<br/>

## 좋은 컴포넌트 만드는 방법

### 기본사항

- 컴포넌트 디자인의 가장 구체적 요소는 렌더링이다
  - 즉 마크업 & 스타일 & 레이아웃
  - 뷰로 따지면 template 영역
  - 버튼 / 인풋 같은 아톰 단위 요소가 가장 구체적
- 구체적 요소는 가장 재사용이 떨어진다
  - 슬롯을 통해 의존성을 주입한다
  - **슬롯 외적 부분의 재사용성을 높인다.**
- 구체적 요소를 다루는 기본적 방법
  - 인터페이스 분리 원칙(슬롯)
  - 캡슐화
- [컴포넌트의 시각적 영역이 더 커질수록 오히려 추상적이어야 한다.](https://reactjs.org/docs/composition-vs-inheritance.html)

### Props 설계

- 상태와 옵션에 관한 내용만 props로 받는다.
  - 렌더링에 쓰일 내용 props X
    - 레이아웃을 만들면 안된다.
    - 마크업을 만들면 안된다. (즉 props로 받은 데이터가 슬롯 영역에 나타나면 잘못 만든 것임)

### State 설계

- UI적 상태는 최대한 상단 컴포넌트에서 관리하면 좋음
  - 사이드 이펙트는 상단 컴포넌트로 빼면 좋다(Container component)
  - 퓨어한 자식 컴포넌트들을 합성할 수 있다.
- UI적 상태 또한 컴포넌트 단위로 추상화할 수 있음.
- UI적 상태를 다루는 로직을 컴포넌트 단위로 추상화할 수 있음.
- UI적 상태 + 로직만 포함한 마크업 없는 컴포넌트 => renderless component

### Event / emit 설계

- HTML 이벤트 인터페이스야 말로 가장 잘 알려지고 재사용성 높은 인터페이스임.
  - HTML Element 이벤트를 최대한 직접 이용한다.
- event는 최대한 feature 단위로 설계하는 것이 좋다.
  - ex) 뒤에 나올 "sortable:stop" 이벤트

<br/>
