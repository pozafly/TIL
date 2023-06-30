# ReactNode vs JSX.Element

> [출처](https://www.howdy-mj.me/react/react-node-and-jsx-element)

React 18.0, TypeScript 2.8

## ReactNode vs JSX.Element

ReactNode vs JSX.Element 모두 외부에서 주입받을 컴포넌트의 타입을 정의할 때 가장 많이 사용한다.

### ReactNode

```ts
type ReactNode =
  | ReactElement
  | string
  | number
  | ReactFragment
  | ReactPortal
  | boolean
  | null
  | undefined

type ReactFragment = Iterable<ReactNode>
```

ReactNode는 ReactElement를 비롯해 대부분의 JavaScript 데이터 타입을 아우르는 범용적인 타입이다. 따라서 어떤 props를 받을 것인데, 구체적으로 어떤 타입이 올지 알 수 없거나, 어떠한 타입도 모두 받고 싶다면 ReactNode로 지정해주는 것이 좋음.

ReactText, ReactChild는 React를 사용할 때 큰 관련이 없기 때문에 곧 deprecated 될 것이다.

```tsx
type BlogProps = {
  profile: React.ReactNode
  introduction: JSX.Element
}

const Blog = ({ profile, introduction }: BlogProps) => {
  return (
    <div>
      {profile}
      {introduction}
    </div>
  )
}
export default Blog;
```

```tsx
const App = () => {
  return (
    <Blog
      profile={'howdy-mj'}
      introduction={'howdy-mj'} // TS2322: Type 'string' is not assignable to type 'Element'.
    />
  )
}
export default App
```

`profile`은 ReactNode이고, `introduction` 은 JSX.Element다. 따라서 ReactNode가 아니기 때문에 에러가 뜬다.

ReactNode뿐 아니라, ReactElement도 있다. 공통점은 두 가지 모두 `React.createElement()`의 리턴 값이다.

---

## ReactElement와 JSX.Element

### ReactElement

```ts
export type ReactElement = {|
  $$typeof: any,
  type: any,
  key: any,
  ref: any,
  props: any,
  // ReactFiber
  _owner: any,

  // __DEV__
  _store: { validated: boolean, ... },
  _self: React$Element<any>,
  _shadowChildren: any,
  _source: Source,
|}
```

```ts
interface ReactElement<
  P = any,
  T extends string | JSXElementConstructor<any> =
    | string
    | JSXElementConstructor<any>
> {
  type: T
  props: P
  key: Key | null
}

type JSXElementConstructor<P> =
  | ((props: P) => ReactElement<any, any> | null)
  | (new (props: P) => Component<any, any>)

type ComponentType<P = {}> = ComponentClass<P> | FunctionComponent<P>

type Key = string | number
```

type이 받는 T 제네릭은 해당 HTML 태그의 타입을 받고, props는 그 외의 컴포넌트가 갖고 있는 속성을 받는다.

### JSX.Element

JSX.Element는 ReactElement 타입과 porps를 모두 any로 받아 확장한 인터페이스다. 따라서 더 범용적으로 사용할 수 있다.

```ts
// Global
declare global {
  namespace JSX {
    interface Element extends React.ReactElement<any, any> {}
  }
}

// React Elements
declare namespace React {
  // ... 생략
}
```

또한 React 관련 타입은 모두 React의 namespace에서 선언되었는데, JSX는 global namespace로 선언되어 있다. 따라서 React 내에서 JSX를 import 하지 않아도 바로 사용 가능하다.
