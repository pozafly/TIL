# validateDOMNesting

<img width="915" alt="스크린샷 2021-03-30 오후 10 57 56" src="https://user-images.githubusercontent.com/59427983/113001170-844ce880-91ab-11eb-9a30-9672cf0d8d68.png">

개츠비로 블로그를 개설 중 이런 오류가 나타났다. 번역해보자면, **`<p>`는 `<p>`의 하위 항목으로 나타날 수 없습니다.**

## 원인

p 태그 안에 p 태그를 사용하면서 나타나는 오류. p 태그는 블록 요소이고, 인라인 요소만 포함할 수 있다.

```jsx
<p>
  <p style={{ textAlign: 'right' }}><strong>2021/3/30</strong></p>
</p>
```

이렇게 써버린 탓이다.

## 해결

p 태그 안의 p 태그를 div 태그로 바꿔주자. span으로 바꾸지 않는 이유는 text-align이 먹히지 않기 때문인데, block 요소만 text-align 가능. 그리고 p 안에서 div를 사용하면 아까와 같은 오류가 나기 때문에 밖으로 빼주었다.

```jsx
<p>
  ...
</p>
<div style={{ textAlign: 'right' }}><strong>2021/3/30</strong></div>
```

워닝이 싸악 가신다.
