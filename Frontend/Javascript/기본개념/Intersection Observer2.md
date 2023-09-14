# Intersection Observer2

> [출처](https://velog.io/@elrion018/%EC%8B%A4%EB%AC%B4%EC%97%90%EC%84%9C-%EB%8A%90%EB%82%80-%EC%A0%90%EC%9D%84-%EA%B3%81%EB%93%A4%EC%9D%B8-Intersection-Observer-API-%EC%A0%95%EB%A6%AC)

## Intersection Observer API란?

2016년 4월에 구글 개발자 페이지에서 소개 됨. 새롭게 API가 만들어진 이유는 기존의 `scroll` 이벤트와 가시성 관찰에 사용되는 `getBoundingClientRect`의 문제점 때문이다.

`scroll` 이벤트는 성능에 악역향을 줄 수 있는데 스크롤 시 짧은 시간 내에 수 백, 수 천의 이벤트가 **동기적**으로 실행될 수 있다. 그리고 페이지 내 각 요소가 각기 목적(광고, 레이지 로딩, 무한 스크롤 등)의 이유로 `scroll` 이벤트를 리스닝하기 때문에 이에 상응하는 콜백이 무수히 실행될 수 있다. 이는 **메인 스레드에 큰 부하**를 줄 수 있다.

그리고 `getBoundingClientRect`는 `reflow`를 발생시킬 수 있다. 본래 브라우저는 최적화를 위해 필요한 여러 작업을 묶어 큐에 쌓아 대기하다가 한 번의 `reflow`로 처리하고자 한다. 그러나 `getBoundingClientRect`는 호출 시 값(top, right 등)을 정확히 읽어들이기 위해 큐를 `flush` 하고 스타일을 적용함으로써 다수의 `reflow`를 발생시킬 수 있다.

사실, 스크롤 관련 루틴이 실무에서 매우 많이 쓰이고 있으므로 신뢰도 있는 공식 API의 필요성이 있다.

`Intersection Observer API` 는 루트 요소와 타겟 요소의 교차점을 관찰한다. 그리고 타겟 요소가 루트 요소와 교차하는지 아닌지를 구별하는 기능을 제공하고 있다. `scroll` 이벤트와 다르게 교차 시 **비동기적**으로 실행되며 가시성 구분 시 `reflow`를 발생시키지 않는다. 여러 모로 성능 상 유리하다.

<br/>

## 교차성(가시성)을 계산하는 방법

intersection observer는 모든 영역을 사각형(`rectangle`)으로 취급한다. 요소가 사각형이 아니거나, 이상하고 불규칙한 모습으로 렌더링 되었다 하더라도, 요소의 모든 부분을 감싸는 가장 작은 사각형으로 가정하고 교차성(가시성)을 계산한다.

이 부분을 명심하자.

<br/>

## 사용법 및 스펙

```js
let options = {
  root: document.querySelector('#scrollArea'),
  rootMargin: '0px',
  threshold: 1.0
}

// options에 따라 인스턴스 생성
let observer = new IntersectionObserver(callback, options);

// 타겟 요소 관찰 시작
let target = document.querySelector('#listItem');
observer.observe(target);
```

`new` 키워드로 인스턴스 생성, `callback`, `options` 2개 인자를 받음. `callback`은 가시성 변화가 생겼을 때 호출되는 콜백임. `options`는 인스턴스에서 콜백이 호출되는 상황을 정의.

<br/>

## Options

### root

타겟 요소의 가시성을 확인할 때 사용되는 루트 요소. 이것은 타겟 요소보다 상위 요소, 즉 요소의 조상 요소여야 한다. 설정하지 않거나 `root` 값을 `null`로 주었을 때 기본 값으로 브라우저 뷰포트가 설정된다.

### rootMargin

`margin`을 주어 **루트 요소의 범위를 확장할 수 있다**. 즉 확장된 영역 안에 타겟 요소가 들어가면 가시성에 변화가 생긴다. `CSS`의 `margin` 값과 유사하게 `top`, `right`, `bottom`, `left` 의 `margin` 정도를 각각 설정할 수 있다. 기본값은 0이며 설정시 **단위를 꼭 입력해야 한다**.

### threshold

콜백이 실행될 타겟 요소의 가시성 퍼센티지를 나타내는 단일 숫자 및 숫자 배열이 들어갈 수 있다. 즉, 요소의 `top`, `bottom`이 노출된 순간만 콜백을 실행할 수 있는 것이 아니라 어느 정도 **타겟 요소가 보여졌는지에 따라서도 콜백을 호출할 수 있다**. 예를 들어 요소가 50% 만큼 보여졌을 때 탐지하고 싶다면 단일 숫자 값 `0.5`를 설정하면 된다. 혹은 25% 단위로 가시성이 변경될 때마다 콜백이 실행되기 하고 싶다면 `[0, 0.25, 0.5, 0.75, 1]`을 설정하면 된다.

```js
// 타겟 요소가 50% 가시성이 확인되었을 때
let observer1 = new IntersectionObserver(callback, {
	threshold: 0.5
});

// 타겟 요소가 25% 단위로 가시성이 확인되었을 때
let observer1 = new IntersectionObserver(callback, {
	threshold: [0, 0.25, 0.5, 0.75, 1]
});
```

<br/>

## Callback

타겟 요소의 관찰이 시작되거나, 가시성에 변화가 감지되면 (threshold 와 만나면) 등록된 `callback`이 실행된다.

```js
let callback = (entries, observer) => {
  entries.forEach(entry => {
    // 각 entry는 가시성 변화가 감지될 때마다 발생하고 그 context를 나타냅니다.
    // target element:
    //   entry.boundingClientRect
    //   entry.intersectionRatio
    //   entry.intersectionRect
    //   entry.isIntersecting
    //   entry.rootBounds
    //   entry.target
    //   entry.time
  });
};
```

이 콜백은 메인스레드에서 처리되고 파라이터로 `entries`와 `observer`를 받게 된다.

### Entries

[IntersectionObserverEntry](https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserverEntry) 인스턴스를 담은 배열이다. 일반적으로 `callback`에 파라미터로 전달되고 후술할 `Intersection Observer.takeRecords()`를 통해 반환받을 수도 있다.

IntersectionObserverEntry는 루트 요소와 타겟 요소의 교차(threshold와 만났을 때)의 상황을 묘사한다. 포함된 프로퍼티들은 모두 **읽기 전용**(read only)ek.

- `IntersectionObserverEntry.boundingClientRect` : 타겟 요소의 사각형 정보([DOMRectReadOnly](https://developer.mozilla.org/en-US/docs/Web/API/DOMRectReadOnly))를 반환한다. `getBoudingRect()` 호출과는 다르게 `reflow`를 발생시키지 않는다.
- `IntersectionObserverEntry.intersectionRect`: 타겟 요소의 가시성이 감지된 부분의 정보([DOMRectReadOnly](https://developer.mozilla.org/en-US/docs/Web/API/DOMRectReadOnly))를 반환한다.
- `IntersectionObserverEntry.intersectionRatio` : 타겟 요소의 `intersectionRect` 이 `boundingClientRect` 와 어느정도로 교차(겹치는 지) 비율(0.0 ~ 1.0)을 반환. 바꿔 말하면 타겟 요소가 루트 요소와 얼마나 교차하는지의 정도.

타겟 요소의 관찰이 시작되면 콜백 또한 바로 호출된다. 타겟 요소와 루트 요소가 젼혀 교차하지 않았음에도 불구하고 말이다. 이는 **`Intersection Observer` 의 기본동작이다.** 이를 예외처리 하기 위해서 `intersectionRatio` 가 사용된다.

```js
let callback = (entries, observer) => {
  entries.forEach(entry => {
    // 타겟 요소가 루트 요소와 교차하는 점이 없으면 콜백을 호출했으되, 조기에 탈출
    if (entry.intersectionRatio <= 0) return;
    // 혹은 isIntersecting을 사용할 수 있다.
    if (!entry.isIntersecting) return;
    
    // 나머지 로직
  })
}
```

- `IntersectionObserverEntry.isIntersecting` : 해당 `entry` 에 타겟 요소가 루트 요소와 교차하는 지 여부를 `Boolean` 값으로 반환.
- `IntersectionObserverEntry.rootBounds` : 루트 요소의 사각형 정보([`DOMRectReadOnly`](https://developer.mozilla.org/en-US/docs/Web/API/DOMRectReadOnly))를 반환합니다. 이 정보는 `rootMargin` 옵션 설정에 영향 받음.
- `IntersectionObserverEntry.target` : 타겟 요소를 반환.
- `IntersectionObserverEntry.time` : 문서([`Document`](https://developer.mozilla.org/en-US/docs/Web/API/Document))가 만들어진 표준 시간([`time origin`](https://developer.mozilla.org/en-US/docs/Web/API/DOMHighResTimeStamp#the_time_origin))을 기준으로 타겟 요소와 루트 요소의 교차가 발생한 시간([`DOMHighResTimeStamp`](https://developer.mozilla.org/en-US/docs/Web/API/DOMHighResTimeStamp))을 반환.

<br/>

## Methods

- `IntersectionObserver.observe(targetElement)` : 타겟 요소에 대한 관찰 시작.
- `IntersectionObserver.unobserve(targetElement)` : 타겟 요소에 대한 관찰을 중지. 관찰의 목적이 이루어져 굳이 계속 관찰을 할 필요가 없는 경우 사용.
- `IntersectionObserver.disconnect()` : 인스턴스의 타겟 요소들에 대한 모든 관찰을 중지.
- `IntersectionObserver.takerecords(targetElement)` : `IntersectionObserverEntry` 인스턴스들의 배열을 리턴.

<br/>

## 용례와 한계

[MDN Intersection Observer API 페이지](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API)에서는 대표적인 용례를 4개 정도 말하고 있다.

- 페이지가 스크롤 되는 도중에 발생하는 이미지나 다른 컨텐츠의 레이지 로딩
- 스크롤 시에, 더 많은 컨텐츠가 로드 및 렌더링되어 사용자가 페이지를 이동하지 않아도 되게 하는 무한스크롤을 구현
- 광고 수익을 계산하기 위한 용도로 광고의 가시성 보고
- 사용자에게 결과가 표시되는 여부에 따라 작업이나 애니메이션을 수행할 지 여부를 결정

이 중 제가 적용한 지연 로딩과 무한스크롤의 예제를 작성해보도록 하자. 그리고 실무에서 **광고의 가시성 보고를 위해 `Intersection Observer API`를 적용하면서 느낀 한계점에 대해서 얘기해보도록 하자.**

### 지연 로딩(lazy loading)

```html
<img src='https://via.placeholder.com/500x400' data-src='https://www.sisa-news.com/data/photos/20200936/art_159912317533_32480a.jpg'></img>
```

```css
body {
  text-align: center
}

img {
  width : 500px;
  height: 400px;
  margin: 40px;
  display: inline-block;
}
```

```javascript
// 옵션 객체
const options = {
  // null을 설정하거나 무엇도 설정하지 않으면 브라우저 viewport가 기준이 된다.
  root: null,
  // 타겟 요소의 20%가 루트 요소와 겹치면 콜백을 실행한다.
  threshold: 0.2
}

// Intersection Observer 인스턴스
const observer = new IntersectionObserver(function(entries,observer) {
  entries.forEach(entry => {
    // 루트 요소와 타겟 요소가 교차하면 dataset에 있는 이미지 url을    타겟요소의 src 에 넣어준다.
    if (entry.isIntersecting) {
      entry.target.src = entry.target.dataset.src
      
      // 지연 로딩이 실행되면 unobserve 해준다.
      observer.unobserve(entry.target)
    }
  })
}, options)

const imgs = document.querySelectorAll('img')
imgs.forEach((img) => {
  // img들을 observe한다.
  observer.observe(img)
})
```

요소의 `dataset`에 저장되어있는 이미지의 url을 타겟 요소와 루트 요소가 교차했을 때 (`entry.isIntersecting`이 true일 때) 타겟 요소의 `src` 속성 값을 `dataset`의 이미지 `url`으로 교체해준다.

<br/>

## 무한 스크롤(infinite scroll)

```html
<div class="list">
</div>
<p class="list-end"></p>
```

```css
.item {
  text-align: center;
  padding: 20px 0px;
  margin: 0px
}

.item:nth-child(even) {
  background-color: gray;
}
```

```javascript
const count = 10 // 한 번 새로운 item들이 추가될 때 추가되는 item의 갯수
let index = 0 // item의 index

// 옵션 객체
const options = {
  // null을 설정하거나 무엇도 설정하지 않으면 브라우저 viewport가 기준이 된다.
  root: null,
  // 타겟 요소의 10%가 루트 요소와 겹치면 콜백을 실행한다.
  threshold: 0.1
}

let observer = new IntersectionObserver(function(entries, observer) {
  entries.forEach(entry => {
    const list = document.querySelector('.list')
    
    // 타겟 요소와 루트 요소가 교차하면
    if (entry.isIntersecting) {
      for (let i = index; i < index + count; i++) {
        // item을 count 숫자 만큼 생성하고 list에 추가해주기
        let item = document.createElement('p')
        
        item.textContent = i
        item.className += 'item'
        list.appendChild(item)
      }
      
      // index에 +count해서 갱신해주기
      index += count
    }
  })
} ,options)

// list의 끝부분을 알려주는 p 타겟 요소를 관찰
observer.observe(document.querySelector('.list-end'))
```

리스트의 끝을 나타내는 타겟 요소(`list-end` 클래스를 가진 `p` 요소)를 관찰하고 루트 요소와 타겟 요소가 교차할 때마다(entry.isIntersecting이 true) 새로운 item 추가.

<br/>

## 한계

실무에선 의미 없는 광고 노출(광고의 가시성 보고)을 가려내야할 때가 있다. 예컨대, 사용자가 화면을 빠르게 스크롤링할 때의 광고 노출은 사용자가 광고를 인식할 수 없으므로 사용자에게, 서비스 제공자에게도 상업적으로 의미 없는 노출이다. 광고의 실효성, 합리적인 광고 요금 청구를 위해서는 이런 광고 노출을 가려내야 한다.

또한 한 요청에 노출 보고서를 모아서 보내는 것이 유리하다. 서버와의 네트워크 통신은 생각보다 코스트가 큰 작업이다. 예를 들어 3개의 광고나 노출되었다고 해서 3개의 요청을 보내는 것보단, 묶어 1개의 요청으로 끝낼 수 있으면 좋다.

의미없는 광고 노출을 가려내고, 노출 보고를 모아서 보내기 위해선 유저가 스크롤링을 멈췄을 때 노출을 감지하고 보고하는 것이 가장 좋다. 스크롤링이 멈췄을 때, `getBoundingClientRect`를 이용해 뷰포트 내 요소의 노출을 감지할 수 있다. 이렇게 한다면 고속 스크롤링으로 인한 노출을 방지할 수 있고, 노출 보고를 모아서 보낼 수 있다.

하지만, 루트 요소와 타겟 요소의 교차로 가시성을 판단하는 Intersection Observer는 구현이 어렵다. 일반적 스크롤링이든 고속 스크롤링이든, 교차가 이루어지기 때문에 이 둘을 구별하기 어렵다.

`IntersectionObserverEntry.time` 으로 시간차를 계산하더라도, **어느정도의 시간차를 기준으로 스크롤링 종류를 구별할 지도 정하기 어렵다.** 또한, 교차를 기준으로 콜백을 실행하기 때문에 **이미 threshold를 지나쳤으나 타겟요소가 뷰포트 내에 있다고 감지되었을 때, 타겟요소의 콜백을 실행시킬 방법이 없어 노출 보고를 모아서 보내기 어렵다.** 결국, **이벤트 기반으로 로직으로 회귀**할 수 밖에 없다.

공식적으로 스크롤을 멈췄을 때 발생하는 이벤트는 제공하지 않는다. 그래서 `scroll-stop` 확장 이벤트를 만들어 사용한다. `scroll` 이벤트가 발생하는 동안 `timer`를 계속 초기화하고, 마지막 `scroll` 이벤트가 종료된 이후 일정 시간 동안 `scroll` 이벤트가 발생하지 않는다면 `scroll-stop` 커스텀 이벤트를 `dispatch` 한다.

```js
// event stop 이벤트
let timer = null;

window.addEventListener("scroll", function() {
  // 기존에 timer가 동작하고 있었다면, clear해준다.
  if (timer !== null) {
    this.clearTimeout(timer);
  }
  
  // timer가 tick되면 scroll-stop event를 dispatch 한다.
  timer = setTimeout(function() {
    const event = new Event("scroll-stop");
    window.dispatchEvent(event);
  }, 150);
});
```

`scroll-stop` 이벤트를 활용하면, 해당 이벤트가 발생했을 시 **스크롤이 멈췄다는 것을 보장할 수 있고, 그 순간 뷰포트 내 요소의 노출을 감지하여 노출 보고를 모아보낼 수 있다.** 그리고 사실상 **`debounce` 를 적용한 것이나 마찬가지**여서 성능상의 걱정도 덜 수 있다.

```javascript
// scroll-stop 이벤트 핸들러 정의
const scrollStopHandler = function() {
  const { top, right, bottom } = el.getBoundingClientRect();

  if (
    ((top <= window.innerHeight && top >= 0) ||
      (bottom <= window.innerHeight && bottom >= 0)) &&
    right <= window.innerWidth &&
    right > 0
  ) {
    // 이미 노출된 컨텐츠라면
    if (isExposed) return;

    // 컨텐츠가 노출되면 노출 통계를 보낸다.
    // 또한 모아 보고하는 모드(data.isCollect)라면 모아보낸다.
    if (data.isCollect) sendExposedStatForCollect(el);
    else sendExposedStat(el);
    isExposed = true;

    // 사용한 이벤트 핸들러는 지워주기
    el.removeEventListener("scroll-stop", scrollStopHandler);
  }
};

// 스크롤 stop 시 이벤트 감지할 핸들러 등록
window.addEventListener("scroll-stop", scrollStopHandler);
```

결국, 광고 요금 청구를 위한 통계를 적절히 쌓기 위해선 한계가 있는 `Intersection Observer`를 사용하기 보다는 **전통적인 `scroll` 이벤트 로직이 적합하다는 것**.