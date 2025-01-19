# Brewfile을 이용해서 팀 개발 환경 만들기

> https://imch.dev/posts/lets-setup-team-development-environment-using-brewfile/

## 프로비저닝 스크립트 작성

Homebrew의 bundle 기능을 이용하고, 쉘 스크립트를 사용해 만들 수 있다. bundle은, `npm` 과 같은 녀석(번들러)이고, `Brewfile` 은 package.json 파일과 같은 종속성을 관리하는 녀석임.

### Brewfile

```sh
$ brew bundle dump
```

하면 Brewfile 이 생성된다.

<img width="252" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/10c28269-15f6-4964-8327-a3acf64e024e">

cat 해서 보면 설치 명령어들이 주욱 나옴. 그리고 Brewfile이 path에 있는 상태에서 `brew bundle` 명령어를 치면,

<img width="246" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/6479d154-0db0-46a0-975d-b1284ab73734">

이렇게 설치되는 구조다.

### Tap

- homebrew내의 기본저장소(Formulae) 외의 서드 파티 저장소다.
- brew tap 명령어를 입력하면 내 맥북에 추가된 탭 목록을 확인할 수 있음.
- brew tap `<user/repo>`를 입력하면 탭을 추가할 수 있고, brew install을 통해 설치할 때 해당 저장소를 사용할 수 있음.
- 입력할 때 `<user/repo>`는 기본적으로 GitHub 저장소를 가정하고 추가되며, repo 이름은 homebrew-*로 시작하는 저장소여야 하지만 실제로 추가할 때에는 homebrew-* 접두사를 생략해도 됩니다. 실제로 homebrew/bundle라는 탭은 https://github.com/homebrew/homebrew-bundle 에서 내용을 확인할 수 있음.

### Cask

- homebrew로 설치하지 않는 외부 GUI를 가진 어플리케이션을 설치할 수 있음.

### Mas

- mas는 App Store에서 설치할 수 있는 애플리케이션을 명령어로 설치할 수 있다.
