# snippet 제작

vscode에서 스니펫을 만들어서 사용할 수 있음. snippet으로 사용하고 싶은 코드를 복사하자.

나는 예를들어 아래의 문구를 적겠다.

```jsx
import React from 'react';
import styled from 'styled-components';

const ${TM_FILENAME_BASE}Block = styled.div``;

function ${TM_FILENAME_BASE}() {
  return (
    <${TM_FILENAME_BASE}Block>
      ${TM_FILENAME_BASE}
    </${TM_FILENAME_BASE}Block>
  );
}

export default ${TM_FILENAME_BASE};
```

이제 확장자를 제외한 파일 이름을 재워줄텐데, `AuthTemplate` 이라는 말을 모두 `${TM_FILENAME_BASE}` 로 대체하자. 그리고 https://snippet-generator.app/ 여기 왼쪽에 적어주자. 

이제 상단에 Snippet의 설명(Description...) 과 줄임 단어 (Tab trigger...)를 입력하자. 설명 부분엔 `Styled React Functional Component` 라고 적고, 줄임 단어는 `srf` 라고 할거임.

이제 다 되었으면 우측 하단에 Copy snippet을 눌러 카피하자.

그다음 vscode로 돌아와서 화면의 가장 왼쪽 가장 상단의 Code -> 기본 설정 -> 사용자 코드 조각 메뉴를 누르자.

<img width="478" alt="스크린샷 2021-03-27 오전 11 25 30" src="https://user-images.githubusercontent.com/59427983/112707504-576cad00-8eef-11eb-8241-8b7addd6ec32.png">

어떤 언어의 Snippet을 설정할건지 물어보면 `javascriptreact` 를 선택하자. JSON 파일 안에 방금 복사한 녀석을 넣어주면 됨.

그리고 컴포넌트를 하나 만들어보자.

<img width="557" alt="스크린샷 2021-03-27 오전 11 30 42" src="https://user-images.githubusercontent.com/59427983/112707600-f691a480-8eef-11eb-8485-34ab15d00c92.png">

이 부분이 만약 Javascript React가 아니라, `Javascript` 라면 이 스니펫이 먹히지 않음. 해당 부분을 클릭한 뒤, `'.js' 에 대한 파일 연결 구성` -> Javascript React로 바꾸자.