# text-align

정렬 css 속성이다. 주로 가운데 맞출 때 적용하는데,

```css
.title {
  text-align: center;
}
```

와 같이 사용한다.

이게 먹히는 조건이 있는데 상위 요소가 `display: block` 일 경우 먹힘.

그리고 적용하고자 하는 요소가 `display: inline` 이면 안먹힘.

따라서 `position: absolute` 를 사용하면 상위 요소 기준이 없으므로 text-align이 먹히지 않게 됨. 상위 요소에 `position: relative` 를 줘도 안됨.

<br/>

주의 사항

1. block 요소에만 text-align 속성을 적용할 수 있다. -> 하위 요소들에게 정렬이 적용됨.
2. 정렬되는 것은 block 요소 안에 있는 inline 요소만 가능함.
3. 2번에서 inline 요소란, text 뿐 아니라 이미지 등도 포함됨.