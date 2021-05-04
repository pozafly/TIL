# config

## .gitconfig

git을 설치하면 모든 설정 파일은 `.gitconfig` 라는 파일에 저장된다.

```shell
$ git config --list
```

위 명령어로 열어볼 수 있음. 파일로 보고 싶다면,

```shell
$ git config --global -e
```

vi편집기로 수정하고 싶지 않고 vscode랑 연동해서 사용하고 싶다면, 먼저 vscode 상에서 터미널 연동이 필요함. 즉, 터미널에서 `code .` 명령어를 때리면 vscode가 켜지도록 해줘야 함. vscode에서 `ctrl + shift + p` 로 팔레트를 열고 code를 치면 `셸 명령: path에 code 명령 설치` 를 눌러주면 된다. 

```shell
$ git config --global core.editor "code --wait"
```

이 명령어 후 다시 `git config --global -e` 명령어를 주면 vscode 상으로 config 파일을 편집할 수 있게 된다.

<br/>

## 사용자 설정

```shell
$ git config --global user.name "[유저이름]"
$ git config --global user.email "[유저이메일]"
```

<br/>

## 운영체제 줄바꿈 설정

```shell
$ git config --global core.autocrlf input  // 맥 사용자
$ git config --global core.autocrlf true  // 윈도우 사용자
```

줄바꿈 문자열 때문에 문제가 생길 수 있는데 반드시 설정해주자.

<br/>

## 단축어 설정

자주 사용하는 명령어를 단축어를 사용해 설정할 수 있음.

예를 들어 `git status` 명령어를 단축어 설정해보자.

```shell
$ git config --global alias.st status
```

이렇게 해주면, `git status` 대신, `git st` 로 바꿔 사용할 수 있다.

<br/>

## git config 도움말

명령어 확인.

```shell
$ git config --h
```

<br/>

## 초기화 & 삭제

## 초기화

폴더를 만든 후 안에 들어가서

```shell
$ git init
```

하면 `.git` 이라는 숨김 폴더가 생김. 이 폴더는 깃에 관련된 모든 정보들이 안에 들어있다.

 `ls -al` 로 확인해볼 수 있고, `open .git` 으로 파인더로 열어볼 수 있다.

<br/>

## 삭제

```shell
$ rm -rf .git
```

