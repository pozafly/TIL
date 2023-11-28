# React UI 상태를 URL에 저장해야 하는 방법과 이유

> [출처](https://ui.toast.com/weekly-pick/ko_20211214)

많은 기능, 모달 창 또는 사이드 패널을 가진 복잡한 웹앱을 사용하면, 원하는 정보가 있는 상태가 되었는데 실수로 탭을 닫은 적이 있다.

<br/>

## 웹에서 딥링크 다시 불러오기

기존 SSR에서 사용자 검색 쿼리를 URL [(https://www.google.com/search?q=your+query+here)](https://www.google.com/search?q=your+query+here)에서 서버로 요청을 보내는 구글이 있음. 이 모델의 좋은 점은 지난주 결과를 기준으로 필터링하면 URL [(https://www.google.com/search?q=react+js&tbs=qdr:w)](https://www.google.com/search?q=react+js&tbs=qdr:w)만 공유해도 동일한 검색 쿼리를 공유할 수 있다는 것.

![image](https://github.com/pozafly/TIL/assets/59427983/b2893dcf-e2b8-4111-ab7c-d53ee21684bd)

SPA에서는 화면 표시 내용을 위해 서버에 요청할 필요 없었기 때문에 URL에 저장할 이유가 없었음. 하지만 웹의 고유한 경험인 공유 가능 링크를 잃어버렸음.

<br/>

## 단순한 딥 링킹

웹앱을 빌드할 때 최소한 `/login` 및 `/home` 과 같이 다른 페이지가 표시될 때는 URL을 변경해야 함. React 생태계에서 `React Router` 라는 클라측 라우팅에 이상적이며, Next.js는 서버측 렌더링도 지원하는 모든 기능을 갖춘 뛰어난 리액트 프레임워크임.

![1_trWHeF3QtAypmyogD7lgLQ@2x](https://github.com/pozafly/TIL/assets/59427983/bf9e292d-3767-4146-846b-4460fd9897b7)

[query-string](https://www.npmjs.com/package/query-string)과 같은 npm 패키지를 사용하고 기본 React Hook을 작성하며 URL 쿼리 파라미터를 상태에 동기화 할 수 있으며, 이에 대한 많은 튜토리얼([1](https://medium.com/swlh/using-react-hooks-to-sync-your-component-state-with-the-url-query-string-81ccdfcb174f), [2](https://www.npmjs.com/package/use-query-params), [3](https://dev.to/gaels/an-alternative-to-handle-global-state-in-react-the-url--3753))이 많이 있지만, 더 간단한 솔루션이 있다.

React 앱 [Rowy](https://rowy.io/?utm_source=medium.com&utm_medium=blog&utm_campaign=How and why you should store React UI state in the URL)의 아키텍처 재작성을 위해 React의 최신 상태 관리 라이브러리를 탐색하던 중, React 팀의 [Recoil](https://recoiljs.org/) 라이브러리를 기반으로 하는 작은 원자 기반 상태 라이브러리인 [Jotai](https://jotai.org/)를 우연히 발견했다.

이 모델의 주요 이점은 상태 원자가 구성 요소 계층 구조로부터 독립적으로 선언되고 앱의 어디에서나 조작할 수 있다는 것이다. 이는 불필요한 리렌더링을 일으키는 React 컨텍스트 문제를 해결한다. 필자는 [이전에 useRef를 활용하여 문제를 해결했다.](https://betterprogramming.pub/how-to-useref-to-fix-react-performance-issues-4d92a8120c09) `Atomic` 상태 개념에 대한 자세한 내용은 [Jotai의 문서](https://jotai.org/docs/basics/concepts)에서, 좀 더 기술적인 버전은 [Recoil의 문서](https://recoiljs.org/docs/introduction/motivation/)에서 읽을 수 있다.

<br/>

## 코드

`Jotai`에는 상태 원자를 URL 해시와 동기화하는 [atomWithHash](https://jotai.org/docs/api/utils#atom-with-hash)라는 `atom` 타입이 있다. URL에 모달의 열린 상태를 저장하는 것을 원한다고 가정한다. `atom`을 생성하여 시작하겠다. ->  📌 현재는 없어짐. 지금은 jotai의 [location](https://jotai.org/docs/extensions/location)이 되었다.

```js
import { atomWithHash } from "jotai/utils";

export const modalOpenAtom = atomWithHash("modalOpen", false);
```

그런 다음 모달 컴포넌트 자체에서 `useState` 같이 `useAtom` 을 사용해 이 `atom` 을 사용할 수 있다.

```jsx
import { useAtom } from "jotai";
import { modalOpenAtom } from "./atoms";

function ExampleModal() {
  const [open, setOpen] = useAtom(modalOpenAtom);
  return (
    <Dialog
      open={open}
      onClose={() => setOpen(false)}
      ...
    />
  )
}
```

`atomWithHash` 는 `useState` 가 사용하는 모든데이터를 저장할 수 있고, URL에 저장할 객체를 자동으로 문저열화해 더 복잡한 상태를 URL에 저장할 수 있어 공유가 가능하다는 점.

이 동작은 선택 사항이며 [replaceState](https://jotai.org/docs/api/utils#atom-with-hash) 옵션을 사용하여 비활성화할 수 있다.

```js
const modalOpenAtom = atomWithHash("modalOpen", false, {
  replaceState: true,
});
```

