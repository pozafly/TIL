# Stash

어떤 작업을 하던 중 다른 요청이 들어와 하던 작업을 멈추고 잠시 브랜치를 변경해야 할 일이 있다고 하자. 이 때 아직 완료하지 않은 일을 commit 하는 것은 껄끄러움.

![stash](https://user-images.githubusercontent.com/59427983/116997032-3893de80-ad17-11eb-9102-f78afddf9395.png)

즉, stash란, 아직 마무리하지 않은 작업을 스택에 잠시 저장할 수 있도록 하는 명령어다. 여기서 stack이라고 하는 이유는 스택과 같이 LIFO 구조처럼 동작하기 때문이다. 마지막에 들어간 것이 처음 나오는 것. push, pop.

<br/>

사용 예시 )

- Working Directory에서 작업중인데 브랜치를 전환하여 다른 사람의 commit을 테스트 해야할 때, (브랜치 전환이 필요할 때)
- 버그를 고치고 있는데 잘 되지 않아 다른 시도를 해야할 때. 즉, 각각의 다른 시도를 해야할 때.

<br/>

<br/>

## 데모

### stash  생성

```sh
$ git stash
or
$ git stash push
```

라고 아무 옵션 없이 입력하면 stash stack에 그냥 저장됨.

```shell
$ git stash push -m "[stash명]"
```

이렇게 이름을 부과해줄 수 있다. 이렇게 되면 내가 작업중인 Working Directory가 깔끔해진 것을 볼 수 있다.

이제, 다시 새로운 작업을 Working Directory에서 새로운 내용을 추가 후 이번엔 Staging Area에 add로 추가한다. 이때 index에 있는 Staging Area에 있는 것을 유지하면서 stash에 저장하고 싶다면

```sh
$ git stash push -m "[stash명]" --keep-index
```

이렇게 `--keep-index` 옵션을 사용하면 된다.

이 상태에서, 새로운 파일(untracked) 추적되지 않은(add 되지 않은) 파일을 새로 생성하고 `git stash` 명령어로 stash에 넣게 되면 새로운 파일은 stash에 들어가지 않고 add 로 추가된 Staging Area에 있는 파일만 stash에 들어가게 된다.

이렇게 추적되지 않은 파일까지 모두 stash에 집어넣고 싶다면,

```sh
$ git stash -u
```

-u 옵션을 붙여주면 모두 들어가게 됨.

<br/>

<br/>

### stash list

stash 한 내용을 보고 싶다면,

```sh
$ git stash list
```

<img width="354" alt="스크린샷 2021-05-04 오후 8 54 51" src="https://user-images.githubusercontent.com/59427983/116999730-fec4d700-ad1a-11eb-9ef7-120dc0f049c5.png">

이렇게 어떤 stash가 저장되어 있는지 볼 수 있다. 사진에서 `stash@{0}` 과 같은 녀석들은 이 녀석의 ID다. 그러면 stash에 저장되어 있는 것들 중 어떤 소스가 수정되었는지 알려면,

```sh
$ git stash show [stash ID]
```

이렇게 ID 값을 입력해주면 세부 내용을 볼 수 있다. 끝에 `-p` 옵션을 주면 더 자세하게 볼 수 있음.

<br/>

<br/>

### apply

이제 다시 나의 업무를 하러 Stash 내용을 내 Working Directory로 불러오고 싶다면,

```sh
$ git stash apply
```

명령어로 돌아올 수 있다. 여기서 apply만 입력하고 뒤에 내용을 입력하지 않으면 가장 최근에 stash에 저장된 녀석을 가져오게 된다. 즉, stack의 pop 과 같은 개념이다. 특정 stash를 가져오고 싶다면 `git stash list` 로 ID를 확인 후 

```sh
$ git stash apply [stash ID]
```

이렇게 ID를 입력해주면 된다.

<br/>

<br/>

### 삭제(pop & drop)

apply에서는 단순히 stash에 있는 내용을 Working Directory에 적용했을 뿐이다. stash list를 보면 list는 그대로다. 하지만,

```sh
$ git stash pop
```

명령어를 사용하면 가장 상위의 stash에 저장되었던 내용이 적용되는 동시에 사라진다. stack에서 사라지듯.

이제 그냥 하나만  삭제만 해보자.

```sh
$ git stash drop [stash ID]
```

stash에 들어있는 내용 전체를 삭제하고 싶다면,

```sh
$ git stash clear
```

<br/>

<br/>

### stash에 있는 내용을 적용한 브랜치 만들기

stash에 저장해둔 녀석을 새로운 브랜치를 만들면서 동시에 적용하고 싶다면?

```sh
$ git stash branch [새로운 브랜치명]
```

