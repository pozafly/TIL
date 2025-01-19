# Async & defer

> 출처: https://youtu.be/tJieVCgGzhs, https://ko.javascript.info/script-async-defer, https://beomy.github.io/tech/browser/async-defer/

<br/>

## Defer

사전적 정의: 미루다. 지연하다.

스크립트를 `백그라운드` 에서 다운로드 함. 다운로드 하는 도중에도 HTML 파싱이 멈추지 않음. 그리고 스크립트 실행은 페이지 구성이 끝날 때까지 **지연**됨.

## Async

defer과 마찬가지로 `백그라운드` 에서 다운로드 함. HTML 페이지는 async 스크립트 다운이 완료되길 기다리지 않고 페이지 내 콘텐츠를 처리, 출력함. 하지만 async 스크립트 실행 중에는 HTML 파싱이 멈춤.

<br/>

## 브라우저 실행 과정

### 1. Head 안에 script 를 포함

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Document</title>
  <script src="main.js"></script>
</head>
  <body></body>
</html>
```

1. parsing HTML: 사용자가 html을 다운받으면 브라우저가 한줄씩 분석을 시작함(parsing). css와 html을 병합하여 DOM 요소로 변환하게 됨.
2. blocked: header에 script 태그를 만나면 잠시 멈추고
3. fetching js: 그 script의 주소안에 있는 javascript 파일을 다운받는다.
4. executing js: 그 javascript를 실행한다.
5. parsing HTML: javascript 작업이 끝나면 다시 파싱을 시작.
6. page is ready

- 단점: js 파일이 크기가 크면 시간이 많이 소요되는 단점이 발생.

<br/>

### 2. Body 안에 script를 포함

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Document</title>
</head>
<body>
  <div></div>
  <script src="main.js"></script>   // 끝 부분
</body>
</html>
```

1. parsing HTML: script 태그가 끝 부분에 있으므로 문서를 먼저 준비하고 script 처리 함.
2. page is ready
3. fetching js
4. executing js

- 단점: HTML을 빨리 본다는 장점은 있지만, 웹이 js에 의존적이라면(컨텐츠를 보기 위해서 js가 반드시 필요한 상황이라면) 정상적 페이지를 보기까지 어쨌든 시간이 많이 걸림.

<br/>

### 3. Script + async

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Document</title>
  <script async src="main.js"></script>  // async
</head>
<body>
  <div></div>
</body>
</html>
```

1. parsing HTML + fetching js: head를 분석하다가 async 구문을 만나면 병렬로 js 파일을 다운로드 받음.
2. blocked + executing js: 다운로드(fetching)가 끝나면 잠시 멈추고(blocked) js를 실행(executing).
3. parsing HTML
4. page is ready

- 단점

1. js가 html을 다 받기도 전에 실행되기 때문에, querySelector 같은 DOM 조작 API를 사용하게 되면 HTML이 정의되어있지 않기 때문에 오류가 날 수 있다.
2. js가 다운받아지고 block 상태로 executing 하기 때문에 어쨌든 시간이 많이 걸릴 수 있다.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Document</title>
  <script async src="a.js"></script>
  <script async src="b.js"></script>
  <script async src="c.js"></script>
</head>
<body>
  <div></div>
</body>
</html>
```

이렇게 선언했을 경우, a, b, c 동시에 다운 받기 시작하고 먼저 다운받아지는 순서대로 실행. 순서에 상관 있는 js 라면 문제가 생길 수 있음.

<br/>

### 3. Script + defer

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Document</title>
  <script defer src="main.js"></script>   // defer
</head>
<body>
  <div></div>
</body>
</html>
```

1. parsing HTML + fetching js: script 태그를 만나 다운 시작. parsing은 멈추지 않음.
2. page is ready: 페이지가 완료되면, 페이지를 보여주고
3. executing js: js 실행

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Document</title>
  <script defer src="a.js"></script>
  <script defer src="b.js"></script>
  <script defer src="c.js"></script>
</head>
<body>
  <div></div>
</body>
</html>
```

a, b, c 동시에 fetching이 되고 HTML parsing이 끝나면 a, b, c 순서대로 js가 executing 됨.

<br/>

## 무엇을 사용할 것인가

브라우저는 기본적으로 script 태그를 만나면 파싱을 중단하고 스크립트를 다운, 파싱, 실행 과정을 진행함. body 태그가 닫히기 바로 직전에 script 태그를 선언했다면, 이미 거의 모든 HTML 파싱이 종료된 상태이기 때문에 async, defer 속성을 사용하든 의미 없음.

하지만 head 태그에 선언되었다 하면, 스크립트 파일에 종속성이 없는 경우 async 속성을 사용하는게 좋음. 반대로 종속성이 있는 경우 defer를 사용하는게 좋음.

<br/>

## Defer, async 를 동시 사용하면?

```html
<script async defer src="https://maps.googleapis.com/maps/api/js?key=YOUR_API_KEY&callback=initMap" type="text/javascript"></script>
```

async 속성을 지원하는 최신 브라우저는 기본적으로 async를 사용함. 하지만 async 속성을 지원하지 않는 구형 브라우저는 defer 속성의 지원 여부에 따라 결정됨. defer를 지원하면 defer 속성에 의해 비동기 적으로 스크립트를 실행하지만 defer 조차 지원하지 않는다면 동기적으로 스크립트를 실행함.

> `정리`
>
> async defer를 외부 api를 끌어다쓸 때 document에 주로 이 문법이 적용되어 있었다.
>
> async: 비동기적으로 js끼리 연관성이 없을 때 사용하면 좋고,
>
> defer: 지연으로 js가 연관성이 있다면 이걸 쓰도록.
