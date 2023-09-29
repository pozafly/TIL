# Lazy Loading

> [웹 성능 최적화를 위한 Image Lazy Loading 기법](https://helloinyong.tistory.com/297)

![image](https://github.com/pozafly/TIL/assets/59427983/d93f773a-a714-447f-bc06-6ce03190058a)

이미지는 모든 웹 사이트와 어플리케이션에서 매우 중요한 요소다. 마케팅 배너 혹은 상품 이미지, 로고 등 웹 사이트 내에서 이미지가 없다는 것은 상상할 수 없음. 하지만, 이미지는 페이지 성능에 가장 많이 영향을 주고 있다.

최신 [HTTP Archive data](https://httparchive.org/reports/state-of-images)에 따라서, 데스크톱 내 페이지 중앙 사이즈는 1511KB 이고, 그중 이미지가 거의 650KB의 크기를 차지하므로 전체 페이지 크기에서 대략 45%를 차지함. 이미지를 웹 페이지에서 없앨 수는 없기 때문에, 우리는 이미지를 그대로 쓰면서 웹 페이지를 빠르게 로딩할 수 있는 방법을 찾아야 함.

<br/>

## Image Lazy Loading이란 무엇인가

Image Lazy Loading은 이미지가 실제로 화면에 보여질 필요가 있을 때 로딩할 수 있도록 하는 테크닉. 바로 로딩을 하지 않고 로딩 시점을 뒤로 미루는 것.

페이지를 로드하자마자 리소스를 로딩하는 일반적인 방식 대신, 실제로 사용자 화면에 보여질 필요가 있을 때까지 기다리는 것.

SPA 내에서 js 파일이 나중에까지 필요하지 않으면 초기에 가져오지 않는 것이 좋다. 이 처럼 이미지도 바로 보여질 필요가 없다가, 실제로 보여질 필요가 있을 때 로딩 하는 것.

<br/>

## 왜 사용해야 하나?

### 1. 성능 향상

페이지 초기 로딩 시 필요한 이미지 수를 줄일 수 있다. 리소스 요청을 줄이는 것은 다운로드 bytes를 줄이는 것이고, 제한된 네트워크 대역폭의 경쟁을 줄이는 것을 의미함. 디바이스가 다른 리소스들을 빠르게 처리해 다운로드 하도록 확보하는 것이다.

### 2. 비용 감소

통신 비용관점. 이미지 전달 또는 다른 전달할 무언가는 전송 바이트 수에 기반하여 청구된다.

<br/>

## 어떤 이미지에 Lazy Loading을 적용할 수 있는가?

페이지에서 스크롤을 아래로 내림으로써, 이미지의 placeholder는 뷰포트(웹 페이지 내 보여지는 부분)에 다가오게 된다. 뷰포트에 보여지게 되면 이미지를 로딩하도록 트리거를 일으킨다.

lazy loading에 어떤 이미지가 적합한지, 그리고 페이지 초기 로딩 시 얼마나 바이트 용량을 줄일 수 있는지 [Lighthouse](https://developers.google.com/web/tools/lighthouse/)을 이용해서 알 수 있다. 또한 [ImageKit's website analyzer](https://imagekit.io/website-analyzer/?utm_source=lazyblog&utm_medium=blog)는 lazy loading 사용 여부를 확인할 수 있을 뿐 아니라, 페이지 내에서 크리티컬한 이미지 관련해서 최적화 용도로도 사용할 수 있다.

<br/>

## Lazy Loading을 다루는 여러가지 기술

이미지는 CSS background를 이용하거나 `<img>` 태그로 가져올 수 있다.

### `<img>`

2단계로 나눌 수 있음.

#### 1. 이미지 로딩 사전 차단. 
- `<img>`는 일반적으로 `src` 이용. 1000번째 이미지이든, 뷰포트 밖에 있든 상관 없이 무조건 로드함.
- 따라서, src 속성 대신 다른 속성에다 이미지 URL을 넣는 것이다.

#### 2. data-src의 데이터를 src 속성으로 옮김

`data-src` 속성에 넣고, 뷰포트에 들어오자마자 로딩할 수 있도록 한다.

#### **자바스크립트 이벤트를 이용하여 image load를 일으키는 방법**

`scroll`, `resize` 그리고 `orientationChange` 이벤트 리스너를 이용한다. scroll 이벤트는 사용자가 스크롤 하는 시점을 확인한다. resize, orientationChange 이벤트는 lazy loading을 위해 함께 사용해야 한다. 

- resize는 윈도우 브라우저 크기 변경 시 트리거.
- orientationChange 이벤트는 디바이스 화면이 가로에서 세로 모드로(또는 반대로) 바뀔 때 발생. 이 처럼 가로, 세로 모드를 변경하는 것은 화면 내 보여지는 이미지의 수가 바뀔 수 있으므로 이미지들을 로드하는 트리거가 필요할 것.

이 이벤트 중 하나가 발생할 때 로딩이 지연되었거나 아직 로딩되지 않은 이미지들을 모두 찾는다. 윈도우 높이, 현재 스크롤 위치, 그리고 이미지 위치를 이용해 어떤 이미지가 뷰포트 안으로 들어왔는지 확인한다.

뷰포트 안으로 들어오면 `<img>`의 `data-src` 속성에 지정된 URL을 `src` 속성에 넣어 이미지 로드한다. 이후 `lazy class`를 제거한다. 모든 이미지가 로딩되면 트리거를 일으키던 이벤트 리스너를 제거한다.

```html
// 처음 뷰포트에 있는 것은 src
<img src="https://ik.imagekit.io/demo/img/image1.jpeg?tr=w-400,h-300" />
// 하단에 있는 data-src
<img class="lazy" data-src="https://ik.imagekit.io/demo/img/image4.jpeg?tr=w-400,h-300" />
```

```css
img {
  background: #F1F1FA;
  width: 400px;
  height: 300px;
  display: block;
  margin: 10px auto;
  border: 0;
}
```

```js
document.addEventListener('DOMContentLoaded', function () {
  const lazyloadImages = document.querySelectorAll('img.lazy');
  let lazyloadThrottleTimeout;

  const lazyload = () => {
    if (lazyloadThrottleTimeout) {
      clearTimeout(lazyloadThrottleTimeout);
    }

    lazyloadThrottleTimeout = setTimeout(() => {
      const scrollTop = window.pageYOffset;
      lazyloadImages.forEach((img) => {
        if (img.offsetTop < window.innerHeight + scrollTop) {
          img.src = img.dataset.src;
          img.classList.remove('lazy');
        }
      });
      if (lazyloadImages.length == 0) {
        document.removeEventListener('scroll', lazyload);
        window.removeEventListener('resize', lazyload);
        window.removeEventListener('orientationChange', lazyload);
      }
    }, 20);
  }

  document.addEventListener('scroll', lazyload);
  window.addEventListener('resize', lazyload);
  window.addEventListener('orientationChange', lazyload);
});
```

주목할 부분은 위 예시에서 처음 3개는 src로 미리 로딩 되어 있음.

#### **Intersection Observer API를 이용하여 image load를 일으키는 방법**

엘리먼트 요소가 뷰포트에 들어가는 것을 감지하고 액션을 취하게 해줌. 성능 면으로 좋다.

이미지 로드를 지연시키기 위해 모든 **이미지에 옵저버를 부착**시킨다. element가 뷰포트에 들어간 것을 API가 감지하면 `isIntersecting` 속성을 이용해 `data-src` 속성에서 `src` 속성으로 이동시켜 브라우저가 이미지를 로드하도록 트리거를 일으킨다. 전부 로드되면 `lazy 클래스` 명을 이미지에서 삭제하고 부착했던 옵저버를 제거한다.

```js
document.addEventListener('DOMContentLoaded', () => {
  let lazyloadImages;

  if ('IntersectionObserver' in window) {
    lazyloadImages = document.querySelectorAll('.lazy');
    const imageObserver = new IntersectionObserver((entries, observer) => {
      entries.forEach(function (entry) {
        if (entry.isIntersecting) {
          const image = entry.target;
          image.src = image.dataset.src;
          image.classList.remove('lazy');
          imageObserver.unobserve(image);
        }
      });
    });
    lazyloadImages.forEach((image) => imageObserver.observe(image));
  } else {
    let lazyloadThrottleTimeout;
    lazyloadImages = document.querySelectorAll('.lazy');

    const lazyload = () => {
      if (lazyloadThrottleTimeout) {
        clearTimeout(lazyloadThrottleTimeout);
      }

      lazyloadThrottleTimeout = setTimeout(() => {
        const scrollTop = window.pageYOffset;
        lazyloadImages.forEach(function (img) {
          if (img.offsetTop < window.innerHeight + scrollTop) {
            img.src = img.dataset.src;
            img.classList.remove('lazy');
          }
        });
        if (lazyloadImages.length == 0) {
          document.removeEventListener('scroll', lazyload);
          window.removeEventListener('resize', lazyload);
          window.removeEventListener('orientationChange', lazyload);
        }
      }, 20);
    };

    document.addEventListener('scroll', lazyload);
    window.addEventListener('resize', lazyload);
    window.addEventListener('orientationChange', lazyload);
  }
});
```

`이벤트 리스너`와 `Intersection Observer` 이 두 방법의 로딩 시간을 비교해보면, Intersection Observer가 이미지를 로드하는 트리거가 훨씬 빠르고, 스크롤할 때 이미지가 느리게 나타나지 않는 것을 확인할 수 있다.

이벤트 리스너는 성능을 위해 `timeout`을 추가했는데, 이 부분은 사용성 측면에서 이미지 로드에 약간의 딜레이 발생하는 미미한 영향이 있음.

#### **Native Lazy Loading 방식**

최신 브라우저에서 native lazy loading을 지원한다. 임베딩할 이미지에 'loading' 속성만 추가해주면 된다.

```html
<img src="example.jpg" loading="lazy" alt="..." />
<iframe src="example.html" loading="lazy"></iframe>
```

- lazy : 뷰포트에서 일정한 거리에 닿을 때까지 로딩 지연.
- eager(default) : 현재 페이지 위치가 위, 아래 어디에 위치하던 상관없이, 페이지가 로딩 되자마자 이미지 로딩.

📌 참고, 로딩 지연된 이미지들이 다운로드 될 때 다른 감싸고 있는 콘텐트 내용이 밀려나는 것을 방지하려면, 반드시 `height`, `width` 속성을 `<img>`에 추가하거나 inline style로 직접 값을 추가해야 한다. (source: [web.dev](https://web.dev/browser-level-image-lazy-loading/#load-in-distance-threshold))

```html
<img src="…" loading="lazy" alt="…" width="200" height="200">
<img src="…" loading="lazy" alt="…" style="height:200px; width:200px;">
```

#### **CSS 속성 중 Background Image를 Lazy Loading 하는 방법**

CSS background 이미지는 간단하지 않다. 현재 문서 내 DOM 노드에 CSS 스타일이 적용되는지 여부를 결정하기 위해 CSSOM(CSS Object Model)과 DOM(Document Object Model) Tree를 구성하는 것이 필요하다.

구현 방식은 똑같다. element가 뷰포트에 들어올 때까지 CSS background image 속성을 적용하지 ㅇ낳도록 하면 된다.

```html
<div id="container">
  <h3>Lazy loading CSS background images</h3>
  <p>
    Nam lacinia tortor quis volutpat lacinia. Aliquam in orci in nunc vehicula
    maximus. Phasellus elementum nulla augue, at aliquam sem pulvinar dapibus.
    Vivamus molestie venenatis risus pulvinar interdum. Phasellus blandit tortor
    eget nulla sagittis auctor. Cras sed leo in velit lobortis euismod.
    Suspendisse non ante tellus.
  </p>
  <div id="bg-image" class="lazy"></div>
</div>
```

```css
#bg-image.lazy {
   background-image: none;
   background-color: #F1F1FA;
}
#bg-image {
  background-image: url("https://ik.imagekit.io/demo/img/image10.jpeg?tr=w-600,h-400");
  max-width: 600px;
  height: 400px;
}
```

```javascript
// intersection observer JavaScript 코드와 동일
```

위 예시에서 주목할 점은 lazy loading 관련해서 구현한 자바스크립트 코드가 똑같다는 것.

ID bg-image를 lazy 클래스를 제거하면 된다.

<br/>

## lazy loading 기법으로 유저 인터페이스 향상시키는 방법

lazy loading은 성능이 뛰어나다. 하지만, 거의 대부분 기업에서는 초기 placeholder가 보기 좋지 않다던가, 로딩 시간이 느리다와 같은 여러 이유로 유저 사용성에 좋지 않다고 판단해 lazy loading을 쓰지 않는다.

### 1. 올바른 image placeholder를 이용한 방법

placeholder는 이미지가 로딩될 때까지 해당 이미지 자리에 대신 표시할 요소를 말한다. 보통 개발자들은 이미지의 단색 이미지를 사용하거나 모든 이미지에 대체할 특정 이미지를 넣는 방식으로 구현한다.

#### a) 주요 색상을 placeholder로 사용하기

고정된 색상을 placeholder로 사용하는 대신, 사용하려는 실제 이미지의 주요한 색상을 이용하는 방법이 있다.

아래는 [Manu.ninja](https://manu.ninja/dominant-colors-for-lazy-loading-images/)에서 소개하고 있는 예시다.

![image](https://ik.imagekit.io/demo/img/pinterest-placeholders.gif)

이미지의 첫 `1x1` 픽셀로 스케일을 감소 시키고 해당 픽셀 요소로 placeholder를 채우는 아주 간단한 방식이다.

`ImageKit`을 사용하면, 체인 변환을 이용해 주요 단일 색상을 얻을 수 있다.

```html
<!-- Original image at 400x300 -->
<img src="https://ik.imagekit.io/demo/img/image4.jpeg?tr=w-400,h-300" alt="original image" />

<!-- Dominant colour image with same dimensions -->
<img src="https://ik.imagekit.io/demo/img/image4.jpeg?tr=w-1,h-1:w-400,h-300" alt="dominant color placeholder" />
```

placeholder는 단지 661 바이트의 용량을 가진다. 원본 이미지의 12700 바이트의 용량과 비교해 **약 19배** 단축 되었다.

#### b) 저화질의 이미지로 placeholder로 사용하기(LQIP)

하나의 색상을 사용하는 대신, 원본 이미지에 대한 저화질의 흐린 이미지를 placeholder로 사용하는 것이다. 이는 이미지가 로딩되는 동안 유저가 보기에 이전보다 더 보기 좋을 뿐 아니라 원본 이미지에 대해 어느정도 추측하도록 유도할 수 있다.

이 방식은 Facebook과 Medium.com 같은 웹 사이트와 어플리케이션에서 사용.

ImageKit을 이용해 LQIP 이미지 url을 적용한 예시다.

```html 
<!-- Original image at 400x300 -->
<img src="https://ik.imagekit.io/demo/img/image4.jpeg?tr=w-400,h-300" alt="original image" />

<!-- Low quality image placeholder with same dimensions -->
<img src="https://ik.imagekit.io/demo/img/image4.jpeg?tr=w-400,h-300,bl-30,q-50" alt="dominant color placeholder" />
```

LQIP(저화질 이미지)는 1300바이트의 크기를 가지므로, 원본 이미지에서 약 10배 정도 감소된 사이즈다. 그리고 다른 placeholder 기술보다 시각적인 면에서 더 좋은 사용성을 제공함.

<br/>

### 2. 이미지 로딩을 위한 버퍼 시간을 추가하는 방법

이미지를 로딩하는 트리거에 대한 차이점을 논의할 때, 뷰포트 하단 부분과 이미지의 상단 부분이 만나는 시점이 뷰포트에 들어오는 시점인 것을 확인할 수 있었음.

#### 문제점

사용자가 빠르게 스크롤 하면 이미지가 로딩될 시간이 필요하다는 것이 placeholder로 보여지게 되는 것이다. 이것은 이미지를 로딩하는 스크롤 이벤트를 성능 이슈로 인해 쓰로틀링을 사용해 딜레이가 약간 발생하는 것.

Intersection Observer 또는 저화질의 이미지를 이용한 방식으로 placeholder의 퍼포먼스를 더 좋게 만들면서도, 더 나아가 간단한 트릭으로 이미지가 뷰 포트에 들어올 때 완벽하게 이미지가 로딩하도록 할 수 있다. 그것은 바로 이미지의 로딩할 트리거 시점에 마진 값을 넣는 것.

#### 해결 방법

이미지가 뷰포트에 완전히 들어올 때 로딩하는 대신, 뷰포트에 들어오는 부분에서 500px 정도 떨어진 곳에 들어오면 로딩을 하는 것이다.

Intersection Observer API에서는 'root'파라미터와 'rootMargin'파라미터(기본 CSS 마진 규칙에서 동작)를 함께 사용하여 'intersection'(이벤트가 발생되는 부분)을 찾는 위치를 증가시킬 수 있음.

<br/>

### 3. lazy loading으로 콘텐트 요소들이 이동하는 것을 방지하는 방법

**문제점**

이미지가 없을 때, 브라우저는 보여지고 있는 콘텐트의 공간을 알고있지 않다. 그래서 만약 CSS로 따로 지정해주지 않으면, 콘텐트를 감싸고 있던 컨테이너 부분은 공간을 따로 가지고 있지 않음으로 0x0 픽셀이 됩니다. 따라서 이미지가 로딩되면, 브라우저가 해당 이미지의 크기에 맞게 컨테이너를 다시 리사이징한다.

이러한 갑작스런 변경은 다른 엘리먼트 요소들이 이동되어 밀려나게 되는 원인으로 인해 일어나게 된다. 이 부분은 Smashing 매거진의 [content shifting article & video](https://www.smashingmagazine.com/2016/08/ways-to-reduce-content-shifting-on-page-load/) 에서 정확히 알 수 있으며, 이는 이미지가 로딩될 때 갑작스런 콘텐트 이동으로 인해 꽤나 유저에게 좋지 않은 사용성을 주게 됨. 

**해결 방법**

콘텐트를 감싸고 있는 컨테이너에 너비/높이를 지정함으로써 이러한 문제를 피할 수 있음. 이로 인해 브라우저는 너비와 높이를 가지고 이미지 영역을 알 수 있음. 이후 이미지가 로딩되면 이미 이미지 사이즈 영역이 이미지 크기에 완벽히 맞게 지정되어 있으므로, 나머지 컨테이너 영역들은 그대로 유지된다.

<br/>

### 4. 모든 이미지에 lazy load를 적용하지 않기

초기 로딩 시에는 로딩이 줄어들기는 하지만, 많은 이미지들로 인해 유저 사용성으로 좋지 않은 결과가 주어질 수도 있다. 특히나 웹 페이지 상단의 이미지는 자바스크립트가 적용되고 동작할 때까지 보여지지 않을 것.

이를 위해, 아래에는 lazy loading에 어울리는 이미지가 무엇인지 식별할 수 있는 일반적인 원칙.

**a)** 뷰포트에 있거나, 웹 페이지에서 시작되는 이미지들은 lazy loading을 하도록 하면 안된다. 페이지 상단 이미지나 마케팅 배너, 로고 등과 같은 이미지는 페이지가 로딩 되자마자 유저에게 보여져야 한다.

또한, 모바일과 데스크톱에는 화면 크기가 다르게 때문에, 초기에 화면에 보여지는 이미지 수가 다르다. 따라서 어떤 리소스들을 미리 로딩할 것인지 결정하기 위해 디바이스 타입을 고려할 필요가 있다.

**b)** 뷰포트에서 살짝 떨어져 있는 이미지는 lazy loading할 이미지로 적합하지 않다. 이것은 앞 단에서 뷰포트에 완벽히 들어서기 전에 미리 로딩을 하는 부분을 다루던 내용에 기반한다면, 뷰포트 하단으로부터 500px 이내인 이미지들은 미리 로딩되어야 하는 것을 예시로 들 수 있다.

**c)** 만약 페이지가 길지 않다면, 한, 두번 정도의 스크롤로 페이지를 내리거나 뷰포트 바깥에 이미지 개수가 5개 이하일 것입니다. 이럴 땐 lay loading을 전부 걸지 않는게 더 좋을 수 있다.

성능 면으로 최종적으로 유저에게 중요한 장점을 제공하지 않을 수도 있습니다. lazy loading를 걸기위해 추가한 JS파일이 고작 몇 안되는 이미지 수의 lazy loading을 위해서 오히려 장점이 사라질 수도 있다.

<br/>

## 자바스크립트에 대한 Lazy Loading의 의존성

이러한 lazy loading에 대한 방식들은 유저 브라우저 내에서 자바스크립트 동작에 의존되어 일어난다.

대부분 유저들이 브라우저 내 자바스크립트 코드가 요즘 거의 모든 웹 사이트에서 효과적으로 실행되는 동안, 자바스크립트가 지원되지 않는 브라우저를 사용한다거나, 자바스크립트 동작을 허용하지 않도록 한 유저들을 위해 계획을 세울 필요가 있음.

유저들에게 최신 브라우저를 이용, 또는 자바스크립트를 활성화 하라는 메시지나 혹은 왜 이미지가 로딩되지 않는지 알려주어야 할 것. 아니면 noscript 태그를 이용해서 유저들에게 좋은 사용성을 제공하도록 할 수도 있다. 그러나 `<noscript>`태그를 이용한 방식은 몇가지 문제점을 가지고 있음.

[여기](https://stackoverflow.com/questions/29145354/dry-lazy-loaded-images-with-noscript-fallback)에 이에 대한 좋은 해결방법이 있습니다. 이에 대한 해결법을 찾는 사람들에게 추천 함.
