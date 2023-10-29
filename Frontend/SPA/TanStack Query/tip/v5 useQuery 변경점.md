# v5 useQuery 변경점

> [출처](https://velog.io/@cnsrn1874/breaking-react-querys-api-on-purpose)

`useQuery` 에서 콜백을 제거한다.

`onSuccess`, `onError`, `onSettled` 콜백을 제거할 가능성이 높다. RFC 때문이다. -2023.4.15

위 콜백은 `useMutation` 콜백이 아니라, `useQuery` 의 콜백에만 해당한다. 이유는, 예상대로 동작하지 않을 가정성이 높은 나쁜 API 이기 때문이다. API에서 가장 중요한 것은 일관성인데 콜백은 그렇지 않다.

## 나쁜 API

`usseQuery` 에는 Query가 성공적으로 실행되었거나 오류가 발생했을 때, 또는 두경우 모두에 호출되는 세 가지 콜백 함수 `onSuccess`, `onError`, `onSettled` 가 있다. 이 함수를 사용해 에러 알림 같은 부작용을 실행할 수 있다.

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

사용자들은 이 API가 직관적이라 좋아한다. 이 콜백이 없었다면 이런 부작용을 일으키기 위해 꺼림칙한 `useEffect` 훅이 필요할 것이다.

```javascript
export function useTodos() {
  const query = useQuery({
    queryKey: ['todos', 'list'],
    queryFn: fetchTodos,
  })
  
  React.useEffect(() => {
    if (query.error) {
      toast.error(query.error.message)
    }
  }, [query.error]);
  
  return query;
}
```

사용자들은 더 이상 `useEffect` 를 작성할 필요가 없다는 것을 React Query의 큰 장점으로 여긴다. 위 예제에서 effect는 `useEffect` 를 사용한 접근 방식의 문제점을 확실히 보여준다. 바로 앱에서 `useTodos()` 를 두 번 호출하면 두 개의 에러 알림을 받게 되는 것이다.

> 실험 결과 `onError` 콜백에서도 마찬가지로 두 번 실행된다.

`useEffect` 를 사용한 구현을 보면 직관적으로 알 수 있다. 두 컴포넌트 모두 커스텀 훅을 호출하고, 둘 다 effect를 등록한 다음, 해당 컴포넌트 내부에서 실행된다. 하지만 `onError` 콜백의 경우 한 눈에 알 수 없다. 두 호출의 중복이 제거될 것이라 예상했겠지만, 그런 일은 일어나지 않는다. 이게 위에서 말하는 `onError` 도 두 번 실행되는 이유이다.

콜백이 지역 상태 값을 가둬둘 수 있기 때문에(클로저) 각각의 컴포넌트에서 실행된다. 한 컴포넌트에서는 콜백을 호출하면서 다른 컴포넌트에서는 안했다면 상상히 일관성이 떨어졌을테고, 한 컴포넌트에서만 실행된다고 했을 때 둘 중 어느 컴포넌트에서 실행해야 할지 우리가 결정할 수도 없다.

위의 상황이라면, `useEffect`, `onError` 모두 잘못 되었다. 해결 책으로는 Query당 한 번만 호출되는 콜백이 있다. [#11: React Query Error Handling](https://tkdodo.eu/blog/react-query-error-handling#the-global-callbacks)에서 작성한 적이 있다.

이 문제를 해결하는 최고의 방법은 `QueryClient` 를 세팅할 때 전역 캐시 단계에서 콜백을 사용하는 것이다.

```javascript
const queryClient = new QueryClient({
  queryCache: new QueryCache({
    onError: (error) =>
      toast.error(`Something went wrong: ${error.message}`),
  }),
})
```

이 콜백은 Query당 한 번만 호출 될 것이다. 그리고 `useQuery` 를 호출한 컴포넌트 외부에 존재하므로 클로저 문제가 없다.

<br/>

## on-demand 메시지 정의

`error` 내부 자체적인 메시지가 아니라 Query 마다 다른 메시지를 보여주고 싶을 수 있다는 것도 이해한다. 그러기 위해 `queryFn` 의 Promise를 reject 할 때 커스텀 Error를 사용할 수도 있지만, 간단한 해결책은 Query의 `meta` 필드를 사용하는 것이다.

`meta` 는 원하는 어떤 정보로돈 채울 수 있는 임의의 객체인데, 전역 콜백 등 Query에 접근할 수 있는 모든 곳에서 사용할 수 있다.

```javascript
const queryClient = new QueryClient({
  queryCache: new QueryCache({
    onError: (error, query) => {
      if (query.meta.errorMessage) {
        toast.error(query.meta.errorMessage)
      }
    },
  }),
})

export function useTodos() {
  return useQuery({
    queryKey: ['todos', 'list'],
    queryFn: fetchTodos,
    meta: {
      errorMessage: 'Failed to fetch todos',
    },
  })
}
```

이렇게 하면 `meta.errorMessage` 가 정의된 모든 Query는 토스트 알림을 받게 된다. 알림을 보여줘야 하는 useQuery 인스턴스에서 onError를 설정하는 것과 아주 비슷하지만, 그와 달리 안전장치가 되어있는 방법이다.


