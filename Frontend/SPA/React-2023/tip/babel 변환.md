# Babel 변환

[참고](https://legacy.reactjs.org/blog/2020/09/22/introducing-the-new-jsx-transform.html)

[바벨 실시간 변환](https://babeljs.io/repl)

![[assets/images/3a9b41a4993f1b6f1a1ea8cf286c062a_MD5.png]]

우측에 보면 options에서 버전을 고를 수 있다.

- Automatic: 버전 17이상
  - JSX: _jsx() -> 따라서 import React 구문 필요 없음.
- Classic: 버전 16이하
  - JSX: React.createElement()
