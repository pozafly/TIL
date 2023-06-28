# Layout, getLayout

> [출처](https://youngble.tistory.com/203), [공식](https://nextjs.org/docs/pages/building-your-application/routing/pages-and-layouts), [출처2](https://www.codeconcisely.com/posts/nextjs-multiple-layouts/)

Layout을 입히기 위해서 `_app.tsx` 파일에 쉽게 레이아웃을 상위 컴포넌트로 선언해 사용할 수 있다.

하지만, getLayout 패턴을 사용하면 아래와 같은 2가지 이점을 추가로 가져갈 수 있다. 이것은 패턴임.

1. 공통으로 걸려있는 레이아웃 대신, 페이지 별로 **대체 레이아웃** 가능
2. 중첩 레이아웃 가능

## 단순 Layout

먼저, 그냥 어떤 패턴 없이 단순 전체 Layout을 걸어보자.

```tsx
// pageLayout.tsx

import { ReactNode } from 'react';
import PageHeader from './PageHeader.tsx';

export default function PageLayout(props: { children: ReactNode }) {
  return (
    <div>
      <PageHeader />
      {props.children}
    </div>
  );
}
```

pageLayout.tsx를 먼저 선언하고, 아래 `_app.tsx`에 넣어주자.

```tsx
// _app.tsx

import PageLayout from '@/features/common/components/PageLayout.tsx';

export default function App({ Component, pageProps }: AppPropsWithLayout) {
  return (
    <PageLayout>
    	<Component {...pageProps} />
    </PageLayout>
  );
}
```

이렇게 하면, 전체 페이지에서 PageLayout이 상위 컴포넌트로 렌더링 된다.

<br />

## 중첩 레이아웃

이제, 각 페이지에 공통 PageLayout이 걸려있는데 그 하위에 **페이지 별로 다른** Layout을 걸고 싶다.

중첩 시키기 위해서는 `getLayout` 함수를 사용할 수 있다. 먼저, `_app.tsx`에서 getLayout 패턴을 먼저 적용해주어야 함.

```tsx
// _app.tsx
import { ReactElement, ReactNode } from 'react';
import type { AppProps } from 'next/app';
import { NextPage } from 'next';
import PageLayout from '@/features/common/components/PageLayout.tsx';

type NextPageWithLayout = NextPage & {
  getLayout?: (page: ReactElement) => ReactNode;
};

type AppPropsWithLayout = AppProps & {
  Component: NextPageWithLayout;
};

export default function App({ Component, pageProps }: AppPropsWithLayout) {
  const getLayout = Component.getLayout ?? (page => page);

  return (
    <PageLayout>
    	{getLayout(<Component {...pageProps} />)}
		</PageLayout>
  );
}
```

해석해보면, getLayout이란 고차 함수를 만들어 Component에서 `getLayout` 함수가 존재하면, page를 리턴하는 함수를 `getLayout` 변수에 담는다는 뜻이다. 그러면, `PageLayout`은 무조건 존재하게 되고, 각 페이지 별 `getLayout` 함수가 존재하면 그 컴포넌트가 렌더링 되도록 하는 것이다.

이제, 어떤 Page에서만 제공할  `SubLayout.tsx` 를 만들자.

```tsx
// SubLayout.tsx

import { ReactNode } from 'react';

export default function SubLayout(props: { children: ReactNode }) {
  return (
    <div>
      SubLayout 입니다.
      {props.children}
    </div>
  );
}
```

그리고 적용할 페이지에서 아래와 같이 `getLayout`을 선언해주면 된다.

```tsx
// SomePage.tsx

import SubLayout from './SubLayout.tsx';

export default function SomePage() {
  return (
    <div>
      some page
    </div>
  );
}

SomePage.getLayout = (page: ReactElement) => {
  return <SubLayout>{page}</SubLayout>;
};
```

그러면 SomePage로 이동했을 때, Layout 중첩 구조를 보자.

```tsx
<PageLayout>
	<SubLayout>
  	<SomePage />
  </SubLayout>
</PageLayout>
```

이런 식으로 렌더링 된다. 하지만, 공통으로 걸려 있는 PageLayout을 대체할 수 는 없을까? PageLayout이 무조건 모든 페이지에서 나오니까 불편하다. `getLayout` 함수가 존재하는 곳에서는 PageLayout 말고 `SubLayout` 만 나오게 할 수는 없을까?

<br />

## 대체 레이아웃

`_app.tsx`를 조금만 수정해주면 된다.

```tsx
// ...
export default function App({ Component, pageProps }: AppPropsWithLayout) {
  const getLayout = Component.getLayout ?? (page => <PageLayout>{page}</PageLayout>);

  return (
    <>
    	{getLayout(<Component {...pageProps} />)}
		</>
  );
}
```

`const getLayout = Component.getLayout ?? (page => <PageLayout>{page}</PageLayout>);` 이 부분이 변경 되었다. page만 렌더링 되던 기존 소스에서, getLayout이 존재하지 않으면 공통 레이아웃인 `PageLayout`이 렌더링 되도록 하고, getLayout 함수가 있는 페이지에서는 PageLayout 대신 페이지에서 정의한 다른 Layout이 나타나도록 처리한 것이다. 

이를 적용한 SomePage를 렌더링 해보면 아래와 같다.

```tsx
<SubLayout>
  <SomePage />
</SubLayout>
```

즉, getLayout 함수가 페이지에 없다면, SubLayout만 나타났다. 또 `_app.tsx`를 위와 같이 선언했지만, 대체 레이아웃이 적용된 페이지에서 PageLayout이 추가적으로 중첩된 구조로 필요할 수도 있다. 그러면 페이지의 getLayout 함수에서 그냥 PageLayout만 선언해주면 된다.

```tsx
// SomePage.tsx
import PageLayout from './PageLayout.tsx';
import SubLayout from './SubLayout.tsx';

export default function SomePage() {
  // ...
}

SomePage.getLayout = (page: ReactElement) => {
  return (
  	<PageLayout>
    	<SubLayout>{page}</SubLayout>
    </PageLayout>
  );
};
```

렌터링 되면 아래와 같이 다시 중첩 구조를 가질 수 있다.

```tsx
<PageLayout>
	<SubLayout>
  	<SomePage />
  </SubLayout>
</PageLayout>
```

간단하게 Next.js에서 설명하고 있는 Layout 패턴에 대해 알아보았다.