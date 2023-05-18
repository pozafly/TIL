# Intersection Observer

> [출처1](https://heropy.blog/2019/10/27/intersection-observer/), [출처2](https://velog.io/@yejinh/Intersection-Observer%EB%A1%9C-%EB%AC%B4%ED%95%9C-%EC%8A%A4%ED%81%AC%EB%A1%A4-%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0), 

<br/>

<br/>

[Lazy Loading](https://developer.mozilla.org/en-US/docs/Web/Performance/Lazy_loading) 기법에는 여러가지 Javascript 기법을 사용할 수 있다.

1. **Loading attribute**
   - 그 근처에 사용자가 스크롤 할 때까지 `<img>`, `<iframe>` 을 로딩하지 않고 근처에 왔을 때, 로딩을 시작함. 근래 나온 기술이라 적용이 되지 않는 브라우저가 많을 수 있다.
2. **Polyfill**
   - 위의 [Loading attribute의 폴리필](https://github.com/mfranzke/loading-attribute-polyfill)이 존재 함. 지원 안된다면 먹여주면 된다. [Intersection Observer의 폴리필](https://github.com/w3c/IntersectionObserver/tree/main/polyfill)
3. **Intersection Observer API**
   - 타겟 엘리멘트와 타겟의 부모 혹은 상위 엘리멘트의 뷰포트가 교차되는 부분을 비동기적으로 관찰하는 API 다. (브라우저에게 해당 기능을 위임.)
4. **Event handlers**
   - 가장 원시적인 방법. scroll 이벤트로 해당 지점에 도달하면 읽어오게 이벤트 핸들러를 부착시켜 동작하게 하는 방법. 이 방법은 api call이 여러 번 있을 수 있으므로 throttle 혹은 debounce 를 따로 적용시켜주어야 한다.

<br/>

<br/>

## Intersection Observer 란?

[Intersection Observer](https://developer.mozilla.org/ko/docs/Web/API/Intersection_Observer_API)는 기본적으로 브라우저 뷰포트(Vieport)와 설정한 요소(Element)의 교차점을 관찰하며, 요소가 뷰포트에 포함 되는지 포함되지 않는지, 더 쉽게는 사용자 화면에 지금 보이는 요소인지 아닌지를 구별하는 기능을 제공한다.

이 기능은 비동기적으로 실행되기 때문에, `scroll` 같은 이벤트 기반의 요소 관찰에서 발생하는 [렌더링 성능](https://developers.google.com/web/fundamentals/performance/rendering/?hl=ko)이나 이벤트 연속 호출 같은 것을 문제 없이 사용할 수 있다.

> 📌 뷰포트(Viewport) 란?
>
> 뷰포트는 **현재 화면에 보여지고 있는 다각형(보통 직사각형)의 영역**이다. 웹 브라우저에서는 현재 창에서 문서를 볼 수 있는 부분(전체 화면이라면 화면 전체)을 말한다. 뷰포트 바깥의 콘텐츠는 스크롤 하기 전에는 보이지 않는다.
>
> 즉, Intersection Observer란 **화면(뷰포트) 상에 내가 지정한 타겟 엘리먼트가 보이고 있는지**를 관찰하는 API다.

<br/>

### 적용 사례

1. 페이지를 스크롤 할, 때 이미지를 Lazy-loading 할 때.
2. Infinity Scrolling을 통해 스크롤을 하며 새로운 컨텐츠를 불러올 때.
3. 광고 수익을 계산하기 위해 광고의 가시성을 참고할 때.
4. 사용자가 결과를 볼 것인지에 따라 애니메이션 동작 여부를 결정할 때.

<br/>

<br/>

## 왜 사용할까? - Scroll Event vs Intersection Observer

`Intersection Observer` 로 무한 스크롤을 구현함으로써 얻는 이득이 무엇이 있을지 알기 위해서 `window.addEventListener` 를 이용한 스크롤 이벤트 구현 방식에 대한 차이점을 알아보자.

<br/>

### 1. 호출 수 제한 방법 debounce, throttle을 사용하지 않아도 된다.

```js
window.addEventLinstener('scroll', () => console.log('scroll!'));
```

스크롤을 해보면 바로 console이 찍힌다. 위로 혹은 아래로 스크롤 할 때마다 해당 함수가 수도 없이 호출됨. 따라서 걷잡을 수 없이 호출되는 함수를 `debounce` 와 `throttle` 을 사용해 컨트롤 하게 된다.

<br/>

### 2. reflow를 하지 않는다.

스크롤 이벤트에서는 현재의 높이 값을 알기 위해 `offsetTop` 을 사용하는데 정확한 값을 가져오기 위해 매번 layout을 새로 그리게 된다.

layout을 새로 그린다는 것은, 렌더 트리를 재생성한다는 뜻인데, `reflow` 라고도 불리우는 이 과정이 반복되면 브라우저의 성능이 저하되고 화면의 버벅거림이 생길 수 밖에 없다.

<br/>

<br/>

## 어떻게 적용할 수 있나?

### 초기화

![intersection-observer-summary](https://user-images.githubusercontent.com/59427983/117634543-f7d61280-b1b9-11eb-829b-cddae2907622.jpeg)

`new IntersectionObserver()` 를 통해 생성한 인스턴스( `io` )로 관찰자(Observer)를 초기화하고 관찰할 대상(Element)을 지정함. 생성자는 2개의 인수( `callback`, `options` )를 가진다.

```js
const io = new IntersectionObserver(callback, options);  // 관찰자 초기화
io.observe(element);  // 관찰할 대상(요소) 등록
```

<br/>

<br/>

### callback

관찰할 대상(Target)이 등록되거나 가시성(Visibility, 보이는지 보이지 않는지)에 변화가 생기면 관찰자는 콜백(Callback)을 실행한다. 콜백은 2개의 인수( `entries`, `observer` ) 를 가진다.

```js
const io = new IntersectionObserver((entries, observer) => {}, options);
io.observe(element);
```

#### entries

`entries` 는 IntersectionObserverEntry 인스턴스의 **배열**이다.

IntersectionObserverEntry는 읽기 전용(Read Only)의 다음 속성들을 포함함.

- `boundingClientRect` : 관찰 대상의 사각형 정보(DOMRectReadOnly)
- `intersectionRect` : 관찰 대상의 교차한 영역 정보(DOMRectReadOnly)
- `intersectionRatio` : 관찰 대상의 교차한 영역 백분율(`intersectionRect` 영역에서 `boundingClientRect` 영역까지 비율, Number)
- `isIntersecting` : 관찰 대상의 교차 상태 (Boolean).
- `rootBounds` : 지정한 루트 요소의 사각형 정보(DOMRectReadOnly)
- `target` : 관찰 대상 요소(Element)
- `time` : 변경이 발생한 시간 정보(DOMHighResTimeStamp)

```js
const io = new IntersectionObserver((entries, observer) => {
  entries.forEach(entry => {
    console.log(entry) // entry is 'IntersectionObserverEntry'
  });
}, options);

io.observe(element);
```

<img width="779" alt="스크린샷 2021-05-11 오전 10 26 54" src="https://user-images.githubusercontent.com/59427983/117744358-7b881180-b243-11eb-9eb0-a5d3d7a484a6.png">

entry(IntersectionObserverEntry) 구조.

<br/>

##### boundingClientRect

관찰 대사의 사각형 정보(DOMRectReadOnly)를 반환한다. 이 값은, `Element.getBoundingClientRect()` 을 사용해 동일하게 얻을 수 있다. (getBoundingClientRect 호출에서 `reflow` 현상이 발생한다.)

![intersection-observer-bounding-client-rect](https://user-images.githubusercontent.com/59427983/117744656-154fbe80-b244-11eb-9077-e7aab8591f31.jpeg)

<img width="252" alt="스크린샷 2021-05-11 오전 10 33 35" src="https://user-images.githubusercontent.com/59427983/117744842-5b0c8700-b244-11eb-9c25-a4017a839555.png">

![intersection-observer-dom-rect](https://user-images.githubusercontent.com/59427983/117744702-27c9f800-b244-11eb-81e3-1a1ce8e857ba.jpeg)

DOMRectReadOnly 구조.

<br/>

##### intersectionRect

관찰 대상과 루트 요소와의 교차하는(겹치는) 영역에 대한 사각 정보

![intersection-observer-intersection-rect](https://user-images.githubusercontent.com/59427983/117744984-a32ba980-b244-11eb-87ee-cbfb9c18bdd2.jpeg)

<br/>

##### intersectionRatio

관찰 대상이 루트 요소와 얼마나 교차하는(겹치는)지의 수치를 `0.0` 과 `1.0` 사이의 숫자로 반환. 이는 `intersectionRect` 영역과 `boundingClientRect` 영역의 비율을 의미함

<br/>

##### isIntersecting

관찰 대상이 루트 요소와 교차 상태로 들어가거나( `true` ), 교차 상태에서 나가는지 ( `false` ) 여부를 나타내는 값(Boolean) 이다.

![intersection-observer-is-intersecting](https://user-images.githubusercontent.com/59427983/117745043-b0e12f00-b244-11eb-8764-8ff15fe9dc5f.jpeg)

<br/>

##### rootBounds

루트 요소에 대한 사각형 정보(DOMRectReadOnly)를 반환함. 이는 옵션 `rootMargin` 에 의해 값이 변경되며, 만약 별도의 루트 요소(옵션 `root` ) 를 선언하지 않았을 경우 `null` 을 반환함.

<br/>

##### target

관찰 대상(Element) 반환.

<br/>

##### time

문서가 작성된 시간을 기준으로 교차 상태 변경이 발생한 시간을 나타내는 DOMHighResTimeStamp 를 반환

<br/>

<br/>

### Observer

콜백이 실생되는 해당 인스턴스를 참조함.

```js
const io = new IntersectionObserver((entries, observer) => {
  console.log(observer);
}, options);

io.observe(element);
```

<img width="378" alt="스크린샷 2021-05-11 오전 10 40 24" src="https://user-images.githubusercontent.com/59427983/117745370-4f6d9000-b245-11eb-86ca-28e2eb33d077.png">

IntersectionObserver의 구조.

<br/>

<br/>

### options

#### root

타겟의 가시성을 검사하기 위해 뷰포트 대신 사용할 요소 객체(루트 요소)를 지정한다. 타겟의 조상 요소이어야 하며 지정하지 않거나 null 일 경우 브라우저의 뷰포트가 기본 사용된다. 기본값은 `null` 이다.

```js
const io = new IntersectionObserver(callback, {
  root: document.getElementById('my-viewport'),
});
```

![intersection-observer-root](https://user-images.githubusercontent.com/59427983/117745616-c9057e00-b245-11eb-8020-858d21b78c1e.jpeg)

<br/>

#### rootMargin

바깥 여백(Margin)을 이용해 Root 범위를 확장하거나 축소할 수 있음. CSS의 `margin`과 같이 4단계로 여백을 설정할 수 있으며, `px` 또는 `%`로 나타낼 수 있음. 기본값은 `0px 0px 0px 0px`이며 **단위를 꼭 입력**해야 한다.

- TOP, RIGHT, BOTTOM, LEFT / e.g. `10px 0px 30px 0px`
- TOP, (LEFT, RIGHT), BOTTOM / e.g. `10px 0px 30px`
- (TOP, BOTTOM), (LEFT, RIGHT) / e.g. `30px 0px`
- (TOP, BOTTOM, LEFT, RIGHT) / e.g. `30px`

```js
const io = new IntersectionObserver(callback, {
  rootMargin: '200px 0px';
})
```

![intersection-observer-root-margin-0](https://user-images.githubusercontent.com/59427983/117745830-3f09e500-b246-11eb-9f9a-a24be6bfd153.jpeg)

![intersection-observer-root-margin+100](https://user-images.githubusercontent.com/59427983/117745842-45985c80-b246-11eb-8978-025db5adbc35.jpeg)

![intersection-observer-root-margin-100](https://user-images.githubusercontent.com/59427983/117745848-4b8e3d80-b246-11eb-90ff-2254bbcea7b6.jpeg)

<br/>

#### threshold

옵저버가 실행되기 위해 타겟의 가시성이 얼마나 필요한지 백분율로 표시함. 기본값은 Array 타입의 `[0]`이지만 Number 타입의 단일 값으로도 작성할 수 있다.

- `0`: 타겟의 가장자리 픽셀이 Root 범위를 교차하는 순간(타겟의 가시성이 0%일 때) 옵저버가 실행된다.
- `0.3`: 타겟의 가시성 30%일 때 옵저버가 실행된다.
- `[0, 0.3, 1]`: 타겟의 가시성이 0%, 30%, 100%일 때 모두 옵저버가 실행된다.

```js
const io = new IntersectionObserver(callback, {
  threshold: 0.3 // or `threshold: [0.3]`
})
```

![intersection-observer-threshold-0](https://user-images.githubusercontent.com/59427983/117745959-79738200-b246-11eb-8cc6-30f47f79d3a1.jpeg)

![intersection-observer-threshold-0 3](https://user-images.githubusercontent.com/59427983/117745968-7e383600-b246-11eb-9ab0-15d099aa6a8c.jpg)

![intersection-observer-threshold-1](https://user-images.githubusercontent.com/59427983/117745982-87290780-b246-11eb-9dbd-c807ebdb0c5c.jpeg)

<br/>

### Methods

#### observe()

대상 요소의 관찰을 시작함.

```js
const io1 = new IntersectionObserver(callback, options)
const io2 = new IntersectionObserver(callback, options)

const div = document.querySelector('div')
const li = document.querySelector('li')
const h2 = document.querySelector('h2')

io1.observe(div) // div 요소 관찰
io2.observe(li) // lI 요소 관찰
io2.observe(h2) // h2 요소 관찰
```

<br/>

#### unobserve()

대상 요소의 관찰을 중지함. 관찰을 중지할 하나의 대상 요소를 인수로 지정해야 함. 단, IntersectionObserver 인스턴스가 관찰하고 있지 않은 대상 요소가 인수로 지정된 경우 아무런 동작도 하지 않는다.

```js
const io1 = new IntersectionObserver(callback, options)
const io2 = new IntersectionObserver(callback, options)

// ...

io1.observe(div)
io2.observe(li)
io2.observe(h2)

io1.unobserve(h2) // nothing..
io2.unobserve(h2) // H2 요소 관찰 중지
```

콜백의 두 번째 인수 `observer`가 해당 인스턴스를 참조하므로, 다음과 같이 작성할 수도 있다.

```js
const io1 = new IntersectionObserver((entries, observer) => {
  entries.forEach(entry => {
    // 가시성의 변화가 있으면 관찰 대상 전체에 대한 콜백이 실행되므로,
    // 관찰 대상의 교차 상태가 false일(보이지 않는) 경우 실행하지 않음.
    if (!entry.isIntersecting) {
      return
    }
    // 관찰 대상의 교차 상태가 true일(보이는) 경우 실행.
    // ...

    // 위 실행을 처리하고(1회) 관찰 중지
    observer.unobserve(entry.target)
  })
}, options)
```

<br/>

#### disconnect()

IntersectionObserver 인스턴스가 관찰하는 모든 요소의 관찰을 중지.

```js
const io1 = new IntersectionObserver(callback, options)
const io2 = new IntersectionObserver(callback, options)

// ...

io1.observe(div)
io2.observe(li)
io2.observe(h2)

io2.disconnect() // io2가 관찰하는 모든 요소(LI, H2) 관찰 중지
```

<br/>

#### takeRecords

IntersectionObserverEntry 객체의 배열을 반환함. (일반적인 상황에서 이 메서드를 호출할 필요가 없음).

<br/>

## 예제 코드

[예제](https://codepen.io/heropark/pen/LYYjMQp)

