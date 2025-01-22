# Focus-within

#:focus-within

css에서:focus-within 의사 클래스가 있다. 주로 아래와 같이 사용함.

```css
form:focus-within {
  background: #ff8;
  color: #000;
}
```

focus-within은 자식 엘리먼트 중 focus를 받게 되면 적용되는 클래스다. 즉, form 태그 하위의 input에 focus를 받으면 위의 css코드가 적용되는 것이다.
