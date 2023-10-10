# loadable components

> [출처](https://medium.com/greendatakr/loadable-components-881e936aa8fa)

코드 스플리팅이란?

SPA에서 번들 사이즈가 커지면 로딩속도나 성능면에서 문제가 생길 수 있다. 코드 스플리팅은 이것들을 여러 개의 번들로 나누거나 동적으로 import 하는 기법을 말한다.

Loadable Components란?

코드 스플리팅을 도와주는 라이브러리다. React가 자체적으로 제공하는 React.lazy나 React.suspense도 있지만, SSR까지 커버 가능하고 사용방법이 거의 동일한 Ladable Components를 페이스북에서도 추천하고 있다.

```jsx
import loadable from '@loadable/component'
const OtherComponent = loadable(() => import('./OtherComponent'))

function MyComponent() {
  return (
    <div>
      <OtherComponent />
    </div>
   )
}
```

간단 사용 예시다. 이러면 OtherComponent는 동적으로 import 된다. 이말은 컴포넌트가 불러질 때 파일을 읽어온다는 뜻이다. 해당 파일에 접근하는 순간은 속도가 느려질 수 있으나 초기 속도를 개선할 수 있는 이유이다.

코드 스플리팅 할 때 어떤 기준으로 나눌 것이냐에 대한 기준을 세우기가 어려운데, 일단 라우트 기준으로 나누어 보는 것을 많은 사람들과 공식문서에서 추천하고 있다.

또한 라이브러리도 아래와 같이 동적으로 가지고 올 수 있다.

```jsx
import loadable from '@loable/component';
const Moment = loadable.lib(() => import('moment')) // .lib를 사용한다.

function FromNow({ date }) {
  return (
    <div>
      <Moment fallback={date.toLocaleDateString()}>
        {({ default: moment }) => moment(date).fromNow()}
      </Moment>
    </div>
  )
}
```

용량이 큰 라이브러리 등을 초기에 불러오지 않고 필요할 때 불러오는 것은 좋은 방법 중 하나이다. (하지만 초기 로딩과 페이지에서 생기는 로딩 간 사용자 경험을 비교해보는 것이 좋다.)

동적 import 도 가능하다.

```jsx
import loadable from '@loadable/component'
const AsyncPage = loadable(props => import(`./${props.page}`))

function MyComponent() {
  return (
    <div>
      <AsyncPage page="Home" />
      <AsyncPage page="Contact" />
    </div>
  )
}
```

위와 같이 AyncPage에서 받은 Props를 사용해 import 할 수 있다. 만약 babel plugin을 사용하지 않은 경우라면, `cacheKey`를 추가 해주어야 한다.

```jsx
import loadable from '@loadable/component'
const AsyncPage = loadable(props => import(`./${props.page}`), {
  cacheKey: props => props.page, //cacheKey를 추가해준다.
})

function MyComponent() {
  const [page, setPage] = useState('Home')
  return (
    <div>
      <button onClick={() => setPage('Home')}>Go to home</button>
      <button onClick={() => setPage('Contact')}>Go to contact</button>
      {page && <AsyncPage page={page} />}
    </div>
  )
}
```

suspense를 사용해 fallback 처리도 할 수 있다.

```jsx
import React, { Suspense } from 'react'
import { lazy } from '@loadable/component'
const OtherComponent = lazy(() => import('./OtherComponent'))

function MyComponent() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <OtherComponent />
      </Suspense>
    </div>
  )
}
```

suspense 없이도 loadable components에서 fallback처리가 가능하다.

```jsx
const OtherComponent = loadable(() => import('./OtherComponent'))

function MyComponent() {
  return (
    <div>
      <OtherComponent fallback={<div>Loading...</div>} />
    </div>
   )
}
```

