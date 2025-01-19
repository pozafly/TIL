# API 연동 시 useEffect

useEffect를 사용하여 컴포넌트가 처음 렌더링 시점에 API를 요청한다. 여기서 중요한 점은 useEffect 에 등록하는 함수에 async 를 붙이면 안된다. useEffect에서 반환해야 하는 값은 뒷정리(cleanup) 함수이기 때문.

따라서 useEffect 내부에서 async / await를 사용하고 싶다면, 함수 내부에 async 키워드가 붙은 또다른 함수를 만들어서 사용해 주어야 함.

또한, 주로 api 연동 시 loading 이라는 상태도 관리해 API요청이 대기 중인지 판별할 것임.

```jsx
const [values, setValues] = useState(null);
const [loading, setLoading] = useState(false);

useEffect(() => {
  // async를 사용하는 함수 따로 선언
  const fetchData = async () => {
    setLoading(true);
    try {
      const response = await axios.get('....api url');
      setValues(response.data);
    } catch (e) {
      console.log(e);
    }
    setLoading(false);
  };
  fetchData();   // 여기서 내부 함수를 부름.
}, []);
```

이렇게 내부에 함수를 따로 선언해주고, async / await를 붙여준다.

또한, null 처리도 해주어야 하는데

```jsx
// 대기 중일 때
if (loading) {
  return <div>대기 중...</div>;
}
// 아직 values 값이 설정되지 않았을 때
if (!values) {
  return null;
}

...

{values.map((value) => (
  <SomeComponent key={value.id} value={value} />
))}
```

이렇게 하위 컴포넌트에서 map으로 돌릴 시 데이터가 없으면 렌더링 과정에 에러가 나기 때문에 `!values` 이처리를 해주어, api를 받아올 때는 null 처리를 해주어야 한다.
