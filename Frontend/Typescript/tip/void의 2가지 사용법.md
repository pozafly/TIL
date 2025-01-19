# Void의 2가지 사용법

void는 return 값이 없다는 뜻이다. never와는 다르다. never는 값이 없다는 뜻이며 모든 타입의 하위 타입으로 타입 대입이 아예 안되는 녀석이다.

단, void는 `undefined`는 반환할 수 있다. `null`은 안된다.

```ts
function a() {}
const b = a();
```

<img width="274" alt="image" src="https://user-images.githubusercontent.com/59427983/237054520-d41b1d2c-31db-4c38-81f6-8a591bdc8c6f.png">

```ts
function a(): void {
  return;
}
```

이렇게 하면 되는 이유는 `return;`은 undefined를 반환하기 때문이다.

```ts
interface Human {
  talk: () => void;
}
const human: Human = {
  talk() {
    return 3; // 가능
  },
};
```

그런데 이것은 된다. 뭐지?

따라서, 함수에서는 void를 3가지로 기억해야한다.

1. void function: void는 undefined만,
2. void method: return 가능
3. void callback: return 가능
4. void 함수 표현식: return 가능

```ts
// callback
function a(callback: () => void): void {}
a(() => {
  return 3; // 가능
});

// 함수 표현식
type VoidFunc = () => void;
const myFunc: VoidFunc = function () {
  return "hello";
};
const myFunc2: VoidFunc = () => {
  return "hello";
};
```

2, 3, 4에서는 void가 return 값을 사용하지 않겠다는 의미이다. 하지만 function 리터럴의 반환 타입 void는 return 값을 사용하지 않겠다는 의미이다.

```ts
// void와 undefined는 다르다.
// declare function forEach(arr: number[], callback:(el: number) => undefined): void;
declare function forEach(arr: number[], callback: (el: number) => void): void;

let target: number[] = [];
const abab = forEach([1, 2, 3], (el) => target.push(el));
```

콜백의 반환 타입을 undefined로 두었을 경우 `target.push(el)` 부분에서 에러가 난다.

<img width="424" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/3d67dedc-2b96-424e-8ab7-e95979b06874">

number 타입은 undefined에 할당할 수 없다는 말이다.

```ts
interface Human {
  talk: () => void;
}
const human: Human = {
  talk() {
    return 3; // 가능
  },
};

// 마우스를 a에 올렸을 때는 void 타입이다.
const a = human.talk(); // 3
```

즉, forEach와 같은 특수한 경우 때문에 TypeScript에서 허용을 했다. 따라서 JavaScript에서 실제 return 값이 있도록 코드를 짜면 안된다.

```ts
const a = human.talk(); // 3
console.log(a); // 3
const b: void = a;
const c: void = b;
console.log(c); // 3

function returnString(arg: string) {
  return arg;
}
returnString(a); // Argument of type 'void' is not assignable to parameter of type 'string'.
```

`returnString()` 함수에는 void 형식인 c를 매개변수로 넣어주었을 때 타입 에러가 발생할 것이기 때문이다.

만약 정말 불가피한 상황에서는 `as unknown as [원하는타입]` 형태로 넣어주자.

```ts
const a = human.talk() as unknown as string; // 우회
function returnString(arg: string) {
  return arg;
}

returnString(a); // 성공
```

---

정리하자면, 타입스크립트에서 void는 2가지 형태로 사용할 수 있다.

1. 함수 선언문에서 사용하면 return은 존재하지 않거나, undefind만 가능하다
2. 그 외(함수 표현식, 콜백함수, 메서드 축약 문법)에서 사용하면 return에 어떤 타입이 와도 상관은 없지만, 사용하지 않는다는 뜻으로 해석해야 한다.
