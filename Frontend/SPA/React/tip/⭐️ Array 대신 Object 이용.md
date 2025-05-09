# Array 대신 Object 이용

배열(cards)이 현재 3개가 있다고 가정하자. 그렇다면 1000개가 되었을 때는? 1000개를 모두 다 불러와서 id를 체크하고 맞으면 추가하는 식으로 계산 할 것이 엄청나게 늘어난다. 즉, 알고리즘을 수행하는 속도는 `늘어나는 배열의 갯수와 동일` 하다. 성능에 좋지 않다. 즉, 이렇게 배열을 처음부터 끝까지 도는 map, for loop 사용금지.

오브젝트의 특징을 이용해서 성능이 좋은 것을 만들 수 있다.

```javascript
const hst = {
  id: '123',
  name: 'hst'
};
hst.id   // 123
hst['id']   // 123
```

이렇게 오브젝트에 `.id` 혹은 `['id']` 를 통해서 동일한 결과 값을 알아올 수 있다. 여기서 javascript의 `computed property`다. 문자열 형태의 키로 동적으로 오브젝트에 접근하기 위해서 사용된다. 이런 Object의 특성을 이용해서, state에 배열을 `Object` 로 바꿔보자.

```jsx
const [cards, setCards] = useState([
  {
    id: '1',
    name: 'Ellie',
  },
  {
    id: '2',
    name: 'Ellie2',
  },
  {
    id: '3',
    name: 'Ellie3',
  },
]);
```

이 녀석을 배열 [] 을 없애고 오브젝트 {} 로.

```jsx
const [cards, setCards] = useState({
  '1': {
    id: '1',
    name: 'Ellie',
  },
  '2': {
    id: '2',
    name: 'Ellie2',
  },
  '3': {
    id: '3',
    name: 'Ellie3',
  },
});
```

이렇게 배열 형식으로 바꿔 주었으니 하위 컴포넌트에서 작성된 map 으로 가져왔던 녀석들을 바꿔주어야 함.

```jsx
{cards.map((card) => (
  <CardEditForm
    key={card.id}
    card={card}
  />
))}
```

이 녀석들을,

```jsx
{Object.keys(cards).map((key) => (
  <CardEditForm
    key={key}
    card={cards[key]}  // 요기
  />
))}
```

이렇게 Object.keys() 메서드로 상위 state의 Key를 가져와서 돌면서 key를 뿌리고 `cards[key]` 를 이용해 card를 내려줄 것이다. 여기서 Object.keys() 는 해당 오브젝트의 key를 문자열로 바꿔서 변환해주므로 javascript의 `computed property` 를 동적으로 받는 것이 되겠지?

전부 바꿔 준 다음 다시 상위 state가 있는 updateCard 함수를 바꿔보자.

```jsx
// 기존 로직
const updateCard = (card) => {
  const updated = cards.map((item) => {
    if (card.id === item.id) {
      return card;
    }
    return item;
  });
};
```

위는 기존 로직이고, 아래는 바꿀 로직이다. map 을 사용하지 않고도 Object의 특성을 살려서 setCards 를 해주었다.

```jsx
const updateCard = (card) => {
  const updated = { ...cards };
  updated[card.id] = card;
  setCards(updated);
};
```

이렇게 되면 map을 사용하지 않고, list를 돌지 않고도 update 되는 모습을 볼 수 있음.

📌 하나더,

다만, setCards인 state를 업데이트 해주는 함수는, 동기적으로 할 수 없을 때도 있다.

위의 코드를 보면, 이전상태 (`{…cards}`) 를 배경으로 해서 값을 변경시키고, 업데이트 했다. 이때, 이전 상태일 수 있는 {…cards}를 기반으로 업데이트 했다는 것인데, 이게 지난 값인지 새로 state가 추가된 상태에서 업데이트 된 것인지 알 수 없는 위험성이 존재한다.

setCards 함수는 `SetStateAction()` 이라는 react 함수를 배경으로 만들어진 것인데, 이것은 이전 상태값을 받아서 새로운 값을 리턴하는 `콜백함수`로도 이용이 가능하다. 위의 코드 기반으로 다시 작성해보면,

```jsx
const updateCard = (card) => {
  setCards(cards => {
    const updated = { ...cards };
    updated[card.id] = card;
    return updated;
  });
};
```

이렇게 된다. setCards 라는 state를 업데이트 시켜주는 함수에, 콜백 함수로 만들어줬다. 이것의 역할은 `업데이트 되는 시점의 cards 를 받아와서` 그 시점에 업데이트 하고 return 해준다는 말이 된다.

위의 로직은 update와 add 에 둘 다 동일하게 적용할 수 있는 로직이므로, 함수의 이름을 updateCard -> `createOrupdateCard` 이렇게 바꿔주고 매핑시켜주자. delete는 간단하다.

```jsx
const deleteCard = (card) => {
  setCards((cards) => {
    const updated = { ...cards };
    delete updated[card.id];   // 넣어주는 것 대신 delete 붙여주기.
    return updated;
  });
};
```

이렇게 delete만 붙여주면 된다.
