# Flex

> 출처: https://youtu.be/7neASrWEFEM

박스와 아이템들을 행, 열로 배치할 수 있음. css의 꽃.

float은 이미지와 텍스트를 어떻게 정렬할 것인지 나누기 위해서 나온 녀석임. 레이아웃을 잡을 때 사용되곤 했는데 float의 순수 목적에는 반하는 것임. FlexBox가 그것을 방지하게 되었음.

<br/>

## 축

Flexbox에는 중심축(Main axis)과 반대축(Cross axis)이 있다. 만약 가로가 중심축이 된다면 반대축은 세로이고, 세로가 중심축이 된다면 가로가 반대축이 되는 것임.

## 속성 적용 목록

Container에 적용할 수 있는 속성 값이 있고, 각 Item들에 적용할 수 있는 속성 값이 있음.

- Container
  - display: Flexbox를 사용하고 싶다면 flex를 주어야 함.
  - flex-direction: 중심축과 반대축을 지정
    - row, row-reverse, column, column-reverse
  - flex-wrap
    - nowrap: 한줄에 적용
    - wrap: item이 한줄에 꽉 차게 되면 다음 줄로 넘어감
    - wrap-reverse: 줄바꿈이 반대로 일어남
  - flex-flow: flex-direction, flex-wrap을 하나로 합친 것
    - ex) flex-flow: column nowrap;
  - justify-content: **중심축** 기준으로 item을 어떻게 배치할 건지 지정
    - flex-start: 중심축에서 처음부터 차례로.
    - flex-end: 중심축에서 반대부터 배치.(단, item의 순서는 유지한다.)
    - center: 가운데 배치.
    - space-around: item 사이에 공간을 주는 것. 첫 요소와 마지막 요소는 1줄이 들어가겠지? 겹치는 건 2줄이 되겠지.
    - space-evenly: 첫 요소 마지막 요소에 상관없이 같은 간격을 준다.
    - space-between: 첫 요소 마지막 요소는 끝에 붙여서 사이에 같은 간격을 줌.
  - align-items: **반대축** 기준으로 item 배치. flex line을 기준으로 아이템 정렬.
    - baseline: 텍스트가 균등한 위치에 두도록 함.
  - align-content: **반대축** 기준으로 item을 어떻게 배치할 건지 지정. flex line을 정렬.
    - justify-content와 속성 값이 같음.
  - ※align-items, align-content 차이

![스크린샷 2021-02-21 오전 10 57 32](https://user-images.githubusercontent.com/59427983/108613276-a7111200-7433-11eb-80ca-379b3cd5289a.png)

    - align-items: justify-content와 반대로 교차 축의 정렬 방법 설정
    - align-content: Container의 Item들이 두줄 이상으로 배치되어 있을 경우 각 줄을 어떻게 배치할 것인지 설정. item이 한 줄로 배치되어 있을 경우 동작하지 않음. 출처: [CSS flex 정리](https://13akstjq.github.io/css/2020/07/01/CSS-flex-%EC%A0%95%EB%A6%AC.html)
      - align-content는 `줄`을 생각하면 됨. 여러 줄들 사이의 간격을 지정하고, align-items는 컨테이너 안에서 어떻게 모든 요소들이 정렬하는지를 지정함.
- Item
  - order: 숫자로 아이템 순서 설정.
  - flex-grow: 창이 늘어날 때 비율.
  - flex-shrink: 창이 줄어들 때 비율.
  - flex-basis: 기본값은 auto, %로 flex-grow, flex-shrink와 상관 없이 늘었다 줄었다 할 수 있다.
  - align-self: 아이템 별로 center 등 justify-content 같이 지정해줄 수 있다.
  - flex: 하나의 플렉스 아이템이 자신의 컨테이너가 차지하는 공간에 맞추기 위해 크기를 키우거나 줄이는 방법을 설정하는 속성. flex는 [`flex-grow`](https://developer.mozilla.org/ko/docs/Web/CSS/flex-grow), [`flex-shrink`](https://developer.mozilla.org/ko/docs/Web/CSS/flex-shrink), [`flex-basis`](https://developer.mozilla.org/ko/docs/Web/CSS/flex-basis)의 단축 속성임. 📌 **item이 하나 밖에 없다면 flex 1을 줘서 모든 공간(여백)을 다 차지하게 할 수 있다.**
    - ex) flex: 1; flex: 2; flex: 1 1 100px;

<br/>

> `정리`
>
> CSS를 사용하면서 가장 편했던 것은 역시 flex인데, 한번 더 정리가 되는 것 같다. item에 먹이는 flex보다는 container에 먹이는 flex를 더 기억하고 있으면 레이아웃 잡기에 참 편하다.
