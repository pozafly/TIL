# Event Loop

<br/>

> 출처 : https://asfirstalways.tistory.com/362

## Javascript 작동 원리에 대해서

> 싱글 스레드 기반드로 동작하는 Javascript, 
>
> 이벤트 루프를 기반으로 하는 싱글 쓰레드 Node.js

<br/>

## Javascript Engin ?

Javascript를 해석하는 **Javascript Engine**과 웹 브라우저에 화면을 그리는 **Rendering Engine**은 다른 것이다. 

- Rendering Engine(또는 Layout Engine) : HTML과 CSS로 작성된 마크업 관련 코드를 콘텐츠로서 웹 페이지에 `rendering` 하는 역할을 함.
- Javascript Engine : Javascript로 작성한 코드를 해석하고 실행하는 `인터프리터` 임. 주로 웹 브라우저에서 이용되지만 Node.js 가 등장하면서 server side에서 V8과 같은 Engine이 이용됨.

<br/>

구글에서 개발한 V8을 비롯, 대부분 Javascript Engine은 3개 영역으로 나뉜다.

> Call Stack, Task Queue(Event Queue), Heap

그리고 추가적으로 `Event loop` 가 존재하며 Task queue에 들어가는 task들을 관리하게 됨.

![스크린샷 2021-02-19 오후 2 08 09](https://user-images.githubusercontent.com/59427983/108460650-7195e880-72bc-11eb-8fc3-664b70dceb71.png)

<br/>

### Call Stack

Javascript는 **단 하나의 호출 스택**(Call Stack)을 사용함. 이러한 특징 때문에 함수가 실행되는 방식을 `Run to Completion` 이라고 함. 하나의 함수가 실행되면 이 함수의 실행이 끝날 때까지 다른 어떤 task도 **수행 될 수 없다**는 의미. 요청이 들어올 때마다 해당 요청을 **순차적**으로 호출 스택에 담아 처리한다. 메소드가 실행될 때, Call Stack에 새로운 프레임이 생기고 **push** 되고 메소드의 실행이 끝나면 해당 프레임은 **pop**되는 원리.

```javascript
function foo(b) {
	var a = 10;
	return a + b;
}

function bar(x) {
	var y = 2;
	return foo(x + y);
}

console.log(bar(1));
```

1. bar라는 함수를 호출했으니 bar에 해당하는 **스택 프레임이 형성**되고 그 안에는 y와 같은 **local variable**과  **arguments**가 함께 생성됨. 
2. 그리고 bar는 foo를 호출하고 있음. 아직 bar 함수가 종료되지 않으니 pop되지 않고 호출된 foo 함수가 Call Stack에 push 됨.(bar 함수 호출 과정과 동일한 과정을 거친다.)

3. foo 함수에서는 a + b라는 값을 return 하면서 함수의 역할을 모두 마쳤으므로 stack에서 pop 된다. 
4. 다시 bar 함수로 돌아와서 foo 함수로부터 받은 값을 return 하면서 bar 함수도 종료되고 stack에서 pop 됨.

즉 아래 그림과 같이 Stack에 task가 쌓이는 구조.

![스크린샷 2021-02-19 오후 2 46 02](https://user-images.githubusercontent.com/59427983/108463147-3518bb80-72c1-11eb-9edf-c28ff6766c9d.png)



<br/>

### Heap

동적으로 생성된 객체(인스턴스)는 힙(heap)에 할당된다. 여기서 heap이란, 구조화 되지 않은 `더미` 같은 메모리 영역을 말함.

<br/>

### Task Queue(Event Queue)

Javascript의 런타임 환경(Javascript Runtime Environment)에서는 처리해야 하는 Task들을 임시 저장하는 대기 큐가 존재한다. 그 대기 큐를 **Task Queue** or **Event Queue**라고 함. 그리고 Call Stack이 **비어졌을 때** 먼저 대기열에 들어온 수서대로 수행된다.

```javascript
setTimeout(function() {
	console.log('first');
}, 0);
console.log('secone');
```

이 코드가 실행되는 순서는 `setTimeout` 에 `0ms` 를 주었으니 delay 되지 않고 바로 실행될 것 같지만 아니다.

```javascript
// second
// first
```

Javascript에서 비동기로 호출되는 함수들은 Call Stack에 쌓이지 않고 **Task Queue**에  **enqueue** 된다. Javascript에서는 이벤트에 의해 실행되는 함수(핸들러)들이 비동기로 실행된다. 자바스크립트 엔진이 아닌 **Web API 영역**에 따로 정의되어 있는 함수들은 비동기로 실행된다. 

```javascript
function test1() {
	console.log('test1');
	test2();
}

function test2() {
	let timer = setTimeout(function() {
		console.log('test2');
	}, 0);
	test3();
}

function test3() {
	console.log('test3');
}

test1();

// test1
// test3
// test2
```

1. test1()을 호출하면 먼저 test1이 찍힌다. 
2. test2() 호출되면서 setTimeout이 실행되고 Call Stack에 쌓인다음, 빠져나온다. 그리고 내부에 걸려있던 **핸들러(익명함수)**는 **콜스택에 들어가서 바로 실행되지 않는다.** 이 부분 중요⭐️. 이 핸들러는 call stack 영역이 아닌 **Event Queue 영역**으로 들어감. 그리고 test3 함수가 콜스택으로 들어감.
3. test3()가 실행되면서 test3이 console에 찍히고, 작업을 마친 test3 함수가 Call Stack에서 **pop**된다. 
4. 이어 test2(), test1() 함수까지 Call Stack에서 **pop**됨. 이 때 이벤트 루프의 콜스택이 **비어있게 된다.** 
5. 바로 이 시점에 queue의 **heap**에서 하나의 event를 가져와 Call Stack으로 넣는다. 이 이벤트는 setTimeout 함수 내부에 있던 익명함수이다. 이제서야 이 함수가 실행된다.

즉, test3()이 끝나고, (Call Stack에서 pop되고) test2()가 끝나고, test1()이 마저 끝나고 나서 이벤트 루프에 의해 하나의 event가 dequeue 된 다음 콜스택으로 들어가서 실행됨. **그러므로 이벤트에 걸려있는 핸들러는 절대 먼저 실행될 수 없다!!**

