# isomorphic

> [출처](https://toss.tech/article/isomorphic-javascript)

## SSR

SSR은 렌더링 작업 일부를 서버에 위임하는 방식. 브라우저에게 완성된 HTML을 전달한다. 이를 통해 사용자는 빠르게 서비스를 이용할 수 있고, 해당 서비스는 SEO를 통해 더 많은 노출 기회를 얻을 수 있다.

SSR을 위해서 별도의 서버 운영이 필요함. 프레임워크를 사용하는 경우, 서버 구축 및 운영등의 문제는 벗어날 수 있지만 렌더링 과정에 서버가 개입되면서 `window is not defined`와 같은 생소한 에러를 경험한다.

단순하게 생각해보면 서버에서 제공한 HTML을 이용할 뿐인데 왜?

<br/>

## Next.js 렌더링 과정

쿼리 파라미터로 전달받은 유저의 이름을 화면에 출력한다.

```jsx
function App() {
	// 쿼리 파라미터로 전달받은 유저의 이름을 얻어온다.
	const name = new URL(location.href).searchParams.get(name);	
	// 유저의 이름을 화면에 출력한다.
	return <div>{name}</div>
}
```

SSR 환경에서 에러가 발생함.

### 1. 서버가 HTML을 생성한다.

![image](https://github.com/pozafly/TIL/assets/59427983/181c677e-e96a-4ae4-ba9b-e133adbbf45c)

환경은 다음과 같은 특징을 가지고 있음

1. 브라우저 객체인 `location` 이 존재하지 않는다. (location is not defined)

> location은 브라우저 환경에서 제공되는 객체로, URL 관련 속성 및 메소드를 제공한다.

2.  페이지(HTML) 생성이 가능하다. (This error happened while generating the page)

즉 1) 브라우저가 아니면서 동시에 2) 페이지(HTML) 생성이 가능하다는 사실을 통해 서버에서 발생한 에러임을 추측 해볼 수 있다.

서버는 클라에서 제공한 컴포넌트를 기반으로 HTML을 생성. 이 때 만약 클라 환경에만 존재하는 브라우저 객체다. 따라서, 해당 에러를 해결하기 위해서는 서버 환경에서 `location`에 접근할 수 없도록 수정해야 함.

```jsx
function App() {
	const name = (() => {
		/* 서버 환경인 경우, 객체에 접근하지 못하도록 수정 .. */
		if ( isServer() ) {
			return null;
		}
		 return new URL(location.href).searchParams.get(name);	
	})();
	/* 유저의 이름을 화면에 출력한다 ..*/
	return <div>{name}</div>
}
```

### 2. Hydration Mismatch

서버 환경에서 브라우저 객체에 접근할 수 없도록 수정한 후, 새로운 에러가 발생함.

![image](https://github.com/pozafly/TIL/assets/59427983/3bb7b055-5613-4460-ad11-f4a460ba97ad)

위 에러를 해결하기 위해서는 `Hydration`에 대한 이해가 필요함

서버에서 생성한 HTML은 단순 마크업이므로 사용자 인터렉션 불가. 따라서 React는 이벤트 리스너, 상태 관리와 같은 클라 로직을 전달받은 HTML과 통합하여 어플리케이션으로 작동할 수 있도록 한다. 이 과정을 `Hydration` 이라고 함.

주의 깊게 봐야할 점은 로직 연결 과정임. React는 요소 (Element)와 로직 정보가 담긴 가상 DOM을 생성, 이를 전달받은 HTML과 비교한다. 따라서 클라의 렌더링 결과가 같은 경우에만 `Hydration`을 수행할 수 있음.

첫 번째 수정사항으로 서버에서 바라보는 `name` 변수의 값은 항상 `null` 이다. 따라서 쿼리 파라미터가 존재하는 경우, 서버와 클라이언트는 각각 다른 결과물을 렌더링 하게 되면서 `Hydration`을 수행할 수 없는 상태가 된다.

```jsx
// 서버
<div>{null}</div>

// 클라이언트
<div>{'김토스'}</div>
```

<br/>

## 하나의 코드, 동일한 결과 `Isomorphic`

`Hydration Mismatch` 를 해결하기 위해 서버와 클라의 렌더링 결과물이 같이야 함. 이를 위해 서버에서 쿼리 파라미터에 접근할 수 있는 별도 로직 작성이 필요함.

```jsx
function App() {
	const name = (() => {
		if (isServer()) {
	    /* 서버 환경에서 쿼리 파라미터 접근 및 반환하는 별도 로직... */
		}
		 return new URL(location.href).searchParams.get(name);	
	});
	return <div>{name}</div>
}
```

다행히도 Next.js는 개발자가 겪을 불편함을 줄여주고자 `useRouter()` 를 제공하고 있다.

```jsx
import { useRouter } from 'next/router';

function App() {
	const name = useRouter().query.name;

	return <div>{name}</div>
}
```

`useRouter()`를 사용하면 별도의 예외 처리 없이도 서버, 클라 어떤 환경이든 동일한 값을 보장받을 수 있다.ㄴ

```jsx
// 서버
<div>{'김토스'}</div>

// 클라이언트
<div>{'김토스'}</div>
```

이처럼 서버, 클라 양측에 동일한 결과를 보장하는 코드를 `isomorphic` 하다고 함.

요구사항을 보면 쿼리 파라미터 값을 화면에 출력하는 매우 간단한 작업이다. 그러나 SSR 환경에 대한 이해가 없었다면, 에러를 해결하는데 많은 시간을 소비했을 것이다.

`isomorphic` 한 코드는 나와 동료의 시간을 절약해줌. 일관된 결과를 서버와 클라 양측에 보장하기 위해서 관련작업이 반드시 필요함. 따라서, 이런 작업들을 추상화 해둔다면 불필요한 코드들을 감춰지고, 구현에만 집중할 수 있게 된다.

<br/>

## 토스의 isomorphic

1. SSRSuspense

`<Suspense />` 는 비동이 요청을 선언적으로 처리할 수 있도록 돕는 컴포넌트.

- Promise가 대기 상태일 때 (Pending) : `<Loading />`
- Promise가 완료됐을 때 (Resolved) : `<APIRequestComponent />`

```jsx
function App() {
	return (
		<Suspense fallback={<Loading/>}>
			<APIRequestComponent />
		</Suspense>
	)
}
```

그러나 React 18 버전 미만에서는 `<Suspense />`가 오직 클라이언트 환경에서만 정상 작동한다는 한계점이 있다. SSR 환경에서 안정적으로 작동할 수 있도록 `<Suspense />`는 컴포넌트가 마운트 되기 전에는 `fallback` 컴포넌트를 렌더링 함.