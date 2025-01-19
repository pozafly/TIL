# 정적 내보내기 (Static Exports)

Next.js를 사용하면 정적(static) 사이트 또는 SPA로 시작한 다음, 나중에 서버가 필요한 기능을 사용하도록 선택적으로 업그레이드 할 수 있다.

`next build` 를 실행할 때 Next.js는 경로 별 HTML 파일을 생성한다. 엄격한(strict) SPA를 개별 HTML 파일로 분할해 클라 측에서 불필요한 JavaScript 코드를 로드하지 않도록 한다. 번들 크기를 줄이고 페이지를 더 빠르게 로드할 수 있다.

Next.js는 정적 내보내기를 지원하므로 HTML / CSS / JS 정적 자산을 제공할 수 있는 모든 웹 서버에 배포 및 호스팅 할 수 있음.

<br/>

## 설정

`next.config.js` 파일에 설정한다.

```js
/**
 * @type {import('next').NextConfig}
 */
const nextConfig = {
  output: 'export',
 
  // Optional: Change links `/me` -> `/me/` and emit `/me.html` -> `/me/index.html`
  // trailingSlash: true,
 
  // Optional: Prevent automatic `/me` -> `/me/`, instead preserve `href`
  // skipTrailingSlashRedirect: true,
 
  // Optional: Change the output directory `out` -> `dist`
  // distDir: 'dist',
}
 
module.exports = nextConfig
```

`next build` 를 실행하면 Next.js가 애플리케이션의 HTML/CSS/JS 에셋이 포함된 `out` 폴더를 생성함.

<img width="178" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/7534ee10-1182-43ce-9940-922282ca105d">

여기서 `npx serve@latest out` 명령어를 치면, out 폴더 기준으로 static 파일을 보여준다.

<br/>

## 지원되는 기능

### Server components

`next build` 를 실행해 정적 내보내기를 생성하면 기존 SSG와 유사하게 `app` 디렉토리 내에서 소비되는 server component가 빌드 중 실행된다.

```tsx
// app/page.tsx

export default async function Page() {
  // This fetch will run on the server during `next build`
  const res = await fetch('https://api.example.com/...')
  const data = await res.json()
 
  return <main>...</main>
}
```

### Client Components

클라에서 데이터 패킹을 하려는 경우 SWR이 포함된 클라 컴포넌트를 사용해 요청을 메모화할 수 있다.

```tsx
// app/other/page.tsx

'use client'
 
import useSWR from 'swr'
 
const fetcher = (url: string) => fetch(url).then((r) => r.json())
 
export default function Page() {
  const { data, error } = useSWR(
    `https://jsonplaceholder.typicode.com/posts/1`,
    fetcher
  )
  if (error) return 'Failed to load'
  if (!data) return 'Loading...'
 
  return data.title
}
```

경로 전환은 클라 측에서 이루어지므로 기존 SPA 처럼 동작함. 예를 들어 다음 인덱스 경로를 사용하면 클라의 다른 게시물로 이동할 수 있다.

```tsx
// app/page.tsx

import Link from 'next/link'
 
export default function Page() {
  return (
    <>
      <h1>Index Page</h1>
      <hr />
      <ul>
        <li>
          <Link href="/post/1">Post 1</Link>
        </li>
        <li>
          <Link href="/post/2">Post 2</Link>
        </li>
      </ul>
    </>
  )
}
```

### Image Optimization

`next/image` 를 통한 이미지 최적화는 next.config.js에서 사용자 정의 이미지 로더를 정의해 정적 내보내기와 함께 사용 가능.

```js
// next.config.js

/** @type {import('next').NextConfig} */
const nextConfig = {
  output: 'export',
  images: {
    loader: 'custom',
    loaderFile: './my-loader.ts',
  },
}
 
module.exports = nextConfig
```

### Route Handlers

route handlers는 `next build` 를 실행할 때 정적 응답을 렌더링 한다. `GET` HTTP 동사만 지원된다. 이 동사는 캐시된 데이터 또는 캐시되지 ㅇ낳은 데이터에서 정적 HTML, JSON, TXT 또는 기타 파일을 생성하는데 사용할 수 있다.

```javascript
// app/data.json/route.ts

export async function GET() {
  return Response.json({ name: 'Lee' })
}
```

위의 `app/data.json/route.ts` 파일은 `next build` 중 정적 파일로 렌더링 되어 `{ name: 'Lee' }` 가 포함된 data.json을 생성한다.

들어오는 요청에서 동적 값을 읽어야 하는 경우 정적 내보내기를 사용할 수 없다.

### Browser APIs

클라이언트 컴포넌트는 `next build` 시 HTML로 미리 렌더링됨. window, 로컬 스토리지, navigator와 같은 웹 API는 서버에서 사용할 수 없으므로 브라우저에서 실행할 때만 안전하게 액세스해야 함.

```tsx
'use client';
 
import { useEffect } from 'react';
 
export default function ClientComponent() {
  useEffect(() => {
    // You now have access to `window`
    console.log(window.innerHeight);
  }, [])
 
  return ...;
}
```

<br/>

## 지원하지 않는 기능

빌드 중 계산할 수 없는 동적 로직이나 Node.js 서버가 필요한 기능은 지원되지 않는다.

- [Dynamic Routes](https://nextjs.org/docs/app/building-your-application/routing/dynamic-routes) with `dynamicParams: true`
- [Dynamic Routes](https://nextjs.org/docs/app/building-your-application/routing/dynamic-routes) without `generateStaticParams()`
- [Route Handlers](https://nextjs.org/docs/app/building-your-application/routing/route-handlers) that rely on Request
- [Cookies](https://nextjs.org/docs/app/api-reference/functions/cookies)
- [Rewrites](https://nextjs.org/docs/app/api-reference/next-config-js/rewrites)
- [Redirects](https://nextjs.org/docs/app/api-reference/next-config-js/redirects)
- [Headers](https://nextjs.org/docs/app/api-reference/next-config-js/headers)
- [Middleware](https://nextjs.org/docs/app/building-your-application/routing/middleware)
- [Incremental Static Regeneration](https://nextjs.org/docs/app/building-your-application/data-fetching/fetching-caching-and-revalidating)
- [Image Optimization](https://nextjs.org/docs/app/building-your-application/optimizing/images) with the default `loader`
- [Draft Mode](https://nextjs.org/docs/app/building-your-application/configuring/draft-mode)

`next dev` 에서 이러한 기능을 사용하려고 하면 루트 레이아웃에서 [`dynamic`](https://nextjs.org/docs/app/api-reference/file-conventions/route-segment-config#dynamic) 옵션을 오류로 설정하는 것과 비슷한 오류가 발생함.

<br/>

## 배포

정적 내보내기를 사용하면 HTML/CSS/JS 정적 자산을 제공할 수 있는 모든 웹 서버에 Next.js를 배포하고 호스팅할 수 있다.

`next build` 를 실행할 때 Next.js는 정적 내보내기를 `out` 폴더로 생성합니다. 예를 들어 다음과 같은 경로가 있다고 가정해 보자:

- `/`
- `/blog/[id]`

빌드를 실행하면 Next.js가 다음 파일을 생성함

- `/out/index.html`
- `/out/404.html`
- `/out/blog/post-1.html`
- `/out/blog/post-2.html`

Nginx와 같은 정적 호스트를 사용하는 경우 들어오는 요청을 올바른 파일로 다시 쓰도록 구성할 수 있다.

```
server {
  listen 80;
  server_name acme.com;
 
  root /var/www/out;
 
  location / {
      try_files $uri $uri.html $uri/ =404;
  }
 
  # This is necessary when `trailingSlash: false`.
  # You can omit this when `trailingSlash: true`.
  location /blog/ {
      rewrite ^/blog/(.*)$ /blog/$1.html break;
  }
 
  error_page 404 /404.html;
  location = /404.html {
      internal;
  }
}
```
