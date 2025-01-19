# Fragments

React 에서 컴포넌트가 여러 엘리먼트를 반환하는 것은 흔한 패턴임. Fragments는 DOM에 별도의 노드를 추가하지 않고 여러 자식을 그룹화할 수 있다.

```jsx
return (
  <React.Fragment>
    <ChildA />
    <ChildB />
    <ChildC />
  </React.Fragment>
);
```

<br/>

## 동기

만약 `<table>` 태그가 있고 안에 `<tr>` 태그를 다른 컴포넌트로 여러개를 생성해야 한다고 해보자.

```jsx
const Table = () => {
  return (
    <table>
      <tr>
        <Columns />
      </tr>
    </table>
  );
};
```

렌더링 된 HTML이 유효하려면 `<Columns />` 가 여러 `<td>` 요소를 반환해야 함. 만약 Columns 컴포넌트에서 `div` 태그를 가장 상위에 두고 tr을 묶으면 HTML은 유효하지 않게 된다.

```jsx
const Columns = () => {
  return (
    <div>
      <td>Hello</td>
      <td>World</td>
    </div>
  );
};
```

이렇게.

```jsx
<table>
  <tr>
    <div>
      <td>Hello</td>
      <td>World</td>
    </div>
  </tr>
</table>
```

Fragments는 이 문제를 해결해줌.

```jsx
const Columns = () => {
  return (
    <React.Fragment>
      <td>Hello</td>
      <td>World</td>
    </React.Fragment>
  );
};
```

이제 됨.

<br/>

## 단축 문법

```jsx
<React.Fragment>
  <td>Hello</td>
  <td>World</td>
</React.Fragment>

<>
  <td>Hello</td>
  <td>World</td>
</>
```

위의 문법과 아래의 문법은 같다. 단, 빈 태그 문법은 `key`를 지원하지 않는다.

<br/>

## Key가 있는 Fragments

key가 있다면 `<React.Fragments>` 문법으로 명시적으로 선언해줘야 함.

```jsx
return (
  <dl>
    {props.items.map(item => (
      // React는 `key`가 없으면 key warning을 발생시킴.
      <React.Fragment key={item.id}>
        <dt>{item.term}</dt>
        <dd>{item.description}</dd>
      </React.Fragment>
    ))}
  </dl>
);
```
