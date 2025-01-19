# hasOwnProperty vs in

## 1. In

```js
// JavaScript in operator.
if ('key' in myObj) {
  ...
}
```

## 2. hasOwnProperty

```js
// hasOwnProperty
if (Object.prototype.hasOwnProperty.call(myObj, "key")) {
  ...
}
```

- `in`: 해당 객체의 prototype chain까지 포함한 모든 객체 키를 조회한다.
- `hasOwnProperty`: 해당 객체가 해당 키를 직접적으로 가질 때만 true를 반환한다.

<br/>

```js
function Person(){};
Person.prototype.eyes = 2;

var kim = new Person();

kim.hasOwnProperty("eyes");  //  false
"eyes" in kim;  //  true
```

위 코드에서 객체 kim은 eyes를 직접 가지고있지 않기때문에 hasOwnProperty 가 false 를 리턴하지만 in 은 true를 리턴한다. 즉, **hasOwnProperty는 객체가 직접 가지고있는 속성만 체크하지만, in 은 프로토타입 링크로 연결된 상위 프로토타입 까지 쭈욱 탐색한다.**

그래서 "toString" in kim 를 한다면 역시 **true**를 리턴한다.
