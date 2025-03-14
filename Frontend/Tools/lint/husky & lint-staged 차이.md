# Husky & lint-staged 차이

> [출처](https://laurieontech.com/posts/husky/)

husky로 pre-commit에서 lint 체크를 하고 있었다. 하지만, lint-staged라는 것도 있어 함께 알아보다 어떤 차이점이 있는지 궁금해졌음.

Husky와 lint-staged는 둘 다 Git 훅을 사용하여 코드 품질을 유지하고 개발자들이 커밋 전에 코드를 검토할 수 있도록 도와주는 도구. 아래는 차이점임.

1. **Husky**:
   - Husky는 Git 훅을 활용하여 특정 Git 이벤트(예: pre-commit, pre-push 등)가 발생했을 때 설정된 작업을 수행할 수 있도록 도와줌.
   - 주로 pre-commit 훅을 사용하여 커밋을 수행하기 전에 코드 검사나 테스트 등을 실행하도록 설정할 수 있음.
   - Husky를 사용하면 프로젝트 루트에 있는 `.husky` 디렉토리 내에 훅 관련 스크립트를 작성하여 설정할 수 있음.
2. **lint-staged**:
   - lint-staged는 Git에 스테이징된 파일들에 대해서만 코드 검사나 포매팅 등의 작업을 수행할 수 있도록 도와줌.
   - pre-commit 훅에서 lint-staged를 사용하면 변경된 파일들 중에서 스테이징된 파일들에 대해서만 코드 검사 및 포매팅을 실행할 수 있음.
   - lint-staged는 특정 파일 패턴에 대한 작업을 설정하고, 해당 파일이 스테이징되었을 때만 해당 작업이 실행됨.

주로 다음과 같은 흐름으로 사용됩니다:

1. Husky를 설정하여 pre-commit 훅을 트리거하면,
2. lint-staged가 스테이징된 파일들을 가져와서 설정된 작업을 수행함.

이렇게 함으로써 코드 품질을 일관되게 유지하고, 개발자가 커밋하기 전에 코드 검사와 포매팅을 수행할 수 있게 됨.

즉, husky와 lint-staged를 동시에 사용하는 이유는 husky는 pre-commit을 촉발 시키는 역할 만 하며, 명령어로 `eslint` 와 같은 것을 실행시킬 수 있지만, 모든 파일을 검사한다. 하지만, lint-staged는 stage에 올라온 파일만 검사하므로 훨씬 효율적으로 lint 검사를 할 수 있는 것임.

<br/>

## LINT-STAGED

Lint-staged는 package.json 파일에 구성된다.

```json
{
  "lint-staged": {
    "*.js": "eslint --fix"
  }
}
```

- 글로브 패턴을 사용하여 어떤 파일 지정.
- 실행할 명령을 린트 스테이지에 지정.

대부분의 경우 두 개 이상의 명령이 필요할 수도 있음. 아래와 같이.

```json
{
  "lint-staged": {
    "*.js": ["eslint", "prettier --write"]
  }
}
```

eslint, prettier 모두 하도록 함.

린트 스테이지는, '**스테이징된**' 파일에서 작동하도록 특별히 설계되었음. 변경했는데, **아직 프로젝트에 커밋하지 않은 파일을 의미함.**

스테이징 된 파일에서 작업하면 한 번에 린트해야 하는 파일 수가 제한되고 워크플로우가 더 빨라짐. 그 녀석들만 린트를 돌릴 것이기 때문임. 명령은 "pre-commit"으로 실행된다.

## HUSKY

HUSKY는 git hook을 사용해 여러 상황(ex pre-commit)

package.json 파일에 설정할 수 있다.

```json
"husky": {
  "hooks": {
    "pre-commit": "lint-staged"
  }
},
```

하지만 최신 버전인 v6에서는 이 접근 방식이 변경됨. 이제 허스키는 해당 워크플로 단계와 일치하는 파일 이름(예: "pre-commit")을 가진 별도의 bash 파일을 사용한다. 다행히도 사용자가 직접 설정할 필요가 없으며, 허스키가 이를 대신 수행할 수 있는 CLI 명령이 있음.

```sh
$ npx husky-init && npm install
$ npx husky add .husky/pre-commit "npm test"
```

명령의 첫 번째 줄은 모든 동료가 파일을 커밋하기 전에 자신의 컴퓨터에 허스키가 설치되어 있는지 확인하는 일회성 초기화 스크립트임.

두 번째 줄은.husky 디렉터리 내에 사전 커밋 파일을 생성. 초기화 명령이 실행되기 전에 husky.sh 스크립트가 실행되고 있음을 알 수 있음. 이 스크립트는 기술적으로 제거할 수 있지만 그대로 두는 것이 좋다. 이 스크립트에서는 검사를 우회하는 --no-verify 플래그를 사용하는 등 몇 가지 작업을 수행할 수 있다.

디렉토리와 관련 파일을 초기화했으면 원하는 명령을 추가할 수 있습니다. 이 경우는 npm test를 npx lint-staged로 대체함.

### PRE-PUSH

커밋 전 워크플로우는 어느 정도 행복한 길이다. 하지만 프로젝트에서 커밋 시 보푸라기가 생기고 싶지 않고 개발자가 브랜치에 변경 사항을 푸시하려고 할 때 보푸라기가 생기는 것을 원하지 않는다면 어떻게 해야 할까요?

`.husky/pre-push` 파일을 만들고 린트 스테이지를 실행하고 싶은 유혹이 있지만, 이 방법은 작동하지 않습니다. 사전 푸시 허스키 워크플로우는 올바르지만 이 시점에서 린트 스테이징을 실행하면 일치하는 파일이 0개가 됩니다. 커밋된 파일은 더 이상 스테이징되지 않기 때문에 잠시 당황스럽긴 했지만 이는 당연한 결과입니다. 대신 몇 가지 옵션이 있습니다.

1. 모든 파일에 대해 ESLint 실행: `eslint '*.js'`
2. `main`에 대한 Diff: `eslint --no-error-on-unmatched-pattern $(git diff main… --name-only --- '*.js')`

이것은 diff 명령의 한 예이며 프로젝트에 따라 고려해야 할 사항이 많다는 점에 유의하세요.
