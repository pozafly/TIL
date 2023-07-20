# onSuccess, onError, onSettled

> [출처](https://github.com/ssi02014/react-query-tutorial#onsuccess-onerror-onsettled)

```js
const onSuccess = useCallback((data) => {
  console.log("Success", data);
}, []);

const onError = useCallback((err) => {
  console.log("Error", err);
}, []);

const onSettled = useCallback(() => {
  console.log("Settled");
}, []);

const { isLoading, isFetching, data, isError, error, refetch } = useQuery(
  ["super-hero"],
  getSuperHero,
  {
    onSuccess,
    onError,
    onSettled,
  }
);
```

- `onSuccess` 함수는 쿼리 요청이 성공적으로 진행되어 새 데이터를 가져오거나 캐시가 업데이트 될 때마다 실행된다.
- `onError` 함수는 쿼리에 오류가 발생하고 오류가 전달되면 실행된다.
- `onSettled` 함수는 쿼리 요청이 성공, 실패 모두 실행된다.

onSuccess, onError, onSettled 콜백은 react-query v5의 `useQuery` 옵션에서 deprecated 될 예정이다. 단, `useMutation`에서는 사용 가능하다.

> [출처](https://velog.io/@cnsrn1874/breaking-react-querys-api-on-purpose)

Query가 성공적으로 실행되었거나 오류가 발생했을 때, 또는 두 경우 모두에 호출되는 세 가지 콜백 함수 `onSuccess`, `onError`, `onSettled`가 있다. 이 함수를 사용해서 에러 알림 같은 부작용을 실행할 수 있다.

```js
export function useTodos() {
  return useQuery({
    queryKey: ['todos', 'list'],
    queryFn: fetchTodos,
    onError: (error) => {
      toast.error(error.message)
    },
  })
}
```

사용자들은 이 API가 직관적이라 좋아함. 이 콜백이 없었다면 부작용을 일으키기 위해 꺼림칙한 `useEffect` 훅이 필요했을 것이다.

```js
export function useTodos() {
  const query = useQuery({
    queryKey: ['todos', 'list'],
    queryFn: fetchTodos,
  })

  React.useEffect(() => {
    if (query.error) {
      toast.error(query.error.message)
    }
  }, [query.error])

  return query
}
```

많은 사용자들은 더 이상 `useEffect`를 작성할 필요가 없다는 것을 React Query의 큰 장점으로 여김. 위 예제에서는 effect는 `useEffect`를 사용한 접근 방식의 문제점을 확실히 보여준다. 앱에서 `useTodos()`를 두 번 호출하면 두 개의 에러 알림을 받게 되는 것임.

두 컴포넌트 모두 커스텀 훅을 호출하고, 둘 다 effect를 등록한 다음, 해당 컴포넌트 내부에서 실행된다. 하지만, `onError` 콜백의 경우 한 눈에 알 수 없다. 두 호출의 중복이 제거될 것이라 예상 했겠지만 그런 일은 일어나지 않는다. 콜백이 지역 상태 값을 가둬둘 수 있기 때문에 (클로저) **각각의 컴포넌트에서 실행된다**. 한 컴포넌트에서는 콜백을 호출하면서 다른 컴포넌트에서는 안했다면 상당이 일관성이 떨어졌을 테고, 한 컴포넌트에서만 실행된다고 했을 때 둘 중 어느 컴포넌트에서 실행해야 할지 우리가 결정할 수도 없다.

하지만 이 시나리오를 위해 `useEffect`를 작성하도록 두지는 않을 것이다. effect는 두 가지 방식 (useEffect, onError) 모두 잘못 되었다. 이 문제를 해결하는 최고의 방법은 `QueryClient` 를 세팅할 때 전역 캐시 단계에서 콜백을 사용하는 것이다.

```js
const queryClient = new QueryClient({
  queryCache: new QueryCache({
    onError: (error) =>
      toast.error(`Something went wrong: ${error.message}`),
  }),
})
```

이 콜백은 Query당 한 번만 호출된다. 그리고 `useQuery`를 호출한 컴포넌트 외부에 존재하므로 클로저 문제가 없다.

---

트위터에서 이뤄진 토론에서 나온 유일한 좋은 사용 사례는 새 채팅 메시지가 도착했을 때 피드를 하단으로 스크롤하는 사례였다. 이 예시는 fetch가 실패했을 때 아이템에 애니메이션을 적용하는 것과 비슷하게 좋은 예다. 컴포넌트별로 다르게 적용하길 원하는 예시이기 때문. 하지만 이런 사례는 극히 드물고, 원한다면 `useEffect`로도 할 수 있다. `useQuery`가 반환하는 `data`와 `error`는 참조적으로 안정적이므로 effect가 너무 자주 실행되면 어쩌나 하는 걱정없이 쉽게 effect를 설정해도 된다.

API는 간단하고 직관적이며 일관되어야 한다. `useQuery`의 콜백은 이 기준에 부합하는 것처럼 보이도록 위장한 버그 생산기입니다. 처음 구현할 때는 원하는 대로 작동할 가능성이 높지만, 앱이 성장함에 따라 리팩터링 하거나 확장할 때는 대가가 따르기 때문에 아주 안 좋다. 또한 에러가 발생하기 쉬운 상태 동기화를 도입하면서도 이게 나쁘다고 느끼지 않기 때문에 안티 패턴을 초래한다.

거의 모든 예제에서 버그가 발생할 수 있고 이러한 사례가 이슈, 토론, 디스코드, 스택오버플로우 질문에서 정기적으로 보고되는 걸 봤다. 이 버그는 추적하기가 매우 어렵다.

이 API가 애초에 *없는* 게 더 나은 이유이며 이게 바로 저희가 이 API를 없애는 이유다.
