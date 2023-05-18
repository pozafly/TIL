# setInterval, setTimeout

> 출처 : https://offbyone.tistory.com/241

자바스크립트 주기적인 작업을 실행하기 위해 setInterval, setTimeout 메소드를 사용할 수 있다.

- 차이점
  - `setInterval` : 일정한 시간 간격으로 작업을 수행하기 위해 사용함. `clearInterval` 함수를 사용해 중지할 수 있음. 주의할 점은 일정한 시간 간격으로 실행되는 작업이 그 시간 간격보다 오래걸릴 경우 문제가 발생할 수 있음.
  - `setTimeout` : 일정한 시간 후에 작업을 한번 실행. 보통 재귀적 호출을 사용해 작업을 반복한다. 기본적으로 **setInterval** 과는 달리 지정된 시간을 기다린 후 작업 수행, 다시 일정한 시간을 기다린 후 작업. 지정된 시간 사이에 작업 시간이 추가 되는 것이다. `clearTimeout` 함수를 사용해 작업을 중지함.
- clearInterval(), clearTimeout() 이 실행중인 작업을 중지시키는 것은 아니다. 지정된 작업은 모두 실행되고 다음 작업 스케줄이 중지 되는 것임.

<br/>

## 1. setInterval 사용 예

```jsx
function PrintTime() {
  var today = new Date();
  var hh = today.getHours();
  var mi = today.getMinutes();
  var ss = today.getSeconds();
  document.getElementById("result").innerHTML = hh + ":" + mi + ":" + ss;
}

// 중지를 위해 ID 보관
var timerId = null;

// 시계 시작
function StartClock() {
  PrintTime();
  timerId = setInterval(PrintTime, 1000);
}

// 시계 중지
function StopClock() {
  if(timerId != null) {
    clearInterval(timerId);
  }
}

...

<!--  시계가 보여질 div -->
<div id="result"></div>

<input type="button" value="시작" onclick="StartClock();" />
<input type="button" value="중지" onclick="StopClock();" />
```

시작 버튼을 누르면 StartClock() 함수에서 `timerId = setInterval(PrintTime, 1000);` 을 호출해 1초 간격으로 PrintTime() 함수를 호출해 현재 시간을 출력한다. 이때 반환 값을 timerId 변수에 저장하고 후에 중지하는데 사용함.

setInterval 을 호출하기 전 PrintTime()을 한번 호출하는 이유는 setInterval 은 지정된 시간 후에 함수를 실행하기 때문에 '시작' 버튼을 누른 후 1초 후 출력되기 때문. 누르자마자 한번 출력하기 위해 먼저 호출.

<br/>

## 2. setTimeout 사용 예

```js
//시계 시작 - 재귀호출로 반복실행
function StartClock() {
  PrintTime();
  timerId = setTimeout(StartClock, 1000);
}

//시계 중지
function StopClock() {
  if(timerId != null) {
    clearTimeout(timerId);
  }
}
```

시작을 누르면 StartClock() 함수가 다시 호출되고, 내부에서 PrintTime() 을 호출하며 시간을 한번 찍는다. 후 setTimeout 을 사용해 1초 후 StartClock()이 다시 시작되도록 지정함. 이때 setTimeout의 반환값도 중지를 위해 저장함. '중지' 버튼을 누르면 clearTimeout(timerId); 를 사용해 작업을 중지한다.

<br/>

## 3.setTimeout을 setInterval과 유사하게 사용하기

setInterval은 일정한 시간 간격으로 작업을 실행시키는 것이고, setTimeout은 작업 사이의 간격을 일정하게 하는 것이 차이점이다. setTimeout에서 작업 시작과 스케줄의 순서를 바꿈으로 setInterval과 유사하게 동작하도록 할 수 있다.

```js
//시계 시작 - 재귀호출로 반복실행
function StartClock() {
  timerId = setTimeout(StartClock, 1000);
  PrintTime();
}
```

앞의 예에서 단지 출력과 스케줄의 순서만 바뀌었다.

timerId = setTimeout(StartClock, 1000); 에서 1초 후 StartClock() 을 호출 하도록 지정한다. 이 라인은 지정만 하고 바로 반환한다. 이제 PrintTime()을 호출하여 출력한다. 1초가 지나면 다시 1초 후 StartClock()을 실행하도록 지정하고, PrintTime() 을 호출한다. 이게 반복되는 것임.

이제 앞에서와는 다르게 1초 기다리는동안에 PrintTime()에 호출하게 되므로 setInterval과 유사하게 동작. 이렇게 사용할 때는 역시 작업이 스케줄 시간안에 끝나는 것이 중요하다.