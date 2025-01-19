# 평가전략(callBy..)

> 출처: https://velog.io/@jimmyjoo/%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-%ED%8F%89%EA%B0%80%EC%A0%84%EB%9E%B5-Call-By-Value-vs-Call-By-Reference-vs-Call-By-Sharing

<br/>

평가 전략(Evaluation Strategy)이란, 프로그래밍 언어에서 함수 호출의 아규먼트(argument)의 순서를 언제 결정하고 함수에 어떤 종류의 값을 통과시킬지 결정하는 것이다.

## Call By Value

값에 의한 전달.

1. arguments로 값이 넘어온다.
2. 값이 넘어올 때 복사된 값이 넘어온다.
3. caller(호출하는 녀석)가 인자를 복사해서 넘겨줬으므로 callee(호출당한 녀석)에서 해당 인자를 지지고 볶아도 **caller에는 영향을 받지 않는다**.

```js
let a = 1;
function addOne(b) {  // callee
  b = b + 1;
}
addOne(a);  // caller
console.log(a);  // 1
```

`a` 라는 변수를 인수로 넘겨줬다. 이때 1이라는 값은 **복사되어** `b` 에게 할당됨. a와 b의 값은 같지만, 둥 다 다른 메모리 공간을 차지하게 되어 별개의 존재이기 때문에 함수 내부에서 `b` 를 요리해도 `a` 한테는 아무런 영향이 없다.

<br/>

## Call By Reference

주소에 의한 전달.

1. arguments로 reference(값에 대한 참조 주소, 메모리 주소를 담고 있는 변수)를 넘겨준다.
2. reference를 넘기다 보니 해당 reference가 가리키는 값을 복사하지 않는다.
3. caller(호출하는 녀석)가 인자를 복사해서 넘기지 않았으므로 callee(호출당한 녀석)에서 해당 인자를 지지고 볶으면 caller는 영향을 받는다.

```js
const me = { name: 'hst' };
function changeName(person) {  // callee
  person.name = 'foo';
}
console.log(me);  // { name: 'hst' }
changeName(me);  // caller
console.log(me);  // { name: 'foo' }
```

- 매개변수 `person` 에 인수로 넘겨진 `me` 의 참조값이 전달됨. 따라서 me, person은 같은 참조값을 가지고 있음. 따라서, changeName 함수 안에서 person.name을 바꿔주면 me.name도 변한다.
- 즉, 참조 타입을 인수로 넘겨주고 함수 안에서 인자를 변경하면 인자도 변한다!

<br/>

## 또 다른 예시

```js
const me = { name: 'hst' };
function changeName(person) {  // callee
  person = { name: 'foo' };
}
console.log(me);  // { name: 'hst' }
changeName(me);  // caller
console.log(me);  // ???
```

함수에 참조 타입을 인수로 넘겨줬고, 함수 내에서 새로운 객체를 할당했으니까 함수 외부에 있는 객체(`me`)도 변했을 것임. 참조 타입은 Call By Reference니까. 정답은 `{ name: 'foo' }`. 라고 생각할 수 있음.

정답은 `{ name: 'hst' }` 이다. 아무 변화가 없다. 왜?

```js
const me = { name: 'hst' };
```

먼저 객체를 만들고 `me` 라는 변수에 할당했다. 이를 간단한 그림으로 표현하면,

<img width="427" alt="스크린샷 2020-11-02 오후 1 29 29" src="https://user-images.githubusercontent.com/59427983/111278013-6c8a4600-867c-11eb-817a-2fb5152c866b.png">

이렇게 됨. 참조 타입이기 때문에 me라는 변수는 객체의 참조값을 가지게 됨.

다음으로 함수를 정의하고 함수를 호출했다.

```js
function changeName(person) {  // callee
  person = { name: 'foo' };
}

changeName(me);  // caller
```

호출하고 인수가 넘어가는 부분부터 살펴보면, 인수로 `me` 객체를 넘겨줬다. 이때 정확하게 짚고 넘어가야할 부분은, `me` 를 인수로 넘겼지만 사실상 참조된 값이 복사되어 넘어간 것임. 따라서 인수(매개변수)인 `person` 은 복사된 참조값을 가지고 있고, 같은 객체를 가리키고 있다.

<img width="428" alt="스크린샷 2020-11-02 오후 1 28 00" src="https://user-images.githubusercontent.com/59427983/111278414-d99ddb80-867c-11eb-9f53-90b3b167ad7b.png">

마지막으로 함수 내부가 실행된다.

```js
person = { name: 'foo' };
```

위 코드가 실행되어 새로운 객체가 인수인 `person` 에게 할당됨.

아래 그림을 통해 살펴보자. 먼저 새로운 객체(`{ name: 'foo' }`) 가 생성됨. 그 후에 `person` 에게 할당됨. 이를 통해 person이 원래 가지고 있던 me의 참조값(0x111)이 새로은 객체의 참조값(0x222)로 바뀌게 됨. 즉, `me` 와 `person` 은 이제 다른 객체를 참조하고 있다.

<img width="425" alt="스크린샷 2020-11-02 오후 1 22 44" src="https://user-images.githubusercontent.com/59427983/111278784-4dd87f00-867d-11eb-974f-c25dbd3d89bd.png">

즉, 자바스크립트는 Call By Value다. Call By Reference라고 생각했던 부분은, 단지 인수로 넘어갔던 값이 참조타입이었고, 함수 내부에서 변경되면 외부에도 변화가 생기기 때문에 naive 하게 '참조 타입은 Call By Reference'라고 단정지은 것.

<br/>

## 결론

원시 타입은 Call By Value, 참조 타입은 Call By Reference 라고 표현한다. 이런 표현은 값의 종류가 원시타입인지, 참조타입인지를 구별해 강조하는 의미에서 부르는 표현임.

하지만 자바스크립트에서

- 원시 타입은 **Call By Value**
- 참조 타입은 **Call By Value of Reference(Call By Sharing)**

라고 불러줘야함. 결국 변수가 가리키는 메모리 공간에 저장되어 있는 값을 복사하여 전달한다는 관점에서는, `자바스크립트는 항상 값에 의한 전달(Call By Value)만 존재한다고 말할 수 있다.`
