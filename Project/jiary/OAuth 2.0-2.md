# OAuth 2.0-2



목적 : 

독자 : 

소재 : 


## Introduction / Background

- 문제에 대한 배경 설명, 왜 문제가 중요한지


## 본문

- 원인
- 해결 방법
- 그 외 고려 사항


## 결론

- Action Item 제시

## 레퍼런스

- 



Google Docs를 사용해서 유저의 정보를 저장하고 싶었다. 물론 웹 어플리케이션에서 RDBMS나 NoSQL를 통해 유저의 정보 및 데이터를 저장하는 것이 일반적이다. 이번에 Next.js를 사용해보면서, 프론트엔드에서만 유저가 생성하는 데이터를 저장하고 싶은데, 브라우저 저장소 외 다른 곳은 없을까 생각해보다가 유저가 문서를 쉽게 작성하도록 하는 것이 목적이었기 때문에 Google Docs API를 사용해 문서를 저장하게끔 만들고 싶었다. 백엔드 없이 말이다.

내가 만든 웹 어플리케이션에서 유저의 Google Docs에 접근하기 위해서는 어떤 유저인지 알아야했고, 따라서 유저가 Google에 로그인이 되어있었어야 했다. OAuth는 다양한 플랫폼 환경에서 권한 부여를 위한 표준 프로토콜이다.

프로토콜이란, 컴퓨터 사이에서 데이터의 교환 방식을 정의하는 규칙 체계다. HTTP 프로토콜은 클라이언트와 서버 간 통신을 위한, 문서를 전송하기 위한 규약(약속)이다. 정해진 규칙에 맞게 전송하면 받는 곳에서 규약에 맞게 해석해서 원하는 데이터를 다시 규약에 맞게 보내준다.

OAuth 또한 프로토콜이다. OAuth이 만들어진 목적은 인증에 있다. 흔히 웹 어플리케이션에서 소셜 로그인 방식으로 로그인을 한다고 하면 OAuth 프로토콜을 사용해 Google, Facebook, Twitter 등의 서버에 규약에 맞게 요청하면 로그인 및 api 요청을 쉽게 할 수 있도록 한다.

> **OAuth**("**O**pen **Auth**orization")는 인터넷 사용자들이 비밀번호를 제공하지 않고 다른 웹사이트 상의 자신들의 정보에 대해 웹사이트나 애플리케이션의 접근 권한을 부여할 수 있는 공통적인 수단으로서 사용되는, 접근 위임을 위한 개방형 표준이다. [출처](https://ko.wikipedia.org/wiki/OAuth)

※ 앞으로 Google, Facebook, Twitter 등 유저 인증을 위한 OAuth 프로토콜을 제공해주는 기업을 provider라고 하겠다.

유저에게 우리가 만든 웹 어플리케이션에 쉽게 로그인 및 회원가입을 하기 위해서 provider에 로그인 하면, provider에 저장했던 유저의 정보를 가져올 수 있다. 그리고, 유저가 provider에서 지원하는 api를 사용할 수 있도록 해준다.

OAuth를 사용하면 크게 2가지를 얻을 수 있다.

1. 인증 (Authentication)
2. 인가 (Authorization)

**인증**은, 유저가 누구인지 검증하는 것을 말한다. Google 로그인 화면에서 ID와 PW를 입력해 로그인 하면, 우리가 만든 웹 어플리케이션이, 유저가 누구인지 검증하는 단계다.

**인가**는, 유저가 누구인지 검증이 되었는데 어떤 권한을 가지고 있는지 검증하는 것을 말한다. 따라서 인증 이후에 이루어지는 단계다. 로그인 후 권한증을 provider에게 건내주고, 권한이 있다면 원하는 리소스를 주는 것을 말한다.

유저가 소셜 로그인을 통해 **인증**을 받았다면 provider로부터 token을 받는다. 그리고, provider의 api를 요청할 때마다 token을 HTTP header에 포함해서 요청하게 되면, provider 서버는 token을 통해 유저를 판별하고, api를 통해 접근하는 리소스가 **인가**되었는지 판단한 뒤 우리의 웹 어플리케이션에게 리소스를 내려주게 된다.

<br />

## 용어 설명

OAuth를 이해하기 위해서는 4가지 용어를 알아야 한다.

#### 1. Resource Owner

리소스 오너는 유저다. 내가 만든 서비스를 사용하는 사람을 말한다. 용어에서 말하는 Resource란, 예를 들어 Google Docs의 리소스를 말하는데, 유저가 미리 만들어둔 문서일 수도 있고, 앞으로 우리 서비스를 통해 만든 문서일 수도 있다. 따라서 유저는 Resouce Owner다.

#### 2. Resouce Server

리소스 서버는 provider의 서버다. Google, Facebook, Twitter등 기업에 우리 웹 어플리케이션이 접근해서 Resouce를 가져와야 하는 것을 생각해보면 쉽다.

#### 3. client

클라이언트는 우리가 만든 어플리케이션이다. Resouce Owner를 대신해서 Resouce Server에 접근해 리소스를 가져오고 표현할 수 있다. 웹 어플리케이션이 될 수도 있고, 데스크톱 어플리케이션일 수 있고, 모바일 어플리케이션이 될 수 있다.

#### 4. Authorization Server

인가 서버는 Resouce Server와 마찬가지로 provider의 서버다. 단, Resouce Server는 '리소스'를 내려주는 서버이며, Authorization Server는 단순히 인증 및 인가만을 담당한다.



## oauth 등록 페이지에서 뭐 등록해야하는지

[Google Console](https://console.cloud.google.com/)에 접속하면, 먼저 OAuth 클라이언트를 생성할 수 있다. 

[oauth-client]

클라이언트를 생성하는데 필요한 정보는 아래와 같다.

- URI : 우리 서비스가 존재할 URI
- 리디렉션 URI : 구글 로그인 후 다시 우리 서비스로 돌아가야 할 때 어디로 돌아갈지 명시해주는 URI

OAuth는 보안 상 반드시 https 위에서 동작하게 되어있지만, 개발 및 테스트를 위해 예외적으로 http를 localhost에서만 허용하고 있다. 또한, 리디렉션은 구글 로그인 페이지에서 유저가 로그인 후 다시 우리 서비스로 돌아와야 하기 때문에 리디렉션 URI가 필요하다. 이 때, 리디렉션 URI에 쿼리 스트링으로 token 또는 code가 함께 넘어올 것이다.

[client-id-and-secret]

클라이언트가 생성되면 `CLIENT_ID` 와 `CLIENT_SECRET`을 준다.







## OAuth 프로세스

[payco-oauth]

사진은 payco 개발자 센터의 사진이다. 알아보았던 용어를 사진과 매칭시켜보면 아래와 같다.

- Resource Owner : 사용자
- Client : 서비스
- Authorization Server : PAYCO 인증 서비스
- Resource Server : PAYCO API 서비스

그러면, 우리가 만드는 웹 어플리케이션에서는 어떤 것이 필요한지 알아보자. 먼저, 로그인 버튼 혹은 로그인 페이지가 필요하다. 유저가 로그인을 시도하면, 우리는 provider의 가이드 대로 provider의 로그인 페이지를 보여주어야 한다. 앞으로 Google의 

이 때 필요한 것은, provider OAuth에 등록한 `CLIENT_ID`를 포함한 url이 필요하다.



## 등록한게 어떻게 사용되고 있는지





## 프론트엔드 만들기





## next.js 백엔드 연동으로 만들기

















## OAuth 1.0과 2.0의 차이











