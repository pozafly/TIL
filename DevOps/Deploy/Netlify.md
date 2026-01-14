# Netlify

전역으로 설치 하는 명령어
```shell
$ npm install netlify-cli -g
```
배포 명령어. 해당 디렉토리에서 실행해야 한다. 그리고 프로젝트를 `npm run build` 로 빌드해주자.
```shell
$ netlify deploy
```
그러면 선택할 수 있는게 2개가 나옴

- Link this directory to an existing site
  - 현재 내 디렉토리를 이미 있는 사이트와 연결시킬 건지,
- Create & configure a new site
  - 새로운 사이트를 만들건지.

새로운 웹사이트를 만들어야 하므로 `Create & configure a new site` 요거.

그리고 팀을 정해주고, site 이름을 정해주고, 배포할 폴더를 지정해야하는데 build 후이니, build라고 적어주자.
```
Website Draft URL: https://6039bc3b3794954457798b10--pozafly-habit-tracker.netlify.app
```
이렇게 url이 나오는데 여기 접속해보면 배포된 홈페이지를 볼 수 있다.

<br/>

<br/>

## 상용화 하기

### 빌드

1. Npm run build 를 하면 최 상단에 dist 폴더가 생기며,
2. Html, js, css 등 정적 파일이 생성됨.
3. 이걸 웹서버에 올리면 됨.

![[assets/images/1d730f0ca23ef37b7ed48c6299846223_MD5.png|13]]

이제 Netlify 홈페이지로 들어가자,

![[assets/images/296054bc834ca44626f7b60d5d2e9338_MD5.png|13]]

이런식으로

- Build command => 빌드 명령어(npm run build)
- Publish directory => 빌드된 파일 위치(dist)

적어주고,

![[assets/images/322ac198d29e3df762469df1b76448dc_MD5.png]]

Production을 눌러보면 빌드 log를 확인할 수 있다.

![[assets/images/a85ae92e0d51fe16fdbcc22481e4349f_MD5.png|13]]

요런식. 만약 실패했다면, 다시 뒤로가서

![[assets/images/fed1f26ecd933d5cf66f505156ffe957_MD5.png|13]]

여기 이부분의 Deploy settings에 들어가보면,

![[assets/images/a7a5ce93bc966d60bd9f2b8f681a21b3_MD5.png|13]]

요런식으로 빌드명령어와 폴더를 재지정 할 수 있음.

그리고 Edit settings에 다시 들어가보면

![[assets/images/3ef9a7879c590f2de54d66863e1efc88_MD5.png|13]]

이렇게 Base directory 를 지정할 수 있는데, 만약 연동한 github가 상위 디렉토리가 하나 더 있을경우(ex. 본 플젝은 vue-cli라고 한다면 그 상위폴더인 vue-example 이라는 폴더로 감싸져 있는경우) 여기서 내부 폴더인 vue-cli 폴더를 다시 Base directory에서 지정하여 주면 됨.

지정 후

![[assets/images/60916ae72fc6532a8078c000337cc7a9_MD5.png|13]]

Deploys탭에 다시 돌아가서 오른쪽 Trigger deploy 를 눌러 Clear cache and deploy site를 눌러서 캐시 삭제 후 자동 배포 됨.

배포 후 사이트가 생성되는데, 생성된 후에도 앞 뒤로 왔다갔다 해보면

![[assets/images/2981beaacd96ec3990db066637a5a814_MD5.png|13]]

이런식의 Page Not Found가 뜰 수 있다. 이때 SPA 호스팅시 서버에 추가해줘야 하는 설정이 있음.

https://cli.vuejs.org/guide/deployment.html#netlify

여기 공식문서 cli 에 들어가서 쫌 내려보면 netlify 란이 있다. 여기에 보면 _redirects 파일이 있어야 한다고 한다. /public 밑에 말이다.

![[assets/images/4b777249b809277512ed7148ec1122dc_MD5.png|10]]

이걸

![[assets/images/3b82a63bf5d537c605439d18f0217aeb_MD5.png|11]]

요렇게 넣어줌. 서버 설정해주지 않으면 서버에서 페이지를 찾을 수 없다고 뜨기 때문에 이걸 설정을 꼭 해줘야 함.

근데 안되넹….

### 환경 변수 파일을 이용한 옵션 설정

https://joshua1988.github.io/vue-camp/deploy/env-setup.html

Cli로 생성한 플젝의 설정을.env 파일에 숨겨서 배포할 수 있다.

![[assets/images/83ed98e29f90b7a99b6cc3311636df26_MD5.png|12]]

가장 상단에.env 파일을 만들고 이렇게 변수=값 형태로 넣었다. 여기서 만약 APP_TITLE=HELLO 라고 했다면, 원하는대로 출력되지 않는다.

웹팩 설정을 더 건들여주어야 하는데, 편하게 하고 싶다면 VUE_ 라는 접두사를 붙여주고 작업하면 된다.

![[assets/images/2e7ad7c5264d582a7135a19bac6fc5fc_MD5.png|13]]

이런식으로 process.env.VUE_APP_TITLE을 하면 HELLO 라는 값이 console 에 찍히겠지? 이런식으로 가능함. 단, 서버를 내렸다가 올려야 한다. local에서도 서버 내렸다 올려야 함.
