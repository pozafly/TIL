# DOMContentLoaded, load, beforeunload, unload

> [출처](https://ko.javascript.info/onload-ondomcontentloaded)

HTML 문서의 생명주기엔 3가지 주요 이벤트가 관여한다.

- `DOMContentLoaded`: 브라우저가 HTML을 전부 읽고 DOM 트리를 완성하는 즉시 발생한다. 이미지 파일(`<img>`)이나, 스타일 시트 등의 기타 자원은 기다리지 않는다.
- `load`: HTML로 DOM 트리를 만드는게 완성되었을 뿐 아니라 이미지, 스타일 시트 같은 외부 자원도 모두 불러오는 것이 끝났을 때 발생.
- `beforeunload` / `unload`: 사용자가 페이지를 떠날 때 발생

활용처

- `DOMContentLoaded`: DOM이 준비된 것을 확인한 후 원하는 DOM 노드를 찾아 핸들러를 등록해 인터페이스를 초기화 할 때.
- `load`: 이미지 사이즈 등을 확인할 때 등. 외부 자원이 로드된 후이기 때문에 스타일이 적용된 상태이므로 화면에 뿌려지는 요소의 실제 크기를 확인할 수 있음.
- `beforeunload`: 사용자가 사이트를 떠나려할 때, 변경되지 않은 사항들을 저장했는지 확인시켜줄 때
- `unload`: 사용자가 진짜 떠나기 전 사용자 분석 정보를 담은 통계 자료를 전송하고자 할 때

자세히 보자.

<br/>

## DOMContentLoaded

`document` 객체에서 발생한다. 이벤트를 다루려면 `addEventListner`를 `document` 객체에 걸자.

```js
document.addEventListener.('DOMContentLoaded', ready);
```

```html
<script>
	function ready() {
    alert('DOM이 준비 되었다.');
    
    // 이미지가 로드되지 않은 상태이기 때문에 사이즈는 0 x 0다.
    alert(`이미지 사이즈: ${img.offsetWidth}x${img.offsetHeight}`);
  }
  document.addEventListener('DOMContentLoaded' ready);
</script>

<img id="img" src="https://en.js.cx/clipart/train.gif?speed=1&cache=0">
```

`DOMContentLoaded` 핸들러는 문서가 로드 되었을 때 실행된다. 따라서 핸들러 아래쪽에 위치한 `<img>` 뿐 아니라 모든 요소에 접근할 수 있다.

그렇지만 이미지가 로든되는 것은 기다리지 않기 때문에 `alert` 창엔 이미지 사이즈가 0이라고 뜬다.

처음 `DOMContentLoaded` 이벤트를 접하면 복잡하지 않은 이벤트라는 생각이 든다. 'DOM 트리가 완성되면 DOMContentLoaded이벤트가 발생한다라고 생각하기 때문.' 하지만, 특이사항이 있다.

### DOMContentLoaded와 scripts

브라우저는 HTML 문서를 처리하는 도중에 `<script>` 태그를 만나면, DOM 트리 구성을 멈추고 `<script>` 를 실행한다. 스크립트 실행이 끝난 후에야 나머지 HTML 문서를 처리한다. `<script>` 에 있는 스크립트가 DOM 조작 관련 로직을 담고 있을 수 있기 때문에 이런 방지책이 만들어졌다.

따라서, `DOMContentLoaded` 이벤트 역시 `<script>` 안에 있는 스크립트가 처리되고 난 후에 발생한다.

```html
<script>
	document.addEventListener('DOMContentLoaded', () => {
    alert('DOM이 준비 됌');
  })
</script>

<script src="https://cdnjs.cloudflare.com/ajax/libs/lodash.js/4.3.0/lodash.js"></script>

<script>
  alert("라이브러리 로딩이 끝나고 인라인 스크립트가 실행되었습니다.");
</script>
```

예시를 실행하면 '라이브러리 로딩이 끝나고…' 가 먼저 보인 후 DOM이 준비 됌이 출력된다. 스크립트가 모두 실행되고 난 후에야 `DOMContentLoaded`가 실행된다.

> DOMContentLoaded를 막지 않는 스크립트
>
> 위와 같은 규칙에 두 가지 예외 상황이 있다.
>
> 1. `async` 속성이 있는 스크립트는 `DOMContentLoaded`를 막지 않는다.
> 2. `document.createElement('script')`로 동적 생성되고 웹 페이지에 추가된 스크립트는 `DOMContentLoaded`를 막지 않는다.

### DOMContentLoaded와 styles

 외부스타일 시트는 DOM에 영향을 주지 않기 때문에 `DOMContentLoaded`는 외부 스타일시트가 로드되기를 기다리지 않는다.

하지만 예외가 있다. 스타일 시트를 불러오는 태그 바로 다음에 스크립트가 위치하면 이 스크립트는 스타일시트가 로드되기 전까지 실행되지 않는다.

```html
<link type="text/css" rel="stylesheet" href="style.css">
<script>
  // 이 스크립트는 위 스타일시트가 로드될 때까지 실행되지 않습니다.
  alert(getComputedStyle(document.body).marginTop);
</script>
```

이런 예외는 스크립트에서 스타일에 영향을 받는 요소의 프로퍼티를 사용할 가능성이 있기 때문에 만들어졌다. 위 예시에선 스크립트에서 요소의 좌표 정보를 사용하고 있다. 스타일이 로드되고, 적용되고 난 다음에야 좌표 정보가 확정되기 때문에 자연스레 이런 제약이 생겼다.

`DOMContentLoaded` 는 스크립트가 로드되길 기다린다. 위의 경우라면 당연히 스타일시트 역시 기다리게 된다.

### 브라우저 내장 자동완성

Firefox와 Chrome, Opera의 폼 자동완성(form autofill)은 DOMContentLoaded에서 일어난다. 물론 사용자가 자동완성을 허용했을 때 그렇다.

<br/>

## window.onload

스타일, 이미지 등의 리소스들이 모두 로드되었을 때 실행된다. `load` 이벤트는 `onload` 프로퍼티를 통해서도 사용할 수 있다.

`window.onload`는 이미지가 모두 로드되고 난 후 실행되기 때문에 이미지 사이즈가 제대로 출력되는 것을 확인할 수 있다.

```html
<script>
  window.addEventListener('load', (event) => {
    alert('페이지 전체가 로드되었습니다.');

    // 이번엔 이미지가 제대로 불러와 진 후에 얼럿창이 실행됩니다.
    alert(`이미지 사이즈: ${img.offsetWidth}x${img.offsetHeight}`);
  };
</script>

<img id="img" src="https://en.js.cx/clipart/train.gif?speed=1&cache=0">
```

<br/>

## window.onunload

사용자가 페이지를 떠날 때, 즉 문서를 완전히 닫을 때 실행된다. 팝업창을 닫는 것과 같은 딜레이가 없는 작업을 수행할 수 있다.

그런데 분석 정보를 보내는 것은 예외 사항에 속한다. 사용자가 웹사이트에서 어떤 행동을 하는지에 대한 분석 정보를 모드고 있다하자. `unload` 이벤트는 사용자가 페이지를 떠날 때 발생하므로 자연스럽게 `unload` 이벤트에서 분석 정보를 서버로 보내는 게 어떨까 하는 생각이 든다.

메서드 `navigator.sendBeacon(url, data)`는 이런 용도를 위해 만들어졌다. `sendBeacon`은 데이터를 백그라운드에서 전송한다. 다른 페이지로 전환 시 분석 정보는 제대로 서버에 전송되지만, 딜레이가 없는 것은 바로 이 때문이다.

```js
let analyticsData = { /* 분석 정보가 담긴 객체 */ }

window.addEventListener('unload', () => {
  navigator.sendBeacon('/analytics, JSON.stringify(analyticsData)');
});
```

- 요청은 POST로 전송된다.
- 문자열 뿐아니라 폼이나 fetch에서 설명하는 기타 포맷들도 보낼 수 있다. 대개는 문자열 형태의 객체가 전송된다.
- 전송 데이터는 64kb를 넘을 수 없다.

`sendBeacon` 요청이 종료된 시점엔 브라우저가 다른 페이지로 전환을 마친 상태일 확률이 높다. 따라서 서버 응답을 받을 수 있는 방법이 없음. 사용자 분석 정보에 관한 응답은 대개 빈 상태.

[fetch](https://ko.javascript.info/fetch) 메서드는 '페이지를 떠난 후'에도 요청이 가능하도록 해주는 플래그 `keepalive`를 지원한다.

<br/>

## window.onbeforeunload

사용자가 현재 페이지를 떠나 다른 페이지로 이동하려 할 때나 창을 닫으려고 할 때 `beforeunload` 핸들러에서 추가 확인을 요청할 수 있다.

<br/>

## readyState

문서가 완전히 로드된 후 `DOMContentLoaded` 핸들러 설정하면 어떻게 되나?

절대 실행되지 않는다. 그런데 가끔 문서가 로드되었는지 아닌지를 판단할 수 없는 경우가 있다. DOM이 완전히 구성된 후 특정 함수를 실행해야 할 때는 DOM 트리 완성여부를 알 수 없어 난감하다. 이 때는 `document.readyState`로 판단할 수 있다.

- `loading`: 문서를 불러오는 중
- `interactive`: 문서가 완전히 불러와졌을 때
- `complete`: 문서를 비롯한 이미지등의 리소스들도 모두 불러와졌을 때

이외에도 상태가 변경되었을 때 실행되는 이벤트 `readystatechange`를 사용하면 상태에 맞게 원하는 작업을 할 수 있다.

```js
// 현재 상태
console.log(document.readyState);

// 상태 변경 출력
document.addEventListener('readystatechange', () => console.log(document.readyState));
```

요즘 잘 사용하지는 않는다.

---

```html
<script>
  log('초기 readyState:' + document.readyState);

  document.addEventListener('readystatechange', () => log('readyState:' + document.readyState));
  document.addEventListener('DOMContentLoaded', () => log('DOMContentLoaded'));

  window.onload = () => log('window onload');
</script>

<iframe src="iframe.html" onload="log('iframe onload')"></iframe>

<img src="http://en.js.cx/clipart/train.gif" id="img">
<script>
  img.onload = () => log('img onload');
</script>
```

1. [1] initial readyState:loading
2. [2] readyState:interactive
3. [2] DOMContentLoaded
4. [3] iframe onload
5. [4] img onload
6. [4] readyState:complete
7. [4] window onload
