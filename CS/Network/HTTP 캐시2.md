# HTTP 캐시2

## 검증 헤더와 조건부 요청

아래 검증 헤더는 모두 '**브라우저 캐시**'에서 사용되는 것들이다.

### 검증 헤더

- 캐시 데이터와 서버 데이터가 같은지 검증하는 데이터
- Last-Modified, ETag

### 조건부 요청 헤더

- 검증 헤더로 조건에 따른 분기
- If-Modified-Since: Last-Modified 사용
- If-None-Match: ETag 사용
- 조건이 만족하면 200 OK
- 조건이 만족하지 않으면 304 Not Modified

---

- max-age는 서버에서 보내주는 값임.
- 그리고 max-age가 만료되었다고 하면, 바로 서버에 요청하는 게 아님

---

- 바로 Last-Modified 헤더(날짜)를 서버에서 보내주는데,
  - 이는 max-age가 만료 되었다면 참고하는것이다.
- 클라이언트에서 재요청했을 때 서버로 부터 온, Last-Modified 날짜 값을 If-Modified-Since 라는 헤더에 넣어서 서버로 보낸다.
- 서버에서 만약 수정되지 않았다면 304 status code로 응답한다.
  - 이때 304는 본문을 포함하지 않고 헤더만 보내주기 때문에 용량이 작다.
- 그러면, 클라이언트에서 304를 보고 아까 캐시되어있던 녀석을 그대로 쓴다.
- 이것이 검증 헤더와 조건부 요청인 것이다.

| 서버          | 클라이언트        |
| ------------- | ----------------- |
| max-age       |                   |
| Last-Modified | If-Modified-Since |

### Last-Modified, If-Modified-Since 단점

- 1초 미만(0.x초) 단위로 캐시 조정이 불가능
- 날짜 기반의 로직 사용
- 데이터를 수정해서 날짜가 다르지만, 같은 데이터를 수정해서 데이터 결과가 똑같은 경우
- 서버에서 별도의 캐시 로직을 관리하고 싶은 경우
  - 예) 스페이스나 주석처럼 크게 영향이 없는 변경에서 캐시를 유지하고 싶은 경우

따라서, ETag, If-None-Match 헤더가 나옴

### ETag, If-None-Match

- ETag(Entity Tag)
- 캐시용 데이터에 임의의 고유한 버전 이름을 달아둠
  - 예) ETag: "v1.0", ETag: "a2jiodwjekjl3"
- 데이터가 변경되면 이 이름을 바꾸어서 변경함(Hash를 다시 생성)
  - 예) ETag: "aaaaa" -> ETag: "bbbbb"
- 단순하게 ETag만 보내서 같으면 유지, 다르면 다시 받기.

따라서, **캐시 제어 로직을 서버에서 완전히 관리**한다.

| 서버          | 클라이언트        |
| ------------- | ----------------- |
| max-age       |                   |
| Last-Modified | If-Modified-Since |
| ETag          | If-None-Match     |

<br/>

<br/>

## 캐시 제어 헤더

- Cache-Control: 캐시 제어
- Pragma: 캐시 제어(하위 호환)
- Expires: 캐시 유효 기간(하위 호환)

## Cache-Control

캐시 지시어(directives)다.

- Cache-Control: max-age: 캐시 유효 기간, 초 단위
- Cache-Control: no-cache
  - 데이터는 캐시해도 되지만, 항상 원(origin) 서버에 검증하고 사용.
  - 이 말은, ETag나 Last-Modified 헤더를 IF-None-Match, If-Modified-Since헤더 가지고 값을 넣어 서버에 검증해라 라는 뜻이다.
- Cache-Control: no-store
  - 데이터에 민감한 정보가 있으므로 저장하면 안됨.(메모리에서 사용하고 최대한 빨리 삭제)

<br/>

<br/>

## 프록시 캐시

- Cache-Control: public
  - 응답이 public 캐시에 저장되어도 됨
- Cache-Control: private
  - 응답이 해당 사용자만들 위한 것임, private 캐시에 저장해야 함(기본 값)
- Cache-Control: s-maxage
  - 프록시 캐시에만 적용되는 max-age
- Age: 60(HTTP 헤더)
  - 오리진 서버에서 응답 후 프록시 캐시 내에 머문 시간(초)

<br/>

<br/>

## 캐시 무효화

이 페이지는 절대로 캐시 되면 안된다 하면 아래 헤더를 모두 다 넣어주어야 한다. (ex 사용자의 통장 잔고. 이 페이지는 계속 갱신이 될 것이기 때문)

- Cache-Control: no-cache, no-store, must-revalidate
- Pragma: no-cache
  - HTTP 1.0 하위 호환

must-revalidate는,

- 캐시 만료 후 최초 조회시 **원 서버에 검증** 해야함
- 원 서버 접근 실패시 반드시 오류가 발생해야 함 - 504(Gateway Timeout)
- must-revalidate는 캐시 유효 시간이라면 캐시를 사용함.
