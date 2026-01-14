# Bitbucket ssh connect refuse

git 계정 2개를 로컬 컴퓨터에 등록시키는 과정에서 ssh key 관련 에러가 났다.

![[assets/images/a7cf0f3974306a99f17643b629e08ddd_MD5.png]]

다음과 같은 에러다.

`~/.ssh/config` 파일에서 나는 에러임.

먼저 key가 로컬 컴퓨터에 잘 등록되었는지 명령어로 알아보자.

```shell
$ ssh-add -l
```

![[assets/images/e40dd8cb45fd90fd29e0824d627c041d_MD5.png]]

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
