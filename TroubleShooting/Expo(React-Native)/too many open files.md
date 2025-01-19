# Too many open files

expo를 처음 실행할 때, 아래와 같은 오류가 났다.

`Error: EMFILE: too many open files, watch
    at FSEvent.FSWatcher._handle.onchange (internal/fs/watchers.js:178:28)`

<br/>

React-Native 프로젝트가 오류가 발생하는 경우 종종 발생한다고 함.

이럴 때는 맥 터미널에서,

```shell
$ brew install watchman
```

을 통해 watchman을 업데이트 해주자.

## Watchman

watchman은 특정 디렉토리나 파일이 변경되면 지정한 동작을 실행할 수 있게 하는 프로그램임. RN은 디버그 모드에서, 소스코드가 변경될 때마다 다시 컴파일 해 화면에 표시하는데 (Hot reload) 이는 Watchman을 이용해서 진행된다고 한다.
