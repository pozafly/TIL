# Suspense

> [출처](https://velog.io/@tap_kim/react-learn-suspense?utm_source=substack&utm_medium=email)

> ※ `Suspense` 를 지원하는 데이터 원본만 Suspense 컴포넌트를 활성화 한다. 여기에는 다음이 포함된다.
>
> - [Relay](https://relay.dev/docs/guided-tour/rendering/loading-states/) 및 [Next.js](https://nextjs.org/docs/getting-started/react-essentials)와 같은 Suspense 지원 프레임워크를 사용한 데이터 페칭
> - [lazy](https://react.dev/reference/react/lazy)를 이용한 컴포넌트 코드 지연 로딩
> - [use](https://react.dev/reference/react/use)를 이용하는 Promise의 값 읽기
>
> Suspense는 이펙트 또는 이벤트 핸들러 내부에서 데이터를 가져오는 시점을 감지하지 **못한다**.
>
> 위의 `Albums` 컴포넌트에서 데이터를 로드하는 정확한 방법은 프레임워크마다 다르다. `Suspense 지원` 프레임워크를 사용하는 경우 해당 데이터 페칭 문서에서 자세한 내용을 확인할 수 있다.
>
> 독립적인 프레임워크를 사용하지 않는 한 데이터 페칭을 위한 `Suspense 지원` 은 아직 지원되지 않는다. `Suspens 지원` 에 대한 데이터 원본을 구현하기 위한 요구 사항은 불안정하고 문서화 되어 있지 않다. Suspense와 데이터 소스를 통합하기 위한 공식 API는 향후 리액트의 새로운 버전에서 출시될 예정이다.
>
> *[공식 문서 자료 제공](https://react.dev/reference/react/Suspense#displaying-a-fallback-while-content-is-loading)*

Suspense(동시성 렌더링과 함께)는 리액트 [v16.6.0](https://legacy.reactjs.org/blog/2018/10/23/react-v-16-6.html) 부터 제공되던 기능. 하지만, `React.lazy` 와 "Suspense 지원 라이브러리"의 제한된 앱 외에는 실제로 사용되는 것을 자주 보지 못했다.

리액트 v19릴리즈가 임박한 현재, Suspense는 아직 최적기에 사용할 준비가 되지 않았다. API와 내부가 아직 불완전해 보입니다. 사실, 리액트 팀이 Suspense API를 불완전하다고 생각하는 것인지 API에 대한 문서가 전혀 존재하지 않는다. [Suspense 문서](https://19.react.dev/reference/react/Suspense)에서는 Suspense를 사용하는 유일한 방법은 "Suspense 지원 프레임워크"뿐이라고 명시하고 있음.

Suspense 지원 라이브러리를 만들어보자.

<br/>

## 철학

Suspense를 들어가기 전 간단 데이터 페칭 컴포넌트를 만들자.

```tsx
import { useEffect, useState } from 'react';

export default function Artist({ artistName }) {
  const [artistInfo, setArtistInfo] = useState(null);

  useEffect(() => {
    fetch(`https://api/artists/${artistName}`)
      .then((res) => res.json())
      .then((json) => setArtistInfo(json));
  }, [artistName]);

  return (
    <div>
      <h2>{artistName}</h2>
      <p>{artistInfo.bio}</p>
    </div>
  );
}
```

이 코드가 실제로는 좋은 코드가 아니다. 누락된 부분

- 에러 상태 처리
- 대기 상태 처리
- 경쟁 조건 처리
- 공유 캐싱(이 부분은 나중에 다룬다)

구현이 완료되면 컴포넌트는 아래와 같음.

```tsx
export default function Artist({ artistName }) {
  const [artistInfo, setArtistInfo] = useState(null);
  const [error, setError] = useState(null);
  const [isPending, setIsPending] = useState(false);

  useEffect(() => {
    let stale = false;
    setIsPending(true);
    setError(null);

    fetch(`https://api/artists/${artistName}`)
      .then((res) => res.json())
      .then((json) => {
        if (!stale) {
          setArtistInfo(json);
          setArtistInfo(json);
        }
      })
      .catch((err) => setError(err));

    return () => (stale = true);
  }, [artistName]);

  if (isPending) return <SpinnerFallback />;
  if (error) return <ErrorFallback />;

  return (
    <div>
      <h2>{artistName}</h2>
      <p>{artistInfo.bio}</p>
    </div>
  );
}
```

이 방식도 좋지만, 데이터를 가져올 때마다 이 코드를 반복하게 될 것이라고 말하면서 훅을 작성하는 방법을 알려줄거임. 이건 좋은 방법이 아님.

보류 및 에러 상태를 추적하는 것의 문제점은 훅에 이 로직을 묻어두더라도 여전히 **컴포넌트 수준에서 이 상태들을 처리**해야 한다는 것임.

```tsx
export default function Artist({ artistName }) {
  const [artistInfo, isPending, error] = useFetch(
    `https://api/artists/${artistName}`
  );

  // 이 작업은 많지는 않지만 매번 이 작업을 수행해야 함.
  if (isPending) return <SpinnerFallback />;
  if (error) return <ErrorFallback />;

  return (
    <div>
      <h2>{artistName}</h2>
      <p>{artistInfo.bio}</p>
    </div>
  );
}

const Album = ({ albumName }) => {
   const [albumInfo, isPending, error] = useFetch(`https://api/artists/${albumName}`)

  // 이 작업은 많지는 않지만 매번 이 작업을 수행해야 합니다.
  if (isPending) return <SpinnerFallback />
  if (error !== null) return <ErrorFallback />

    return ...
}
```

`useFetch` 가 `SpinnerFallback` 과 `ErrorFallback` 반환을 처리할 수 있다고 상상해보자. `ErrorFallback` 의 경우 [에러 바운더리](https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary)가 있다.

```tsx
export default function Artist({ artistName }: Props) {
  const [artistInfo, isPending, error] = useFetch(
    `https://api/artists/${artistName}`
  );

  if (isPending) return <SpinnerFallback />;
  // 가장 가까운 에러 바운더리에서 핸들링 함. (useFetch로 이동할 수 있음)
  if (error) return <ErrorFallback />;

  return (
    <div>
      <h2>{artistName}</h2>
      <p>{artistInfo.bio}</p>
    </div>
  );
}

const App = () => {
  return (
    <ErrorBoundary fallback={<ErrorFallback />}>
      <Artist artistName="Julian Casablanca" />
    </ErrorBoundary>
  )
}
```

`SpinnerFallback` 의 경우, 사실 이것이 바로 Suspense의 목적임. 혼란을 피하려고 fallback 매커니즘을 트리거 하는 구현은 생략하겠지만, 컴포넌트의 관점에서 보면 다음과 같다.

```tsx
const Artist = ({ artistName }) => {
  // 와우! 이제 훅이 에러와 로딩 상태를 모두 내부에서 처리!
  const [artistInfo] = useFetch(`https://api/artists/${artistName}`)

    return (
    <div>
      <h2>{artistName}</h2>
      <p>{artistInfo.bio}</p>
      <Album albumName={artistInfo.lastAlbumName}>
    </div>
  )
}

const App = () => {
  return (
    <ErrorBoundary fallback={<ErrorFallback />}>
      <Suspense fallback={<SpinnerFallback />}>
        <Artist artistName="Julian Casablanca" />
      </Supense>
    </ErrorBoundary>
    )
}
```

이 시점에서는 `useFetch` 의 구현 코드를 고민하지 마라. 아직 갈 길이 멀기 때문.

<br/>

## 주의: 바운더리 재설정 상태

보류 및 에러 상태를 처리하기 위해 바운더리를 사용할 때의 부작용은 바운더리에 도달하면 모든 자식이 `fallback` 을 위해 버려진다는 것이다. 그 결과 자식 컴포넌트의 상태가 손실된다.

```tsx
const Counter = () => {
  // 에러 바운더리에 도달하면 모든 상태가 초기화됩니다.
  const [count, setCount] = useState(0);
  const [error, setError] = useState<null | Error>(null);

  // 에러 바운더리 트리거
  if (error) throw error;

  const increment = () => setCount((n) => n + 1);
  return (
    <div>
      <p>Counter: {count}</p>
      <button onClick={increment}>Increment</button>
      <button
        onClick={() => {
          // 에러 바운더리를 트리거하려면,
          // 에러는 반드시, 렌더링 주기 중에 발생해야 합니다.
          // 이벤트 핸들러나 이펙트에서 에러를 던지면
          // 에러 바운더리를 트리거하지 않습니다!
          setError(new Error("Whoops something went wrong!"));
        }}
      >
        Throw Error
      </button>
    </div>
  );
};
```

이 방법은 괜찮은 방법이다. 예를 들어 특정 컴포넌트 내 에러가 발생한 경우 이를 제거하는 가장 확실한 방법은 완전히 재설정하는 것이다.

> 팁 : 바운더리를 전략적으로 배치하면 어플리케이션의 어떤 부분이 함께 재설정되는지 제어할 수 있다.
>
> 루트 래핑만 할 수 있는 것이 아니라 모든 컴포넌트를 래핑할 필요도 없다는 점을 기억하자. 이는 에러 바운더리와 Suspense 바운더리 모두에 해당된다.

---

보류 중인 상태를 추적하는 측면에서 이는 어려운 문제다. 초기 렌더링에서 상태는 보류 중이다. 그러면 Suspense 바운더리가 트리거 된다. Suspense 바운더리가 재설정되면 컴포넌트가 다시 마운트되어 새 요청을 전송하고 바운더리를 다시 트리거 함. 따라서 이런 패러다임에서는 동일 요청에 대한 지속적인 참조가 없으면 데이터를 성공적으로 표시할 수 없다.

따라서 컴포넌트의 생명주기보다 오리 지속되는 캐시가 필요함.

<br/>

## 공유 캐시 구축

공유 캐시는 중요함. 동기화 되지 않은 데이터, 너무 많은 요청을 보내는 것을 방지하고, 고급 데이터 관리 기능을 구현하는데 필수적이다. 우리의 경우, 컴포넌트가 자신의 요청 상태를 관리하지 않도록 하여 더욱 단순하게 만드는 것도 중요하다. 그렇지 않으면 컴포넌트는 요청 상태를 잃기 쉽다.

캐싱은 제대로 수행하기 어렵다. 사실, 데이터 페칭 라이브러리 대부분이 캐싱 라이브러리일 정도로 이는 매우 어렵다.

캐시를 의도적으로 단순하게 유지함으로써 '캐시 작성 방법' 이라는 토끼 굴에 빠지는 것을 피할 수 있다. API를 사용하면 특정 URL을 요청할 수 있으며 이전과 마찬가지로 `데이터`, `보류` 중 및 `에러` 에 대한 값을 반환한다. 유일한 차이점은 요청 생명 주기가 요청하는 컴포넌트가 아닌 캐시 내에서 관리된다는 것이다.

그런 다음 캐시를 실제 Suspense 지원 라이브러리로 전환한다.

### FetchCache 클래스

캐시가 갖는 정확한 API 와 기능에 대해 창의적으로 접근할 수 있다.

```js
class FetchCache {
  // 모든 요청을 위한 컨테이너
  requestMap = new Map();
  // 상태 업데이트의 브로드캐스팅을 위한 콜백 추적
  subscribers = new Set();

  fetchUrl(url, refetch) {
    const currentData = this.requestMap.get(url);

    if (currentData) {
      // 이 요청은 이미 진행 중입니다.
      if (currentData.status === "pending") return currentData;
      // 데이터가 이미 캐시에 있고 명시적으로 다시 요청되지 않은 경우 currentData를 반환함
      // 상태는 완료(fulfilled)되었거나 거부(rejected)된 것으로 처리
      if (!refetch) return currentData;
    }

    // 경쟁 요청을 피하려고 상태를 보류 중으로 설정
    const pendingState = { status: "pending" };
    this.requestMap.set(url, pendingState);

    const broadcastUpdate = () => {
      // 실행하지 않기 위해 알림 지연
      // 렌더링 주기 중
      // https://reactjs.org/link/setstate-in-render
      setTimeout(() => {
        for (const callback of this.subscribers) {
          callback();
        }
      }, 0);
    };

    // 요청을 발송하고 관찰하기
    fetch(url)
      .then((res) => res.json())
      .then((data) => {
        // 성공 상태를 캐시에 저장
        this.requestMap.set(url, { status: "fulfilled", value: data });
      })
      .catch((error) => {
        // 에러 상태를 캐시에 저장
        this.requestMap.set(url, { status: "rejected", reason: error });
      })
      .finally(() => {
        // 어떤 일이 발생시 구독자에게 알림
        // 해당 상태가 갱신해야 함을 알림
        broadcastUpdate();
      });

    // 요청이 현재 보류 중임을 보고합니다.
    broadcastUpdate();

    // 요청이 현재 보류 중임을 보고합니다.
    return pendingState;
  }

  subscribe(callback) {
    this.subscribers.add(callback);
    return () => this.subscribers.delete(callback);
  }
}
```

- `fetchUrl(url, refetch)` : url을 요청한다. 캐시에 있는 현재 상태를 반환한다.
- `subscribe(callback)` : 캐시에서 새 데이터를 사용할 수 있을 때 알림을 위한 콜백을 등록한다. 구독을 취소하는 함수를 반환한다.

### FetchCache Provider



















































