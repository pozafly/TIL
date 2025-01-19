# HTTPS로 실행

때에 따라 API 호출을 위해서 https로 실행해야 할 수도 있다. https 환경을 직접 구축하는 것은 어려운데, create-react-app에서 자체적으로 https로 실행하는 옵션을 제공한다.

- 맥

  ```sh
  $ HTTP=true npm start
  ```

- 윈도우

  ```SH
  $ set HTTP=true && npm start
  ```

이 명령어를 실행하면 자체 서명된 인증서(self-signed certificate)와 함께 https 사이트로 접속한다. 자체 서명된 인증서이기 때문에 안전하지 않다는 경고 문구가 뜨지만 무시하고 진행하면 됨.

<br/>

## 두 번째 방법

> [출처](https://velog.io/@yaytomato/React-%EC%9B%B9%EC%82%AC%EC%9D%B4%ED%8A%B8-https%EB%A1%9C-%EB%A1%9C%EC%BB%AC-%ED%85%8C%EC%8A%A4%ED%8C%85%ED%95%98%EA%B8%B0)

로컬에서 https://dev.example.com으로 테스팅하기

1. `/etc/hosts` 에서 현재 ip와 가짜 url을 매핑해준다.
2. `mkcert` 로 가짜 url SSL 인증서를 생성한다.
3. `package.json` 에서 `npm start` 스크립트에 SSL 인증서 위치를 설정해준다
4. `https://dev.example.com:3000` 으로 접속한다.

Chrome 버전 M88은 같은 도메인의 http와 https 사이트를 cross-site로 취급하도록 정책이 변화함. 이를 `Schemeful same-site` 정책이라고 함.

<br/>

### 1. `/etc/hosts` 에서 현재 ip와 가짜 url을 매핑해준다

터미널에서 아래의 명령어로 ip를 찾자

```shell
$ ifconfig
```

그리고 `/etc/hosts` 파일로 들어가 ip와 dev.example.com을 매핑해주자. 그러면 브라우저에 dev.example.com url을 입력해 접속하려할 때 내 ip에서 돌아가는 서버를 띄워 줌.

```shell
$ sudo vi /etc/hosts
```

password를 물어보면 컴퓨터 비번을 적어준다. (/etc/hosts 수정은 루트만 가능해 sudo를 붙여줌)

```
# 
# Host Database
#
# localhost is used to configure the loopback interface
# when the system is booting.  Do not change this entry.
##
127.0.0.1       localhost
255.255.255.255 broadcasthost
::1             localhost

// 👇 얘를 추가한다
아까 알아낸 ip 주소     dev.example.com
```

<br/>

### 2. `mkcert` 로 가짜 url SSL 인증서를 생성한다

`mkcert`는 개발환경에 SSL 인증서를 만들어주는 라이브러리다. 터미널에서 brew로 간단하게 다운받을 수 있다.

```bash
$ brew install mkcert
```

React 어플리케이션 루트 폴더로 들어가 `mkcert`를 실행하고 example.com SSL 인증서를 만들어준다.

```bash
$ mkcert -install
$ mkcert "*.example.com" 127.0.0.1 ::1
```

그럼 루트 폴더에 `_wildcard.example.com-key.pem` `_wildcard.example.com.pem` 파일들이 생성된다.

<br/>

### 3. `package.json`에서 `npm start` 스크립트에 SSL 인증서 위치를 설정해준다

`package.json` scripts의 start를 수정해준다.

```json
  "scripts": {
    ...,
    "start": "HTTPS=true SSL_CRT_FILE=_wildcard.example.com.pem SSL_KEY_FILE=_wildcard.example.com-key.pem HOST=0.0.0.0 start:react",
    ...
```

실행한 후 `https://dev.example.com:3000` 으로 접속한다.
