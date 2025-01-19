# NVM

<br/>

node version manager: node.js 버전을 관리해줌.

# 설치

https://github.com/nvm-sh/nvm

 여기서 readme.md에

```shell
$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.37.2/install.sh | bash
```

이런 명령어가 있다. vscode 터미널에서 bash 에 붙여넣어서 하면 됨.

설치가 완료되면 nvm -v 라고 하면 안뜸. 컴퓨터 환경에서 nvm을 인식하게 하기 위해서는

```
export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
 [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm 
```

이 문구가 필요함.

<img width="883" alt="1" src="https://user-images.githubusercontent.com/59427983/119072355-4a1efb00-ba26-11eb-98ab-61d885928788.png">

`~/.bashrc` 파일에 붙어넣어야 함.

```shell
$ vi ~/.bashrc
```

그리고 `nvm -v` 하면 nvm의 버전이 나오는데 나오면 성공.

일반적으로 nvm install 버전… 이렇게하면 버전이 깔리기도 하고 그 버전으로 바꿔지기도 함.
