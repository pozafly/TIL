# Github 파일 삭제

> 출처: https://gmlwjd9405.github.io/2018/05/17/git-delete-incorrect-files.html

.gitignore에 넣지 않고 원격 저장소에 push 했다고 가정하자. 이때 해당 파일을 지울 수 있는 방법.

<br/>

## Github에 잘못 올라간 파일 삭제 과정

1. 원격 저장소에서 파일 삭제하기

이미 github remote에 push 했기 때문에 로컬의 저장소에서 파일을 삭제해도 원격 저장소에서는 삭제되지 않음.

- `git rm` VS `git rm -cached`
  - git rm: 원격 저장소와 로컬 저장소에 있는 파일을 삭제한다.
  - git rm -cached: 원격 저장소에 있는 파일을 삭제한다. 로컬 저장소에 있는 파일은 유지.

```shell
// 원격 저장소와 로컬 저장소에 있는 파일을 삭제한다.
$ git rm [File Name]

// 원격 저장소에 있는 파일을 삭제한다. 로컬 저장소에 있는 파일은 삭제하지 않는다.
$ git rm --cached [File Name]
```

<br/>

1. .gitignore 설정하기

만약 .gitignore가 제대로 설정되어 있지 않다면.gitignore 설정하여 다믕에 개인이 관리해야 하는 파일들이 원격 저장소에 올라가지 않도록 해야함. 단,.gitignore는 `git add` 명령어 전에 설정되어 있어야 적용이 가능하다.

<br/>

1. 원격 저장소에 적용하기

버전 관리에서 완전히 제외하기 위해서는 반드시 commit과 push를 수행해야함.

```shell
// 버전 관리에서 완전히 제외하기 위해서는 반드시 commit 명령어를 수행해야 함.
$ git commit -m "Fixed untracked files"

// 원격 저장소(origin)에 push
$ git push origin master
```
