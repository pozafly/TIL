# add, commit, push 취소

> 출처 : https://gmlwjd9405.github.io/2018/05/25/git-add-cancle.html

<br/>

## git add 취소하기(파일 상태를 Unstage로 변경하기)

`git add .` 명령어로 모든 파일을 Staging Area에 넣은 경우, Staging Area(git add 를 수행한 상태)에 넣은 파일을 빼고 싶을 때가 있음.

```shell
// 모든 파일이 Staged 상태로 바뀜
$ git add .

// 파일의 상태 확인
$ git status
```

이 때, `git reset HEAD [file]` 명령어를 통해 git add 를 취소할 수 있다.

- 뒤에 파일명이 없으면 add한 파일 전체를 취소함.
- a.vue 파일을 Unstaged 상태로 바꿔보면,

```sh
$ git reset HEAD a.vue
```

<br/>

## git commit 취소하기

### 1. commit 취소

완료한 commit을 취소해야할 때가 있음.

- 너무 일찍 commit한 경우
- 어떤 파일을 빼먹고 commit 한 경우
- 이 때, `git reset HEAD^` 명령어를 통해 git commit을 취소할 수 있다.

```sh
// commit 목록 확인
$ git log
```

1. reset --soft
   - commit은 취소하고 해당 파일들은 `staged` 상태로 워킹 디렉토리에 보존

```sh
$ git reset --soft HEAD^
```

2. reset --mixed
   - commit을 취소하고 해당 파일들을 `unstaged` 상태로 워킹 디렉토리에 보존

```shell
$ git reset --mixed HEAD^ // 기본 옵션
$ git reset HEAD^ // 위와 동일
$ git reset HEAD~2 // 마지막 2개의 commit을 취소
```

3. reset --hard
   - commit을 취소하고 해당 파일들은 unstaged 상태로 워킹 디렉토리에서 삭제

```shell
$ git reset --hard HEAD^
```

---

### 2. commit message 변경

commit message를 잘못 적은 경우, `git commit -amend` 명령어를 통해 git commit message를 변경할 수 있다.

```sh
$ git commit --amend
```

📌 Tip

1. git reset 명령은 아래의 옵션과 관련해 주의해서 사용해야 함.

   - reset 옵션
     - -soft : index 보존(add 한 상태, staged 상태), 워킹 디렉토리의 파일 보존. 즉, 모두 보존.
     - -mixed : index 취소(add 하기 전 상태, unstaged 상태), 워킹 디렉토리 파일 보존(기본 옵션).
     - -hard : index 취소(add 하기 전 상태, unstaged 상태), 워킹 디렉토리의 파일 삭제. 즉, 모두 취소

2. 만약 워킹 디렉토리를 원격 저장소의 마지막 commit 상태로 되돌리고 싶다면 아래 명령어 사용.

   - 단, 이 명령을 사용하면 원격 저장소에 있는 마지막 commit 이후의 워킹 디렉토리와 add 했던 파일들이 모두 사라진다!!!

   ```SH
   // 워킹 디렉토리를 원격 저장소의 마지막 commit 상태로 되돌린다.
   $ git reset --hard HEAD
   ```

<br/>

## git push 취소

이 명령을 사용하면 지신의 local 내용을 remote에 강제로 덮어쓰기를 하는 것이기 때문에 주의해야 함.

- 되돌아간 commit 이후 모든 commit 정보가 사라지기 때문에 주의해야 한다.
- 특히, 협업 프로젝트에서는 동기화 문제가 발생할 수 있으므로 팀원과 상의 후 진행하는 것이 좋다.

1. 워킹 디렉토리에서 commit을 되돌린다.

   - 가장 최근의 commit을 취소하고 워킹 디렉터리를 되돌린다.

     ```sh
     // 가장 최근의 commit을 취소(기본 옵션: --mixed)
     $ git reset HEAD^
     ```

   - 원하는 시점으로 워킹 디렉토리를 되돌린다.

     ```sh
     // Reflog(브랜치와 HEAD가 지난 몇 달 동안에 가리켰었던 커밋) 목록 확인.
     $ git reflog 또는 git log -g
     // 원하는 시점으로 워킹 디렉토리를 되돌린다.
     $ git reset HEAD@{number} 또는 $ git reset [commit id]
     ```

2. 되돌려진 상태에서 다시 commit을 한다.

```sh
$ git commit -m "....."
```

3. 원격 저장소에 강제로 push 한다.

```shell
$ git push origin [branch name] -f
또는
$ git push origin +[branch name]
```

<br/>

## untracked 파일 삭제하기

git clean 명령은 추적 중이지 않은 파일만 지우는게 기본 동작이다. 즉, .gitignore 에 명시하여 무시되는 파일은 지우지 않는다.

```shell
$ git clean -f   // 디렉토리를 제외한 파일들만 삭제
$ git clean -f -d   // 디렉토리까지 삭제
$ git clean -f -d -x  // 무시된 파일까지 삭제
```

- -d 옵션 : 디렉토리까지 지우는 것
- -x 옵션 : 무시된 파일(.DS_Store나, .gitignore에 등록한 확장자 파일들) 까지 모두 지우는 것
- -n 옵션 : 가상으로 실행해보고 어떤 파일들이 지워질지 알려줌.