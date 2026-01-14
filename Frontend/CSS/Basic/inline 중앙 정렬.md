# Inline 중앙 정렬

> 같이 보기: [centering](https://poiemaweb.com/css3-centering), [css-tricks](https://css-tricks.com/centering-css-complete-guide/)

## 1. Display: table-cell 사용하기

```css
div {
  border: 1px solid black;
  width: 100px;
  height: 80px;
}

span {
  display: table-cell;
  height: inherit;
  vertical-align: middle;
}
```

이렇게 span 태그의 display를 table-cell로 변경하고 높이 값을 부모 태그에 상속받게 해서 똑같이 만들면 span 태그의 높이 값이 상위 태그의 높이 값과 동일하게 된다. 이 때, `vertical-align: middle` 을 사용해서 가운데로 맞추는 것임.

![[assets/images/6af4f9fcfb396ae5024dd737313d541f_MD5.png]]

그럼 딱 이런 모습이 된다. 여기서 만약, 수평 정렬도 하고 싶다고 한다면 할 수 없음. div 태그에 text-align을 넣어줘도 안된다. 왜냐면 이미 span 태그는 display: tabl-cell 이 되었기 때문임.

## 2. Line-height 이용

그러면 span태그의 display: inline으로 유지하면서 가운데 정렬하는 법을 알아보자.

```css
div {
  border: 1px solid black;
  width: 100px;
  height: 80px;
  text-align: center;
}

span {
  line-height: 80px;
}
```

span에 line-height를 줬다. 상위 태그의 height 값 만큼 준 것인데, 행간이 늘어나서 가운데로 수직 정렬된 것을 볼 수 있음. vertical-align은 이용하지 않았음. 그리고 이렇게 되면 상위 태그에서 text-align을 이용할 수 있으므로 center로 주면 아래와 같이 완전 중앙 정렬된다.

![[assets/images/e711cd4ed5725d2ddd32d7659d622cdd_MD5.png]]

단, 이 방법은 div 태그에 height 값이 변동이 생기면 line-height가 따라 움직이는 게 아니기 때문에 세심히 잘 봐줘야 함.

또한, 두줄이 되면 틀어지게 된다.

사실 display flex만 사용하면 해결되긴 함.
