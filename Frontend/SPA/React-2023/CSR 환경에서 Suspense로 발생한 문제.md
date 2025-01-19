# CSR 환경에서 Suspense로 발생한 문제

> [출처](https://tech.kakaopay.com/post/react-router-dom-csr-prefetch/)

CSR에서 Suspense를 통해 선언적 로딩 처리를 할 때 순차적 API 호출이 발생하고 있었는데, 이를 해결하면서 30% 정도 성능 개선 효과를 얻을 수 있다.

## React-router-dom 기반 CSR 라우팅

react-router-dom v6 부터 `RouterProvider` 와 `createBrowserRouter` 를 이용해 객체 형태로 라우팅을 설정할 수 있음.

```jsx
const router = createBrowserRouter([
  {
    path: '/',
    element: <Home />,
  },
]);

<RouterProvider router={router} />;
```

또한 `lazy` 함수를 이용 code splitting을 설정할 수 있음. 각 페이지 번들을 지연 로딩하므로 해당 페이지가 로딩될 fallback 컴포넌트를 Suspense를 통해 설정할 수 있음. 복잡한 코드 없이 코드 분할을 통해 번들 크기를 줄이고 초기 로딩 속도를 줄일 수 있는 효과를 얻을 수 있음.

[lazy](https://ko.react.dev/reference/react/lazy)

```jsx
import { lazy, Suspense } from 'react';

const LazyHome = lazy(() => import('./Home'));

const router = createBrowserRouter([
  {
    path: '/',
    element: (
      <Suspense fallback={<Loading />}>
        <LazyHome />
      </Suspense>
    ),
  },
]);
```

하지만, 실제 서비스에서는 더 복잡하다. 정적 콘텐츠를 보여줄 때도 있지만, 서버와의 통신을 위해 동적 데이터를 보여줄 때도 있기 때문에 이 과정에서 서비스의 성능이 저하되는 경우도 존재함.

<br/>

## @tanstack/react-query와 함께 Suspense를 사용했을 때의 문제점

suspense를 사용하면서 하나의 컴포넌트에서 여러 개의 데이터 호출이 이루어질 경우 순차적으로 API 호출이 이루어질 수 있다. 아래 검코넌트는 두 개의 `useQuery` 를 사용하는데, suspense 옵션을 켜게 되면 첫 번째 쿼리가 내부적으로 Promise를 발생시키고 다른 쿼리가 실행되기 전 컴포넌트를 일시 중단하기 때문에 순차적으로 API 호출이 발생하게 된다.

```jsx
function Home() {
  const { data: userData } = useQuery({
    queryKey: ['USER'],
    queryFn: getUser,
    suspense: true,
  });
  const { data: bannerData } = useQuery({
    queryKey: ['BANNER'],
    queryFn: getBanner,
    suspense: true,
  });
  const { isCardUser } = userData ?? {};
  const { src } = bannerData ?? {};
  return (
    <div>
      {/** 다른 컴포넌트 ... */}
      {isCardUser && <Banner src={src} />}
      {/** 다른 컴포넌트 ... */}
    </div>
  );
}
```

네트워크 탭을 보면 API 요청이 순차적으로 발생하고 있음.

![image](https://github.com/pozafly/TIL/assets/59427983/c3e90c13-cbdc-4d93-a7f9-e5652a8fe2b9)

### Suspnse 옵션을 사용하지 않는 컴포넌트

useQuery를 사용하는 부분의 suspense 옵션을 제거하면 두 개의 쿼리가 병렬로 실행된다.

```jsx
function Home() {
  const { data: userData, isLoading: isUserLoading } = useQuery({
    queryKey: ['USER'],
    queryFn: getUser,
  });
  const { data: bannerData, isLoading: isBannerLoading } = useQuery({
    queryKey: ['BANNER'],
    queryFn: getBanner,
  });

  if (isUserLoading || isBannerLoading) {
    return <Loading />;
  }
  // ...
}
```

suspense 옵션을 비활성화하면 순차적 API 호출은 제거할 수 있지만 기존 선언적 컴포넌트 구성에 대한 장점은 이용할 수 없게 되었다.

![image](https://github.com/pozafly/TIL/assets/59427983/92191fdd-990d-4b73-ae43-646628badd86)

<br/>

## 순차적 API 호출의 문제점

예시처럼 적은 수의 API 호출할 때는 큰 문제 없더라도, 호출하는 API의 수가 많을 경우 순차 호출로 비효율이 증가할 것이다.

![image](https://github.com/pozafly/TIL/assets/59427983/df0e3dd1-3c91-4e2b-92de-d58f2baaa1ac)

suspense를 사용하고 있으므로 화면 전체 렌더링이 끝나는 시점은 순차적으로 호출된 API 중 마지막 호출된 API 응답이 오는 시점 이후가 될 것이다.

<br/>

## 순차적 API 호출을 제거할 수 있는 방법

### 1. @tanstack/react-query의 useQueries

> v5 부터는 `useSuspenseQueries` 로 변경되었다. [링크](https://tanstack.com/query/v5/docs/react/reference/useSuspenseQueries)

```jsx
function Home() {
  const [{ data: userData }, { data: bannerData }] = useQueries({
    queries: [
      { queryKey: ['USER'], queryFn: getUser, suspense: true },
      { queryKey: ['BANNER'], queryFn: getBanner, suspense: true },
    ],
  });
  // ...
}
```

useQuery 훅에 전달하던 매개 변수들을 `queries` 인자에 배열 형태로 전달하면 `useQueries` 훅을 이용할 수 있음.

![image](https://github.com/pozafly/TIL/assets/59427983/d147d6f8-8f84-45ec-bc0a-e70eaa9ca924)

### 2. React-router-dom의 loader

> `loader` 는 데이터 라우터를 사용할 경우에만 사용할 수 있다. [링크](https://reactrouter.com/en/main/routers/picking-a-router)

```tsx
import type { LoaderFunctionArgs } from 'react-router-dom';

// EXAMPLE: /card/:cardId
async function loader({ params }: LoaderFunctionArgs) {
  const { cardId = '' } = params;
  return getCardData(cardId);
}
```

`loader` 함수는 `params` 를 포함하는 객체를 인자로 받고 동적 라우팅을 사용할 경우 `params` 를 통해 Path 파라미터 값을 전달받을 수 있음.

> `params`에 대한 자세한 내용은 [React Router 공식 문서](https://reactrouter.com/en/main/route/loader#params)를 참고.

또한 Path 파라미터가 아닌 Search 파라미터를 이용, 데이터를 가져와야 할 경우 아래와 같이 request 인자를 사용할 수 있다.

```tsx
import type { LoaderFunctionArgs } from 'react-router-dom';

// EXAMPLE: /card/:cardId/list?month=2023-11
async function loader({ request, params }: LoaderFunctionArgs) {
  const { cardId = '' } = params;
  const searchParams = new URL(request.url).searchParams;
  const month = searchParams.get('month');
  return getCardSpentListByMonth(month);
}
```

> `request`에 대한 자세한 내용은 [React Router 공식 문서](https://reactrouter.com/en/main/route/loader#request)를 참고.

loader 함수에 반환된 데이터는 컴포넌트에서 `useLoaderData` 훅을 이용해 사용할 수 있음.

> `useLoader` 훅의 반환타입은 `unknown` 이므로 Typescript 사용할 경우 `as` 를 이용해 타입 단언을 함께 사용하면 편리함.

```jsx
function Home() {
  const data = useLoaderData();
}
```

이제 `loader`를 통해 순차적 API 호출을 제거하는 방법을 살펴보자. @tanstack/react-query에서는 쿼리의 타입에 따라 `prefetchQuery` 또는 `prefetchInfiniteQuery`를 이용해 데이터를 prefetch 할 수 있다. 지금은 하나의 `Suspense`를 이용해 전체 로딩을 처리하고 있으므로 여러 쿼리들을 하나의 `Promise`로 묶어주기 위해 추가적으로 `Promise.all`을 사용한다.

```jsx
import { queryClient } from '~/somewhere';

async function loader() {
  return await Promise.all([
    queryClient.prefetchQuery({ queryKey: ['USER'], queryFn: getUser }),
    queryClient.prefetchQuery({ queryKey: ['BANNER'], queryFn: getBanner }),
  ]);
}

const router = createBrowserRouter([
  {
    path: '/',
    loader,
    element: (
      <Suspense fallback={<Loading />}>
        <LazyHome />
      </Suspense>
    ),
  },
]);
```

이대로 코드를 사용하면 기존 코드의 구조를 유지하며 데이터를 병렬 호출을 할 수 있다. 하지만 지금의 코드로는 기존에 이용하던 `Suspense`의 fallback 요소가 정상적으로 렌더링 되지 않는다. 정상적으로 fallback 요소를 렌더링 하기 위해서는 react-router-dom에서 제공하는 `defer` 함수를 이용해 `loader`의 반환값이 지연된 값임을 알려야 함.

```js
import { defer } from 'react-router-dom';

function loader() {
  return defer({
    data: Promise.all([
      queryClient.prefetchQuery({ queryKey: ['USER'], queryFn: getUser }),
      queryClient.prefetchQuery({ queryKey: ['BANNER'], queryFn: getBanner }),
    ]),
  });
}
```

또한 react-router-dom에서 제공하는 `Await` 컴포넌트를 이용해야 함. `Await` 컴포넌트는 `useLoaderData`가 반환하는 `Promise`가 `resolve`될 경우 fallback 요소를 언마운트 하고 하위 요소를 렌더링 함.

> `defer` 함수와 `Await` 컴포넌트에 대한 자세한 내용은 [React Router 공식 문서](https://reactrouter.com/en/main/guides/deferred)를 참고.

```jsx
import { Await, useLoaderData } from 'react-router-dom';

function Home() {
  const { data } = useLoaderData() as { data: Promise<void[]> };
  const { data: userData } = useQuery({
    queryKey: ['USER'],
    queryFn: getUser,
    suspense: true,
  });
  const { data: bannerData } = useQuery({
    queryKey: ['BANNER'],
    queryFn: getBanner,
    suspense: true,
  });
  const { isCardUser } = userData ?? {};
  const { src } = bannerData ?? {};
  return (
    <Await resolve={data}>
      <div>
        {/** 다른 컴포넌트 ... */}
        {isCardUser && <Banner src={src} />}
        {/** 다른 컴포넌트 ... */}
      </div>
    <Await>
  );
}
```

`Await` 컴포넌트와 `useLoaderData` 훅을 추가적으로 이용해야 하는 다소 번거로운 방법이지만, 기존의 코드 구조를 유지한 상태로 데이터를 빠르게 가져올 수 있게 되었음. 그렇다면 해당 기능을 적용한 서비스는 어느 정도의 성능 개선 효과를 얻을 수 있었을까?
