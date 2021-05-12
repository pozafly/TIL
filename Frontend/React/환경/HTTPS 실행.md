# HTTPS 실행

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