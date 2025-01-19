# URL 객체

## 생성자

URL 생성자는 매개 변수로 제공한 URL을 나타내는 새로운 URL 객체를 반환한다. URL을 분석, 생성, 정규화, 인코딩 할 때 사용하며, URL의 각 구성요소를 쉽게 읽고 쓸 수 있는 속성을 제공한다. `URL` 객체 생성은 생성자에 전체 URL 문자열, 또는 상대 URL과 기준 URL을 생성자에 전달해 진행한다. 이렇게 생성한 URL 객체를 사용해 URL을 쉽게 바꾸거나 읽을 수 있다.

```js
const url = new URL(url [, base]);
```

- `url`: 절대 또는 상대 URL을 나타내는 USVString. url이 상대 URL인 경우 `base` 매개변수를 기준 URL로 사용하므로 `base`도 필수로 지정해야 한다. 절대 URL인 경우 base는 무시한다.
- `base`: `url` 매개변수가 상대 URL 인 경우 사용할 기준 URL을 나타내는 USVString. 기본값은, `''`

## 속성

- hash
  - `'#'` 과 URL 프래그먼트 식별자를 담은 string임.
- host
  - URL의 도메인(호스트 이름), `':'`, 포트를 담은 string.
- hostname
  - URL의 도메인을 담은 string.
- href
  - 전체 URL을 반환하는 문자열화 속성.
- origin (read-only)
  - URL의 출처, 즉 스킴, 도메인, 포트를 담은 string
- password
  - 도메인 이름 이전에 지정된 비번을 담은 string
- pathname
  - `'/'`와 URL의 경로를 담은 string
- port
  - URL의 포트 번호를 담은 string
- protocol
  - URL의 프로토콜 스킴을 담은 string. 마지막 `':'` 를 포함한다.
- search
  - URL의 매개변수 문자을을 나타내는 string. 어떤 매개변수라도 존재하는 경우 `'?'` 문자로 시작해, 모든 매개변수를 포함한다.
- searchParams (read-only)
  - `search` 속성의 매개변수 각각에 접근할 수 있는 URLSearchParams 임.
- username
  - 도메인 이름 이전에 지정된 사용자 이름을 담은 string

---

만약

## 사용법

### 에러 발생

만약에 `new URL()` 의 생성자로 잘못된 값(URL 형식에 맞지 않는)이 들어온다고 하자.

```js
const url = new URL('abc') // Uncaught TypeError: Failed to construct 'URL': Invalid URL
```

그렇다면 바로 TypeError가 발생한다. 따라서, 잘못된 값이 들어오지 않도록 개발자에게 즉시 경고할 수 있고, 더 안전하게 코딩할 수 있도록 도와준다.

### URL만 뽑기

아래처럼, 맨 끝에 `'/'` 가 있는지 없는지에 따라 `indexOf` 같은 것으로 판별 후 자르지 않아도 된다.

```js
const a = 'https://naver.com';
const b = 'https://naver.com/';
```

이렇게 url이 있다고 하자. 그러면, 이 url의 length를 구해야 한다고 하자.

정확한 URL을 얻고 싶다면,

```js
const a = 'https://naver.com';
const b = 'https://naver.com/';
const aa = new URL(a);
const bb = new URL(b);

console.log(aa); // URL {origin: 'https://naver.com', protocol: 'https:', ...}
console.log(bb); // URL {origin: 'https://naver.com', protocol: 'https:', ...}
```

origin을 가져오면, `b`의 마지막 슬래시가 없어진 것을 확인할 수 있다. 다른 이들과 동시에 개발할 때 더욱 안전하게 개발이 가능하다.

※ 저렇게 마지막에 슬래시를 붙이는 것을 `트레일링 슬래시(trailing slash)` 라고 함. -> 트레일링 슬래시는 디렉토리임을 나타내고, 트레일링 슬래시가 없다면 파일을 나타낸다. 즉, URL이 원래 나타내는 것은 리소스이기 때문임.
