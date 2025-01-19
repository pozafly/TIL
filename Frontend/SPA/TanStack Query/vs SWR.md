# Vs SWR

> [출처](https://yanggoon.dev/showcase/swr-query)

## 각 라이브러리의 의도

어떤 문제를 해결하기 위해 만들어졌는가?

### SWR

> SWR is a strategy to first return the data from cache (stale), then send the fetch request (revalidate), and finally come with the up-to-date data.
>
> SWR은 stale: 캐시에서 데이터를 찾아 먼저 반환하고, revalidate: 데이터 가져오기 요청을 보내어, 결과적으로 최신의 데이터를 사용하기 위한 전략입니다.

마지막 표현이 핵심인데, SWR이 리액트에서 사용된다는 점을 생각하면, SWR은 '최신의 데이터를 사용해 렌더링' 하기 위한 목적을 가진다. 하지만 놓쳐서는 안되는 **결과적으로** 라는 표현을 곱씹어보면, 최신의 데이터를 사용하는 것보다 빠르게 캐시 데이터로 렌더링을 하는 것에 더 높은 우선순위를 두었다는 의미다.

## React Query

> React Query … makes fetching, caching, synchronizing and updating server state in your React applications a breeze.
> …
>
> React Query는 리액트 어플리케이션에서 서버 상태를 가져오고, 캐싱하고, 동기화하며, 업데이트하는 작업을 손쉽게 만들어주는 라이브러리입니다.

이 표현의 핵심은 **Server State** 인데, 이게 대체 뭘까?

> Is persisted remotely in a location you do not control or own
> 당신이 컨트롤할 수 없는 영역에 영속성을 가지고 존재하며
>
> Requires asynchronous APIs for fetching and updating
> 데이터를 읽고, 쓰려면 비동기 API가 필요하며
>
> Implies shared ownership and can be changed by other people without your knowledge
> 당신이 모르는 새에 다른 사람들에 의해 값이 바뀔 수도 있으며
>
> Can potentially become "out of date" in your applications if you're not careful
> 주의해서 관리하지 않으면 "구식" 정보가 되어버릴 가능성이 존재하는

그런 상태를 Server state라고 부른다. 백엔드 서버 너머에 존재하는 Database를 상상하면 될 것 같다. 그런데 이게 왜 나왔을까? Frontend 영역에서 Database를 관리해야 한다는 이야기일까? 라고 생각하면 또 바로 다음 문단에서 우리의 궁금증을 해소해준다.

> Caching
> 렌더링을 빠르게 하려면 데이터를 캐싱해둬야 할 것이고
> (사실 이 데이터 캐싱이 아래 대부분의 문제를 불러일으키는 원인이다)
>
> Deduping multiple requests for the same data into a single request
> 여러 컴포넌트에서 동일한 데이터가 필요하다면 하나의 요청으로 처리해야 하며
> (각 컴포넌트별로 동일한 request를 마구마구 쏘아보내는 상황을 피하자)
>
> Updating "out of date" data in the background
> 캐싱했던 데이터가 "구식" 데이터가 되지 않게 업데이트도 해 줘야 하는데
>
> Knowing when data is "out of date"
> 그 "구식"이 되는 시점이 언제인지도 파악해야 하고
>
> Reflecting updates to data as quickly as possible
> 최대한 빠르게 업데이트를 반영하고 싶을 텐데
>
> Performance optimizations like pagination and lazy loading data
> 페이징이나 레이지로딩 같은 최적화도 적용해야 하고
>
> Managing memory and garbage collection of server state
> 캐싱하고 있던 데이터가 메모리를 넘칠 정도로 많아지지 않게 메모리 관리까지 필요할 수도 있고
>
> Memoizing query results with structural sharing
> structural sharing도 놓칠 수 없지
> (이 용어가 익숙하진 않겠지만, 데이터의 일부에만 변화가 생겼을 때, 새 객체를 만들지 않고 기존 객체들을 재사용하는 방식인데
> 리듀서를 작성할 때, 우리가 왜 `{ …state, …updatedData }` 처럼 destructing을 했었는지를 떠올려보자)

요약하면, redux에서 store 쓰듯, API 호출을 줄이기 위해 API를 통해 불러온 데이터를 한 번만 불러와 저장해두고 반복해서 쓰려고 할 때 그 저장된 데이터를 server state 라고 부르는 것이고, 이 데이터들을 관리할 때 꽤 많은 문제가 발생할 수 있다는 말을 아주 길게 써놓은 것이다.

그러니까 React Query는 **API 호출로 받아온 데이터를 쉽게 관리할 수 있도록 지원**해주는 라이브러리다.

<br/>

## 비교

SWR은 빠르게 렌더링 하는데 필요한 데이터를 제공하는 것에 초점을 맞추고 있고, React Query는 API 호출로 받아온 데이터를 관리하는 것에 초점을 맞추고 있다. 얼핏보면 같은 일을 동일하게 할 수 있는 라이브러리인데, 이 의도의 차이가 어떤 부분에서 차이를 가지는지 알아보자.

### 캐싱

각자 목적을 달성하기 위해 가장 중요하게 생각하는 부분은 '캐싱' 이다.

한 번 불러운 데이터를 서로 다른 컴포넌트의 렌더링에 반복해서 사용하기 위해서는 어딘가에 저장해야 하고, 이 데이터를 반복에서 꺼내쓸 수 있어야 한다. 이 때, ReactQuery에서 지적했던 것처럼, 개발자에게는 몇 가지 고민거리가 같이 생기는데, 주목할만한 주제를 뽑자면 다음과 같음.

> **Deduping**: Deduping multiple requests for the same data into a single request
> 여러 컴포넌트에서 동일한 데이터가 필요하다면 하나의 요청으로 처리해야 하며
>
> **Revalidating**: Updating "out of date" data in the background
> 캐싱했던 데이터가 "구식" 데이터가 되지 않게 업데이트도 해 줘야 하는데
>
> **Stale**: Knowing when data is "out of date"
> 그 "구식"이 되는 시점이 언제인지도 파악해야 하고

### Deduping

- 두 라이브러리가 모두 지원하는가?
  - 모두 지원.
  - SWR: `dedupingInterval` 옵션이 기본 2초로 설정. 이 시간 동안 들어온 요청만 Deduping 처리.
  - React Query: 하나의 request가 끝나기 전까지 들어온 요청을 자동 deduping 해준다.
- React Query는 dedupingInterval 옵션이 없나?
  - ReactQuery는 `staleTime` 옵션을 제공하고 있다. 받아온 데이터의 유통기한이 얼마인가를 지정하는 값이고, 이 시간이 지나기 전에 데이터를 읽어가려고 할 때에는 아직 유통기한이 남은-캐싱된- 데이터를 반환한다. 따라서 deduping은 데이터를 받아오기 전 까지만 처리하면 된다-는 개념이다.
