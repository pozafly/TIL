# 재사용

> [출처](https://blog.outsider.ne.kr/1591)

Github Actions에는 워크 플로우를 [재사용할 수 있는 워크플로우](https://docs.github.com/en/enterprise-cloud@latest/actions/using-workflows/reusing-workflows)가 있다.

워크 플로우 동작을 여러 저장소에서 쓰기 원할 때 가장 쉽게 떠올릴 수 있는 방법은 [직접 만드는 방법](https://docs.github.com/en/actions/creating-actions/about-custom-actions)임. Docker 컨테이너나 JavaScript 액션을 사용할 수 있어 원하는 동작을 마음대로 만들 수 있다.

액션을 직접 만드는게 가장 좋지만, 비슷한 액션을 여러 저장소에서 쓰게 되는 경우가 많이 생김. 아주 간단하게는 도커 로그인/빌드/푸시하는 워크 플로우라거나 배포하는 과정 등은 모든 프로젝트에서 반복적으로 사용하게 되는 경우가 많다.

starter 워크플로우를 사용하면 각 프로젝트에서 워크플로우를 처음 사용할 때 기본 템플릿으로 사용할 수 있다. 생성 시의 템플릿이므로 이후부터는 각 프로젝트에서 관리해야 하고 뭔가 바꿔야 한다면 사용한 곳을 모두 찾아 변경해주어야 한다. 직접 액션을 만들 정도로 복잡하진 않은데 반복되는 워크플로우를 한 곳에서 관리하고 싶다면 재사용할 수 있는 워크플로우가 적절할 수 있다. 비슷한 용도로 사용할 수 있는 기능이 많아 재사용할 수 있는 워크플로우의 제약과 동작 방식을 알아야 써먹을 수 있을 것 같아 테스트 해봄.

<br/>

## 재사용할 수 있는 워크플로우 생성하기

재사용 되는 워크플로우 즉, 호출되는 워크플로우를 `Called` 워크플로우라고 부른다.

```yaml
name: Reusable Workflow

on:
  push:
  workflow_call:
    inputs:
      username:
        required: true
        type: string
    secrets:
      SECRET_SEED:
        required: true

jobs:
  reusable_workflow_job:
    runs-on: ubuntu-latest
    steps:
      - run: | 
          echo ${{ inputs.username }}
          echo ${{ secrets.SECRET_SEED }}
          echo ${{ secrets.SECRET_SEED }} | sed 's/./& /g' 
```

워크플로우를 다른 곳에서도 호출할 수 있게 하려면(워크플로우로 재사용할 수 있게 만드려면) [workflow_call](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#workflow_call)을 지정해야 한다. `workflow_call`이 없으면 다른 워크플로우에서 호출할 수가 없다.

재사용할 수 있는 워크플로우를 공통으로 관리할 한 저장소에 모아두는 것도 가능하지만 한 저장소에서 이미 사용하고 있는 워크플로우를 다른 곳에서 호출해서 쓰는 것도 가능하다. 워크플로우가 호출되는 이벤트를 지정하는 `on:`에 `push`는 그대로 둔 채 `workflow_call`을 추가로 지정한다.

`workflow_call` 에서는 호출할 때 값을 전달할 수 있도록 `inputs`, `secrets` [링크](https://docs.github.com/en/actions/using-workflows/reusing-workflows#using-inputs-and-secrets-in-a-reusable-workflow)를 지정할 수 있다. 이는 그 아래에서 사용한 것처럼 `inputs.username`과 `secrets.SECRET_SEED`를 지정할 수 있다. 마지막 `echo ${{ secrets.SECRET_SEED }} | sed 's/./& /g' ` 부분은 워크플로우 실행 로그에서 시크릿은 `***` 로 출력되기 때문에 이후 의도대로 값이 달라지는지 테스트하기 윟해 시크릿 값이 출력되도록 GitHub의 검사를 우회한 것이다. (당연히 여기서는 테스트라서 그렇고 실제로 시크릿을 출력하는 것은 아주 위험하다.)

![image](https://github.com/pozafly/TIL/assets/59427983/a5aab580-b45a-483e-a101-3bc02700f944)

이 저장소의 시크릿에 `SECRET_SEED`를 저장해두고 워크플로우를 푸시해서 실행해보자. `push` 이벤트가 있으므로 코드 푸시만 해도 이 저장소가 실행되고 이는 `workflow_call`로 실행된 것이 아니라 그냥 실행된 것이다.

![image](https://github.com/pozafly/TIL/assets/59427983/ca8a758b-c078-4cd9-9030-d5110f82b58c)

`inputs`는 당연히 전달하지 않으므로 아무것도 출력되지 않고 시크릿으로 저장했던 `hello`가 출력되었다. 그냥 출력했을 때는 `***`로 출력된 것이고 마스킹 처리를 우회하기 위해 각 글자 사이에 공백이 들어가 `h e l l o`가 출력되었다.

<br/>

## 재사용할 수 있는 워크플로우 호출하기

같은 저장소에서 이 워크플로우를 사용해보자. 호출하는 쪽을 Caller 워크플로우라고 부른다.

```yaml
name: Caller Workflow

on:
  push:

jobs:
  call-workflow-1-in-local-repo:
    uses: outsideris/actions-test/.github/workflows/called.yaml@36f0d0aef48da832eb03a4d1d82e1705392bbfdb
    with:
      username: outsider
    secrets:
      SECRET_SEED: world
  call-workflow-2-in-local-repo:
    uses: ./.github/workflows/called.yaml
    with:
      username: mona
    secrets:
      SECRET_SEED: seed
```

Caller에서는 `uses` 키워드를 이용해 지정하는데 위처럼 `jobs.<job_id>.uses` 수준에서만 사용할 수 있음. 일반적인 워크플로우처럼 잡 안에 있는 `steps`에서는 사용할 수 없다. 재사용한 워크플로우의 결과를 사용해야할 때는 `outputs`를 이용해 주고 받아야 한다.

`uses` 키워드에서는 다음 두 가지 문법으로 워프클로우를 지정한다.

- `{owner}/{repo}/{path}/{filename}@{ref}` : public/internal 저장소의 재사용할 수 있는 워크플로우
- `./{path}/{filename}` : 같은 저장소의 재사용할 수 있는 워크플로우

위 예시에서는 데모용이라 두 가지 방법을 다 사용함. 위에서 `{ref}` 는 Git 커밋 SHA, 브랜치명, 태그명이 될 수 있는데 SHA는 전체 SHA를 지정해야만 한다. private 저장소의 경우에는 같은 저장소 내에서만 재사용할 수 있는 워크플로우를 호출할 수 있다. internal 저장소는 GitHub Enterprise를 사용할 때 org 밑에서만 public 처럼 쓸 수 있는 내부 저장소를 의미한다.

첫번째 잡 실행결과

![image](https://github.com/pozafly/TIL/assets/59427983/6ed50aba-66fb-4e38-9fc3-658ebd9a2221)

두번째 잡 실행결과

![image](https://github.com/pozafly/TIL/assets/59427983/b0f0ee1a-3c23-4275-8ff9-db59898d1f7e)

같은 저장소에 이미 `SECRET_SEED`가 시크릿으로 저장되어 있음에도 `push` 때와는 달리 전달한 입력값과 시크릿이 출력된 것을 확인할 수 있다.

이번엔 다른 저장소에서 호출을 해보자. 다른 저장소에서는 `{owner}/{repo}/{path}/{filename}@{ref}` 패턴만 써야 한다는 것을 제외하고는 위와 동일하다. 재사용할 수 있는 워크플로우에 접근만 할 수 있다면 앞에서 본 것과 다르진 않지만 그래도 해보자.

```yaml
name: Reusable Caller Workflow

on:
  push:

jobs:
  call-workflow-in-another-repo:
    uses: outsideris/actions-test/.github/workflows/called.yaml@4189beb6cbbebde39af9a0a4f3f2695d165973cf
    with:
      username: testuser
    secrets:
      SECRET_SEED: REMOTE_SEED
```

이를 실행하면 아까와 동일하게 워크플로우를 재사용할 수 있는 것을 볼 수 있다.

![image](https://github.com/pozafly/TIL/assets/59427983/24f519ae-dd2f-41b7-b8ad-84af0a697409)

# 제약사항

재사용할 수 있는 워크플로우에는 몇 가지 제약사항이 있다.

- 재사용된 워크플로우는 재사용할 수 있는 워크플로우를 호출할 수 없다.
- 앞에서 얘기한 대로 private 저장소의 재사용할 수 있는 워크플로우는 같은 저장소에서만 사용할 수 있다.
- caller 워크플로우에 `env` 컨텍스트로 정의된 환경변수는 called 워크플로우에 전달되지 않는다.
- 재사용할 수 있는 워크플로우를 호출하는 잡에서 `strategy` 프로퍼티는 지원되지 않는다.