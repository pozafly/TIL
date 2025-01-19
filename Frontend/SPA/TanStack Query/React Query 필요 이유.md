# React Query 필요 이유

> [출처](https://velog.io/@cnsrn1874/why-you-need-react-query)

react 앱에서 비동기 상태와 상호작용하는 방식을 간소화한다. 서버로부터 데이터를 가져오는 것만큼 '간단한' 작ㅇ버에는 React Query가 필요없다고 주장하는 글이 있음.

어느 정도는 타당함. React Query는 캐싱, 재시도, 폴링, 데이터 동기화, 프리페칭 등 수 많은 기능을 제공함. 이 기능이 필요없다고 하면 괜찮지만, 그래도 React Query를 사용하지 않는 이유가 되어서는 안된다.

```tsx
// fetch in useEffect
function Bookmarks({ category }) {
  const [data, setData] = useState([])
  const [error, setError] = useState()

  useEffect(() => {
    fetch(`${endpoint}/${category}`)
      .then(res => res.json())
      .then(d => setData(d))
      .catch(e => setError(e))
  }, [category])

  // 데이터와 에러 상태에 따른 JSX 반환
}
```

여기에는 숨어있는 버그가 존재함.

<br/>

## 1. 경쟁 상태

effect는 `category` 가 변경될 때마다 다시 fetch 하도록 한다. 하지만 네트워크 응답은 요청 순서와 다르게 도착할 수 있음. `category`를 `books`에서 `movies`로 변경했는데 `movies`의 응답이 `books`의 응답보다 먼저 도착하면 컴포넌트에 잘못된 데이터가 존재하게 된다.

![image](https://github.com/user-attachments/assets/67350c0a-8f94-4d0c-bd3f-17650baff9f1)

결국 `category` 의 상태는 `movies` 인데 실제 렌더링 하는 데이터는 `books` 인, 일관되지 않은 상태와 함께 하게 된다.

그럼 clean up 함수와 `ignore` 라는 변수로 해결할 수 있다고 한다.

```tsx
// ignore-flag

function Bookmarks({ category }) {
  const [data, setData] = useState([])
  const [error, setError] = useState()

  useEffect(() => {
>   let ignore = false
    fetch(`${endpoint}/${category}`)
      .then(res => res.json())
      .then(d => {
>       if (!ignore) {
>         setData(d)
>       }
      .catch(e => {
>       if (!ignore) {
>         setError(e)
>       }
      })
>     return () => {
>       ignore = true
>     }
  }, [category])

  // 데이터와 에러 상태에 따른 JSX 반환
}
```

이제 category 가 변경되는 effect의 클린업 함수가 실행되고, `ignore` 플래그는 true가 된다. 이후 도착하는 fetch 응답에 대해 더이상 setState를 호출하지 않을 것이다.

<br/>

## 2. 로딩 상태

로딩 상태가 하나도 없다. 요청이 진행되는 동안 보류 중 UI를 표시할 방법이 없다. 첫 번째 요청도, 그 이후에도 없다. 추가해볼까?

```tsx
// loading-state

function Bookmarks({ category }) {
> const [isLoading, setIsLoading] = useState(true)
  const [data, setData] = useState([])
  const [error, setError] = useState()

  useEffect(() => {
    let ignore = false
>   setIsLoading(true)
    fetch(`${endpoint}/${category}`)
      .then(res => res.json())
      .then(d => {
        if (!ignore) {
          setData(d)
        }
      .catch(e => {
        if (!ignore) {
          setError(e)
        }
      })
>     .finally(() => {
>       if (!ignore) {
>         setIsLoading(false)
>       }
>     })
      return () => {
        ignore = true
      }
  }, [category])

  // 데이터와 에러 상태에 따른 JSX 반환
}
```

<br/>

## 3. 빈 상태

`data` 를 빈 배열로 초기화하면 `undefined` 인지 매번 확인하지 않아도 되니 좋은 생각 같다. 하지만 아직 등록된 항목이 없는 카테고리를 fetch 했는데 실제로 빈 배열이 반환되면 어떻게 함? 데이터가 '아직 없는 것'과 '전혀 없는 것'을 구분할 방법이 없을 것이다. 따라서 로딩 상태가 도움이 되지만, 여전히 `undefined` 로 초기화 하는 것이 더 낫다.

```tsx
// empty-state

function Bookmarks({ category }) {
  const [isLoading, setIsLoading] = useState(true)
> const [data, setData] = useState()
  const [error, setError] = useState()

  useEffect(() => {
    let ignore = false
    setIsLoading(true)
    fetch(`${endpoint}/${category}`)
      .then(res => res.json())
      .then(d => {
        if (!ignore) {
          setData(d)
        }
      .catch(e => {
        if (!ignore) {
          setError(e)
        }
      })
      .finally(() => {
        if (!ignore) {
          setIsLoading(false)
        }
      })
      return () => {
        ignore = true
      }
  }, [category])

  // 데이터와 에러 상태에 따른 JSX 반환
}
```

<br/>

## 4. Category가 변경될 때 Data와 Error가 재설정되지 않음

 상태 값이며 cetegory가 변경되어도 재설정되지 않는다. 즉, 카테고리 하나를 fetch 하다 실패하고, 다른 카테고리로 바꿔 성공하면 상태는 아래와 같은 것이다.

```tsx
data: dataFromCurrentCategory // 현재 카테고리의 데이터
error: errorFromPreviousCategory // 이전 카테고리의 에러
```

이러면 결과는 상태에 따른 JSX 렌더링을 어떻게 하냐에 따라 달라진다. `error` 를 먼저 확인하면 유효한 데이터가 있어도 에러 UI와 함께 이전 메시지를 렌더링 할 것이다.

```tsx
// error-first

return (
  <div>
    {error ? (
      <div>Error: {error.message}</div>
    ) : (
      <ul>
        {data.map(item => (
          <li key={item.id}>{item.name}</div>
        ))}
      </ul>
    )}
  </div>
)
```

data를 먼저 확인하면 두 번째 요청이 실패할 때 같은 문제가 발생함. 따라서 `data` 와 `error` 를 매번 재설정해야 한다.

```tsx
// reset-state

function Bookmarks({ category }) {
  const [isLoading, setIsLoading] = useState(true)
  const [data, setData] = useState()
  const [error, setError] = useState()

  useEffect(() => {
    let ignore = false
    setIsLoading(true)
    fetch(`${endpoint}/${category}`)
      .then(res => res.json())
      .then(d => {
        if (!ignore) {
          setData(d)
>         setError(undefined)
        }
      .catch(e => {
        if (!ignore) {
          setError(e)
>         setData(undefined)
        }
      })
      .finally(() => {
        if (!ignore) {
          setIsLoading(false)
        }
      })
      return () => {
        ignore = true
      }
  }, [category])

  // 데이터와 에러 상태에 따른 JSX 반환
}
```

## 5. StrictMode에서 두 번 실행됨

성가신 일이다. 이펙트를 두 번 호출해서 누락된 클린업 함수와 같은 버그를 찾을 수 있게 도와준다.

두 번 실행되는 걸 방지하려면 'ref를 사용한 대안'을 또 추가해야 한느데, 그럴만한 일은 아니다.

## 보너스: 에러처리

fetch는 HTTP 에러가 발생해도 reject를 하지 않아 `res.ok` 를 확인하고 직접 에러를 던저야 한다.

```tsx
// error-handling

function Bookmarks({ category }) {
  const [isLoading, setIsLoading] = useState(true)
  const [data, setData] = useState()
  const [error, setError] = useState()

  useEffect(() => {
    let ignore = false
    setIsLoading(true)
    fetch(`${endpoint}/${category}`)
>     .then(res => {
>       if (!res.ok) {
>         throw new Error('Failed to fetch')
>       }
>       return res.json()
      })
      .then(d => {
        if (!ignore) {
          setData(d)
          setError(undefined)
        }
      .catch(e => {
        if (!ignore) {
          setError(e)
          setData(undefined)
        }
      })
      .finally(() => {
        if (!ignore) {
          setIsLoading(false)
        }
      })
      return () => {
        ignore = true
      }
  }, [category])

  // 데이터와 에러 상태에 따른 JSX 반환
}
```

> **fetch가 에러 응답을 reject하지 않는 이유**
>
> fetch가 이렇게 동작하는 이유에 대해 자세히 알아보려면 [Artem Zakharchenko](https://twitter.com/kettanaito)가 작성한 [이 훌륭한 글](https://kettanaito.com/blog/why-fetch-promise-doesnt-reject-on-error-responses)을 확인해 보라.

---

이렇게 `useEffect` 훅은 예외 케이스와 상태 관리를 고려해야 하게 되자 엄청 더러운 스파게티 코드가 되어버렸다.

-> 데이터 fetching은 간단합니다. 비동기 상태 관리는 그렇지 않습니다.

이게 배울 점임.

여기가 React Query가 개입하는 지점이다. React Query는 데이터 fetch 라이브러리가 아니라(!) 비동기 상태 관리자이니까. 그러니 엔드포인트로부터 데이터를 fetch 하는 것 같은 간단 작업에 React Query를 사용하고 싶지 않다고 한다면 그건 맞다. React Query를 사용한다 해도 똑같은 `fetch` 코드를 작성해야 하니까.

하지만 앱에서 비동기 상태를 예상대로 사용할 수 있게 만드는 걸 최대한 쉽게 하려면 여전히 React Qeury 가 필요하다. React Query를 사용하기 전 `ignore` 불리언 값을 작성해본적 없기 때문이다.

```tsx
// react-query

function Bookmarks({ category }) {
  const { isLoading, data, error } = useQuery({
    queryKey: ['bookmarks', category],
    queryFn: () =>
      fetch(`${endpoint}/${category}`).then((res) => {
        if (!res.ok) {
          throw new Error('Failed to fetch')
        }
        return res.json()
      }),
  })

  // 데이터와 에러 상태에 따른 JSX 반환
}
```

코드 량은 스파게티 코드의 약 50% 이고, 버그 투성이 원본 코드와 거의 같은 양이다. 그리고 이렇게 하면 발젼한 모든 버그가 자동으로 해결된다.

<br/>

## 버그

- 상태는 항상 입력값(카테고리)에 의해 저장되므로 경쟁 상태가 발생하지 않는다.
- 로딩, 데이터, 에러 상태와 타입 레벨에서의 판별(discriminated) 유니언까지 공짜로 제공됨.
- 빈 상태가 명확하게 구분되고 `placeholderData` 같은 기능으로 더욱 향상시킬 수 있음.
- 여러분이 선택하지 않는 한, 이전 카테고리의 데이터나 에러를 받지 않을 것이다.
- `StrictMode` 에 의해 실행된 것도 포함해, 중복 fetch가 효율적으로 제거된다.

<br/>

## 보너스: 취소

```tsx
// cancellation

function Bookmarks({ category }) {
  const { isLoading, data, error } = useQuery({
    queryKey: ['bookmarks', category],
>   queryFn: ({ signal }) =>
>     fetch(`${endpoint}/${category}`, { signal }).then((res) => {
        if (!res.ok) {
          throw new Error('Failed to fetch')
        }
        return res.json()
      }),
  })

  // 데이터와 에러 상태에 따른 JSX 반환
}
```

`queryFn`에 들어가는 `signal`을 `fetch`에 넘기면 카테고리 변경 시 요청이 자동으로 중단된다.
