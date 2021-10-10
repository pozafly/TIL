# BoxModel

![image](https://user-images.githubusercontent.com/59427983/136687634-04be9df1-d00d-4800-917a-05901ef99662.png)

이렇게 생긴 녀석이 박스 모델이다. content, padding, border, margin 4가지 영역으로 나누어져 있음.

## TL;DR

- content 영역
  - 너비, 높이는 width, height 를 통해 제어 가능.
- 요소의 전체 크기
  - width, height 그리고, padding, border를 모두 더한 값.
- margin
  - 음수값 사용 가능.
  - 상하 요소 사이의 병합 현상이 일어나고, 큰 값을 기준으로 병합 된다.

<br/>

<br/>

## margin

margin 값은 padding 값과 다르게 **auto** 값이 존재한다. auto 값은 해당 요소에 📌 반드시 **width** 값이 있어야 적용된다.

```css
div {
  width: 300px;
  margin: 0 auto;
}
```

이렇게 해주면 너비 300px에 가운데 정렬이 된다. 마진 값이 양쪽으로 auto 가 들어가는데 자동으로 채워주기 때문임. 하지만 상하의 경우 수직 중앙 정렬이 되지 않음.

<br/>

<br/>

## margin collapse(마진 병함)

마진 병합은 인접한 두 개 이상의 수직 방향 박스의 마진이 하나로 합쳐지는 것을 의미함. 큰 값이 작은 값을 잡아먹어버림.

1. `두 요소가 상하로 인접한 경우` : 위 요소의 하단 마진과 아래 요소의 상단 마진의 병합이 일어남.
2. `부모 요소와 첫 번째 자식 요소 또는 마지막 자식 요소`
   1. 부모 요소의 상단 마진과 첫 번째 자식 요소의 상단 마진 병합이 일어남.
   2. 부모 요소 하단 마진과 마지막 자식 요소의 하단 마진 병합이 일어남.
3. `내용이 없는 빈 요소의 경우` : 해당 요소의 상단 마진과 하단 마진의 병합이 일어남.

마진 병합은 **수직 방향**으로만 이루어짐. 좌우에 대해서는 일어나지 않음. 마진 병합은 마진이 맞닿아야 발생하기 때문에 2,3의 경우 패딩 및 보더가 없어야 함.

![image](https://user-images.githubusercontent.com/59427983/136686862-ed86d5ac-4e0c-49a5-91d7-e44cfa5ba3a9.png)

<br/>

<br/>

## margin & padding

|         |  +   |  -   | auto | 단위  |
| :-----: | :--: | :--: | :--: | :---: |
| margin  |  O   |  O   |  O   | px, % |
| padding |  O   |  X   |  X   | px, % |

⭐️ % 는 위 아래에 적용했을 때도 **가로축 기준**으로 적용된다. % 값은 상위 요소의 가로축이다.

<br/>

<br/>

## width

width 속성은 가로 크기를 정의함. 단, 📌 **content 영역**의 너비를 지정한다. 기본 값은 auto 임.

![image](https://user-images.githubusercontent.com/59427983/136687268-81a29ac6-fdc7-4d80-8ea8-8848762fc6fd.png)

여기를 잘 보면, `width: 300px` 으로 선언했지만 실 값은 10px 이 두번 즉, 320px로 정의 되었다. 왜?

content 영역에만 width 값이 적용되었기 때문이다. border를 10px을 주었기 때문에 width값은 300 이지만, 좌우에 10px씩 border 값이 들어가서 계산 된 것임. padding 값이 들어가게 되면 마찬가지임.

<br/>

<br/>

## height

세로도 마찬가지로 **content** 영역의 높이를 지정하는 것임. default 값은 auto. 컨텐츠 내용만큼 늘어나게 된다. 

📌 단, % 값은 약간 다르다.

```css
.parent {
  border: 10px solid blue;
}
.child {
  height: 50%;
  border: 10px solid red;
}
```

이렇게 height 값을 50% 를 주었다. 어떤 변화가 있을까? **아무런 변화가 없음**. default 값인 auto와 동일하다. containing block 요소를 기준으로 height 값을 잡기 때문인데, parent와 child 사이에 가지는 공간이라고 생각하면 된다.

```css
.parent {
  height: 100px;  // ⚡️추가
  border: 10px solid blue;
}
.child {
  height: 50%;
  border: 10px solid red;
}
```

이렇게 상위 요소에 100px이라는 고정 값을 주게 되면 child에서 %값이 먹히는 모습을 볼 수 있다.

즉, height에서 %를 사용하려면 ⭐️ **상위 요소에 고정 값**이 필요하다.