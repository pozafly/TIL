# application/x-www-urlencoded

> [출처1](https://wildeveloperetrain.tistory.com/304), [출처2](https://jw910911.tistory.com/117)

## application/x-www-urlencoded

html form을 통한 **POST** 전송 방식 중 가장 기본이 되는 content-type이다. 데이터를 url 인코딩 후 웹 서버에 보내는 방식이다.

모든 브라우저에서는 application/x-www-urlencoded 콘텐트 타입에 대해 body의 데이터를 자동으로 encoding 하도록 구현되어 있다. 인코딩이 필요하기 때문에 크기가 큰 데이터에 대해 해당 방식으로 보내기가 적합하지 않다는 특징이 있음.

### 규칙

1. key=value 형식

요청 데이터를 key와 value의 쌍으로 구성한다. 각 쌍은 `=` 으로 키와 값으로 연결되고, 여러 개의 key-value 쌍을 `&` 로 구분된다. 이런 형식을 통해 여러 개의 데이터를 한 번에 전송할 수 있다.

1. URL Encoding

데이터의 특수 문자나 공백과 같은 부분이 url 인코딩 규칙 `[RFC1738]` 에 따라 인코딩 된다. 특정 문자들은 `%` 기호와 그 문자의 ASCII 코드를 표시하여 인코딩 된다. (공백은 `+`)

예)?nick=호선우 -> nick=%ED%98%B8%EC%84%A0%EC%9A%B0

<br/>

## multipart/form-data

위의 방식은 인코딩 방식이 들어가기 때문에 대량의 데이터를 보내는 것에 적합하지 않음. 그래서 이미지 같은 바이너리 데이터를 보낼 때는 `multipart/form-data` 라는 콘텐트 타입으로 보내게 된다.

Multipart/form-data 특징은 웹 클라이언트가 요청을 보낼 때 폼 데이터를 <u>여러 부분으로 나눠 보낼 수 있다는 것</u>이다.

예를 들어 이미지 파일과 이미지 파일에 대한 메타데이터(설명 등)를 폼 데이터로 서버에 요청을 보낸다고 했을 때, 이미지 파일에 대한 content-type은 `image/jpeg` 가 될 것이고, 설명에 대한 content-type은 `application/x-www-form-urlencoded` 가 될 것이다.

이렇게 서로 다른 두 개의 content-type을 가진 데이터를 하나의 HTTP Request Body에 보내야 한다. 따라서 폼 데이터를 여러 부분으로 나눠서 보낼 수 있는 `multipart/form-data` 가 등장한 것.

<br/>

## 비교

### application/x-www-urlencoded

![[assets/images/503b239a7deba41db10bb29cccbc2ff8_MD5.png]]

### multipart/form-data

![[assets/images/35aca6fb6ce2e1aaa7bf484a6f9a608e_MD5.png]]

### application/json

![[assets/images/fca14ad509d6f137a74f2c40af528301_MD5.png]]

결론적으로 단순 텍스트를 POST 메서드로 전송할 때는 `application/x-www-form-urlencoded` 를 선택하는게 더 효율적이다.
