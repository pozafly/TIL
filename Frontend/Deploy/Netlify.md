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

<img width="243" alt="1" src="https://user-images.githubusercontent.com/59427983/118975333-e0173f00-b9ae-11eb-8032-be2e57829872.png">

이제 Netlify 홈페이지로 들어가자,

<img width="1904" alt="2" src="https://user-images.githubusercontent.com/59427983/118975407-f32a0f00-b9ae-11eb-8414-bc8bfc2f2e87.png">

이런식으로 

- Build command => 빌드 명령어(npm run build)
- Publish directory => 빌드된 파일 위치(dist)

적어주고,

![3](https://user-images.githubusercontent.com/59427983/118975503-0806a280-b9af-11eb-91e3-df9b665de272.png)

Production을 눌러보면 빌드 log를 확인할 수 있다.

<img width="704" alt="4" src="https://user-images.githubusercontent.com/59427983/118975556-1654be80-b9af-11eb-82e1-d25e8bd4f288.png"> 

요런식. 만약 실패했다면, 다시 뒤로가서 

<img width="718" alt="5" src="https://user-images.githubusercontent.com/59427983/118975617-28cef800-b9af-11eb-842b-a6ed04098c8e.png">

여기 이부분의 Deploy settings에 들어가보면,

<img width="957" alt="6" src="https://user-images.githubusercontent.com/59427983/118975696-397f6e00-b9af-11eb-9866-ccf8051f040e.png">

요런식으로 빌드명령어와 폴더를 재지정 할 수 있음.

그리고 Edit settings에 다시 들어가보면

<img width="941" alt="7" src="https://user-images.githubusercontent.com/59427983/118975754-48662080-b9af-11eb-810d-a3145d103d66.png">