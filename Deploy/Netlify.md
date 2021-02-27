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