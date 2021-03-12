# form reset

회원가입이나, 여러 정보를 필요로 하는 form 태그에서 일일이 `input = '';` 이렇게 각각 주기보다는 form 태그 자체에서 모두 reset 해주는게 있음.

```jsx
const formRef = useRef();

const onSubmit = (event) => {
  ...
  formRef.current.reset();   // 여기 이부분
}

<form ref={formRef}>
  ...<input....
</form>
```

이런식으로 form에 ref를 걸어 `current.reset()` 만 해주면 된다.

