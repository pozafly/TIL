# bitbucket ssh connect refuse

git 계정 2개를 로컬 컴퓨터에 등록시키는 과정에서 ssh key 관련 에러가 났다.

![스크린샷 2021-06-27 오후 5 28 35](https://user-images.githubusercontent.com/59427983/123537951-26f61280-d76d-11eb-94f2-838fa2c6d209.png)

다음과 같은 에러다.

`~/.ssh/config` 파일에서 나는 에러임.

먼저 key가 로컬 컴퓨터에 잘 등록되었는지 명령어로 알아보자.

```shell
$ ssh-add -l
```

![스크린샷 2021-06-27 오후 5 31 09](https://user-images.githubusercontent.com/59427983/123538016-894f1300-d76d-11eb-8c60-9b7cf2a4e698.png)

나는 다음과 같이 2개 ssh key가 잘 등록되어 있다. 그렇다면 이제 ~/.ssh/config 파일을 보자.

```
# Bitbucket Test key
Host bitbucket
    AddKeysToAgent yes
    IdentityFile ~/.ssh/bitbucket
```

이렇게만 되어 있는데,

```
# Bitbucket Test key
Host bitbucket
    HostName altssh.bitbucket.org
    User git
    Port 443
    AddKeysToAgent yes
    IdentityFile ~/.ssh/bitbucket
```

이렇게 추가해주도록 하자.

```shell
$ ssh -T bitbucket
```

이걸로 테스트 해보면 된다.

