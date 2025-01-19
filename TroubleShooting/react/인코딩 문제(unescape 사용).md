# 인코딩 문제(unescape 사용)

<img width="278" alt="스크린샷 2021-02-27 오후 10 16 09" src="https://user-images.githubusercontent.com/59427983/109389019-dc46c400-794d-11eb-96ec-72901820bbb5.png">

유튜브 api를 사용하다가 이런 문제를 만났다. return 되는 api에 특수문자가 포함 된 것이다.

관련 스택오버플로우: https://stackoverflow.com/questions/55385560/how-to-fix-youtube-api-results-title-that-are-returned-encoded?noredirect=1&lq=1

이걸 쓰면 된다. https://github.com/jonschlinkert/unescape/

라이브러리 구현상황은 https://github.com/jonschlinkert/unescape/blob/master/index.js 이것이다. 먼저 react에 깔아보자.

```shell
$ yarn add unescape
// npm 버전도 존재한다.
```

라이브러리가 상대적으로 작기 때문에 라이브러리를 추가 하면 된다.

```jsx
import unescape from 'unescape';
...
<p className={styles.title}>{unescape(snippet.title)}</p>   // unescape() 메서드 사용
```

이렇게 임포트 해주고, 깨지는 부분에서 unescape() 메서드를 사용해 특수문자를 html에서 표현해주면 끝.
