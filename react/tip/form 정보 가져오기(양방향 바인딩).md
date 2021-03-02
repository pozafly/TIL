# form 정보 가져오기(양방향 바인딩)

<br/>

폼 안에 있는 input의 정보를 양방향 바인딩을 사용하려면 react에서는 onChange 이벤트를 통해서 가져와야 한다.

add 할 때는 onChange가 필요없이 submit 버튼을 누르면 ref로 한번에 가져와서 제출하는 방법을 사용했다.

```jsx
const onSubmit = (event) => {
  event.preventDefault();
  const card = {
    id: Date.now(), // UUID
    name: nameRef.current.value || '',
    company: companyRef.current.value || '',
    theme: themeRef.current.value,
    title: titleRef.current.value || '',
    email: emailRef.current.value || '',
    message: messageRef.current.value || '',
    fileName: '',
    fileURL: '',
  };
  formRef.current.reset();
  onAdd(card);
};
```

이런 식으로.

하지만 update 할 때는 컴포넌트끼리 양방향 통신을 해야하므로 onChange() 함수를 사용하여 input value가 변할 때마다 바로바로 UI에 적용되어야 한다.

```jsx
const onChange = (event) => {
  if (event.currentTarget == null) return;   // 값이 없을 때는 빠져나옴.
  event.preventDefault();
  updateCard({
    ...card,   // 불변성 유지.
    [event.currentTarget.name]: event.currentTarget.value,  // key: value
  })
};
```

input 태그에 `onChange={onChange}` 를 먹여주고, 위와같이 onChange 함수를 생성해준다. 그런 후, 상위 state가 존재하는 곳까지 callback 함수로 이어주면 된다.

그리고 나서 input에 값을 쳐보면, onChange함수에 console.log를 걸면 log는 잘 찍히는 걸 볼 수 있는데, 다만 input 태그 UI의 값이 변하지는 않는다. 이유는, state를 변경시켜주지 않고 아직 console.log만 걸어뒀기 때문임.

```jsx
// 상위 state가 존재하는 jsx
const updateCard = (card) => {
  const updated = cards.map((item) => {
    if (card.id === item.id) {
      return card;
    }
    return item;
  });
};
```

이런 식으로 map으로 새 배열을 만들어서 리턴해주곤 했는데 성능상 문제가 생길 수 있다.

이어지는 내용은 [여기] 를 보자.

