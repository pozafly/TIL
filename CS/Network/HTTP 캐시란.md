# HTTP 캐시

## 캐시의 생명 주기

HTTP 에서 **리소스**(resource)란, 웹 브라우저가 HTTP 요청으로 가져올 수 있는 모든 종류의 파일을 말한다. 대표적으로 HTML, CSS, JS, 이미지, 비디오 파일 등이 리소스에 해당함.

웹 브라우저가 서버에서 지금까지 요청한 적이 없는 리소스를 가져오려 할 때, 서버와 브라우저는 완전한 HTTP 요청/응답을 주고 받는다. HTTP 요청도 완전하고, 응답도 완전하다. 이후 HTTP 응답에 포함된 **Cache-Control 헤더**에 따라 받은 리소스의 생명 주기가 결정된다.

### 캐시의 유효 기간: max-age

서버의 Cache-Control 헤더의 값으로 `max-age=<seconds>` 값을 지정하면, 이 리소스의 캐시가 유효한 시간은 `<seconds>` 초가 된다.

#### 캐시의 유효 기간이 지나기 전

한번 받아온 리소스의 유효 기간이 지나기 전이라면, 브라우저는 서버에 요청을 보내지 않고 디스크 또는 메모리에서만 캐시를 읽어와 계속 사용한다.

![스크린샷 2023-03-05 오후 6.44.20](../../images/스크린샷 2023-03-05 오후 6.44.20.png)

위와 같이 어떤 JS 파일을 요청하는 경우 `Cache-Control` 헤더 값은 `max-age=31536000` 이기 때문에, 이 리소스는 1년(31,536,000초) 동안 캐시할 수 있다.

사진은 유효한 캐시가 메모리에 남아있기 때문에 (from memory cache)라고 표기된 것을 확인할 수 있다.

"서버에 요청을 보내지 않고" 라고 하는 말에 주의해야 한다. 한번 브라우저에 캐시가 저장되면 만료될 때까지 캐시는 계속 브라우저에 남아 있게 된다. 때문에 CDN Invalidation을 포함한 서버의 어떤 작업이 있어도 브라우저의 유효한 캐시를 지우기는 어렵다.

> ※ Cache-Control max-age 값 대신 Expires 헤더로 캐시 만료 시간을 정확히 지정할 수도 있다.

#### 캐시의 유효 기간이 지난 이후: 재검증

그렇다면 캐시의 유효 기간이 지나면 캐시가 완전히 사라지나? 그렇지 않다. 대신 브라우저는 서버에 [조건부 요청(Conditional request)](https://developer.mozilla.org/ko/docs/Web/HTTP/Conditional_requests)을 통해 캐시가 유효한지 재검증(Revalidation)을 수행한다.

![smart-web-service-cache-2](smart-web-service-cache-2.png)

재검증 결과 브라우저가 가지고 있는 캐시가 유효하다면, 서버는 **[304 Not Modified]** 요청을 내려준다. 304는 캐시가 만료되어 서버에 재요청을 보내고, 서버는 변경된 것이 없다고 판단해 304를 내려주는 것이다. **[304 Not Modified]** 응답은 HTTP 본문을 포함하지 않기 때문에 매우 빠르게 내려받을 수 있다. 예를 들어, 위 사진을 보면 59.1KB 리소스의 캐시 검증을 위해 324B 만의 네트워크 송수신만을 주고 받았음을 볼 수 있다.

![smart-web-service-cache-3](smart-web-service-cache-3.png)

대표적인 재검증 요청 헤더들로는 아래와 같은 헤더가 있다.

1. If-None-Match: 캐시된 리소스의 `ETag` 값과 현재 서버 리소스의 `ETag` 값이 같은지 확인한다.
2. If-Modified-Since: 캐시된 리소스의 `Last-Modified` 값 이후에 서버 리소스가 수정되었는지 확인한다.

위 `ETag`와 `Last-Modified` 값은 기존에 받았던 리소스 응답 헤더에 있는 값을 사용한다. 재검증 결과 캐시가 유효하지 않으면, 서버는 **[200 OK]** 또는 적합한 상태 코드를 본문과 함께 내려준다. 추가로 HTTP 요청을 보낼 필요 없이 바로 최신 값을 내려받을 수 있기 때문에 매우 효율적이다.

> max-age=0 주의보 정의 대로라면 max-age=0 값이 Cache-Control 헤더로 설정되었을 때, 매번 리소스를 요청할 때마다 서버에 재검증 요청을 보내야할 것이다. 그렇지만 일부 모바일 브라우저의 경우 웹 브라우저를 껐다 켜기 전까지 리소스가 만료되지 않도록 하는 경우가 있다. 네트워크 요청을 라끼고 사용자에게 빠른 웹 경험을 제공하기 위해서라고 한다.
>
> 이 경우 웹 브라우저를 껐다 켜거나, 아래에서 소개할 no-store 값을 사용하면 된다.

### No-cache와 no-store

Cache-Control에서 가장 헷갈리는 두 가지 값이 있다면 바로 `no-cache`와 `no-store`다. 동작은 매우 다르다.

- no-cache 값은 대부분의 브라우저에서 max-age=0과 동일한 뜻을 가진다. 즉, 캐시는 저장하지만 사용하려고 할 때마다 서버에 재검증 요청을 보내야 한다.
- no-store 값은 캐시를 절대로 해서는 안 되는 리소스일 때 사용한다. 캐시를 만들어 저장조차 하지 말라는 강력한 Cache-Control 값이다. no-store를 사용하면 브라우저는 어떤 경우에도 캐시 저장소에 해당 리소스를 저장하지 않는다.

<br/>

## 캐시의 위치

![smart-web-service-cache-4](smart-web-service-cache-4.png)

CDN과 같은 중간 서버를 사용할 때, 캐시는 여러 곳에 생길 수 있다. 서버가 가지고 있는 원래 응답을 CDN이 캐시한다. CDN의 캐시된 응답은 사용자 브라우저가 다시 가져와서 캐시한다. 이처럼 HTTP 캐시는 여러 레이어에 저장될 수 있기 때문에 세심히 다루어야 한다.

### CDN Invalidation

일반적으로 캐시를 없애기 위해서 'CDN Invalidation'을 수행한다고 이야기 한다. CDN Invalidation은 위 다이어그램 가운데 위치하는 CDN에 저장되어 있는 캐시를 삭제한다는 뜻이다. 브라우저의 캐시는 다른 곳에 위치하기 때문에 CDN 캐시를 삭제한다고 해서 브라우저 캐시가 삭제된다는 것은 아니다.

경우에 따라 중간 서버나 CDN이 여러 개 있는 경우도 발생하는데, 이 경우 전체 캐시를 날리려면 중간 서버 각각에 대해서 캐시를 삭제해야 한다.

이렇게 한번 저장된 캐시는 지우기 어렵기 때문에 Cache-Control의 max-age 값은 신중히 설정해야 함.

### Cache-Control: public과 private

CDN과 같은 중간 서버가 특정 리소스를 캐시할 수 있는지 여부를 지정하기 위해 Cache-Control 헤더 값으로 public, private를 추가할 수 있다.

public은 모든 사람과 중간 서버가 캐시를 저장할 수 있음을 나타내고, private은 가장 끝의 사용자 브라우저만 캐시를 저장할 수 있음을 나타낸다. 다시 말하면 public은 브라우저, 캐시서버 모두 캐싱 해라, private은 브라우저만 캐싱해라.

|          | public | private |
| -------- | ------ | ------- |
| 브라우저 | O      | O       |
| CDN      | O      | X       |

기존과 max-age 값과 조합하려면 `Cache-Control: public, max-age=86400` 과 같이 콤마로 연결할 수 있다.

### S-maxage

중간 서버에서만 적용되는 max-age 값을 설정하기 위해 s-maxage 값을 사용할 수 있다.

예를 들어, Cache-Control 값을 `s-maxage=31536000, max-age=0` 과 같이 설정하면 CDN에서는 1년동안 캐시되지만 브라우저에서는 매번 재검증 요청을 보내도록 설정할 수 있습니다.

<br/>

## 토스에서의 Cache-Control

### HTML 파일

일반적으로 랜딩 페이지 같은 HTML 리소스는 새로 배포가 이루어질 때마다 값이 바뀔 수 있다. 때문에 브라우저는 항상 HTML 파일을 불러올 때 새로운 배포가 있는지 확인해야 한다.

![smart-web-service-cache-5](smart-web-service-cache-5.png)

이런 리소스에 대해 토스는 Cache-Control 값으로 **max-age=0, s-maxage=31536000** 을 설정했다. 이로써 브라우저는 HTML 파일을 가져올 때마다 서버에 재검증 요청을 보내고, 그 사이에 배포가 있었다면 새로운 HTML 파일을 내려받는다.

CDN은 계속해서 HTML 파일에 대한 캐시를 가지고 있도록 했다. 대신 배포가 이루어질 때마다 CDN Invalidation을 발생시켜 CDN이 서버로부터 새로운 HTML 파일들을 받아오도록 설정했다.

### JS, CSS 파일

JavaScript나 CSS 파일은 프론트엔드 웹 서비스를 빌드할 때마다 새로 생긴다. 토스는 임의의 버전 번호를 URL 앞부분에 붙여 빌드 결과물마다 고유한 URL을 가지도록 설정하고 있다.

![smart-web-service-cache-6](smart-web-service-cache-6.png)

이렇게 JS, CSS 파일을 관리했을 때, 같은 URL에 대해 내용이 바뀔 수 있는 경우는 없다. 내용이 바뀔 여지가 없으므로 리소스의 캐시가 만료될 일도 없다.

이런 리소스에 대해 토스는 Cache-Control 값으로 max-age의 최대치인 **max-age=31536000** 을 설정하고 있다. 이로써 새로 배포가 일어나지 않는 한, 브라우저는 캐시에 저장된 JavaScript 파일을 계속 사용함.

캐시 설정을 섬세히 제어함으로써 사용자는 더 빠르게 HTTP 리소스를 로드할 수 있고, 개발자는 트래픽 비용을 절감할 수 있다. Cache-Control, ETag 헤더를 리소스의 성격에 따라 잘 설정하는 것만으로 캐시를 정확하게 설정할 수 있다는 것을 살펴봤다.

> 출처: https://toss.tech/article/smart-web-service-cache
>
> 참고:
>
> - https://www.youtube.com/watch?v=UxNz_08oS4E
> - https://web.dev/i18n/ko/http-cache/
