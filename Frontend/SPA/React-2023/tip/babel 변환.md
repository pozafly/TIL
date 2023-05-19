# babel 변환

[참고](https://legacy.reactjs.org/blog/2020/09/22/introducing-the-new-jsx-transform.html)

[바벨 실시간 변환](https://babeljs.io/repl)

<img width="277" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/1521fb94-8201-40c1-9583-b2378348b0f8">

우측에 보면 options에서 버전을 고를 수 있다.

- Automatic : 버전 17이상
  - JSX : _jsx() -> 따라서 import React 구문 필요 없음.
- Classic : 버전 16이하
  - JSX : React.createElement()