# remote 변경

> [출처](https://webisfree.com/2020-04-14/[git]-git-remote-repository-%EB%B3%80%EA%B2%BD%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95)

Git에서 리모트 저장소(remote repository) 를 다른 주소 URL 로 변경하고자 할 때.



## 언제 리모트 저장소 변경이 필요?

새롭게 remote repository를 생성한 경우가 있다. 새로운 계정으로 repository를 생성했는데 이 계정으로 형상관리를 해야하는 경우.

- 기존 주소 : https://github.com/testA/my.git
- 새로운 주소 : https://github.com/testB/your.git

<br/>

## 변경하기

## 방법1.

### 기존 레파지토리 remote 제거

```sh
$ git remote remove origin
```

### 새 레파지토리 remote 추가

```sh
$ git remote add orgin [레파지토리 주소]
```

<br/>

## 방법2.

소스들이 라이브되어 서비스 중이라면 조심해야 한다. 모든 소스를 백업 해두거나 이전 리포지토리를 나중에 삭제하는 것도 방법임.

```shell
$ git remote set-url origin <새로운gitUrl>
```

이렇게 하면 변경 됨.

```shell
$ git config --list
```

로 현재 설정된 모든 내용을 리스트로 출력해보자. 리스트에서 remote.origin.url을 찾아 주소가 바뀌었는지 확인해보자. 혹은

```shell
$ git remote -v
```

를 사용할 수도 있음.

<br/>

## 변경 후 주의사항

일단 저장소 주소만 바뀌었고, 실제로 remote 서버와 동기화가 이루어지지 않았다. 이때 동기화 하는 방법으로 아래 명령어를 사용할 수 있다.

```sh
$ git remote update origin --prune
```

명령어는 리모트 주소와 동기화를 수행한다. 이때 여러 케이스가 존재, 에러 발생할 수 있음.

`현재 저장소의 소스와 동기화할 리모트 소스가 서로 상이한 경우`  일 수 있고,

`새로운 리모트 저장소에 해당하는 브랜치가 없는 경우` 일 수 있다.

