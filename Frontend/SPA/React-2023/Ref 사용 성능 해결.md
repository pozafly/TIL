# Ref 사용 성능 해결

> [출처](https://betterprogramming.pub/how-to-useref-to-fix-react-performance-issues-4d92a8120c09)

공식 홈피는 '탈출구'로 소개되어 있다.

<br/>

## 문제

![1_1h6w52_v9rflIGJ9WlDPGw](https://github.com/pozafly/TIL/assets/59427983/2e21a035-7ccd-4a28-b8bd-3bf17e5bdb80)

기본 테이블 위로 미끄러지는 단일 행 편집하는 양식과 유사한 UI인 측면 서랍 UI 이다.

측면 서랍에 렌더링하는 내용은 현재 선택된 행에 따라 달라진다. 이는 상태에 저장되어야 한다. 이 상태를 배치하는 가장 논리적 위치는 측면 서랍 구성 요소 자체 내에 있다. 사용자가 다른 셀을 선택하면 측면 서랍에만 영향을 주기 때문이다. 하지만,

- 테이블 구성 요소에서 이 상태를 *설정* 해야 한다. 우리는 `react-data-grid`테이블 자체를 렌더링하는 데 사용하고 있으며 사용자가 셀을 선택할 때마다 호출되는 콜백 소품을 허용함. 현재는 해당 이벤트에 응답할 수 있는 유일한 방법.
- 하지만 측면 서랍과 테이블 구성요소는 형제이므로 서로의 상태에 직접 액세스할 수 없다.

그럼, React 에서는 상태를 가까운 조상으로 끌어올리라고 한다. 하지만 그렇게 하지 않았음.

1. `TablePage`어떤 상태도 포함하지 않았으며 주로 테이블과 측면 서랍 구성 요소를 위한 컨테이너였으며 둘 다 props를 받지 못했다.
2. 우리는 이미 컴포넌트 트리의 루트에 가까운 [컨텍스트를](https://reactjs.org/docs/context.html) 통해 많은 "글로벌" 데이터를 공유하고 있었으며 이 상태를 중앙 데이터 저장소에 추가하는 것이 합리적이라고 느꼈다.

문제는 Context API를 통해 상태를 관리하면 업데이트로 인해 전체 앱이 리렌더링 된다는 점이다. 한 번에 수십 개의 셀이 표시될 수 있고, 각 셀에는 자체 편집기 컴포넌트가 있다. 이로 인해 약 650ms 렌더링 시간이 발생하고, 애니메이션에서 지연이 발생한다.

![1_DPrtPDYRTq3IBR9_Hsh6dQ](https://github.com/pozafly/TIL/assets/59427983/9f55ad6b-1caa-4fc5-ba6f-917ad15ef96a)

<br/>

## 아하 모먼트

1. 컨텍스트 분할.
2. React.memo로 래핑하고 `useMemo` 를 사용한다.
3. 테이블을 렌더링 하는 데 사용된 컴포넌트를 메모한다.

하지만, 이렇게 하면 코드를 완전히 수정하거나 리팩토링하는데 더 많은 시간을 소요해야만 했다.

`useRef` 를 사용하는 것이다. ref 값은 `.current` 속성을 변경해도 리렌더링하지 않는다.

<br/>

## 해결책

1. 열린 상태와 셀 상태를 측면 서랍에 보관한다.
2. 해당 상태에 대한 참조를 생성하고 컨텍스트에 저장한다.
3. 사용자가 셀을 클릭하면 테이블의 참조를 사용하여 상태 설정 함수(사이드 서랍 내부)를 호출한다.

![image](https://github.com/pozafly/TIL/assets/59427983/62103538-b926-46e7-ae00-199431075e5b)

```jsx
import { SideDrawerRef } from 'SideDrawer'

export function ProjectContextProvider({ children }) {
  const sideDrawerRef = useRef<SideDrawerRef>();

  return (
    <ProjectContext.Provider value={{ sideDrawerRef }}>
      {children}
    </ProjectContext.Provider>
  )
}
```

```jsx
export type SideDrawerRef = {
  cell: SelectedCell;
  setCell: React.Dispatch<React.SetStateAction<SelectedCell>>;
  open: boolean;
  setOpen: React.Dispatch<React.SetStateAction<boolean>>;
};

export default function SideDrawer() {
  const classes = useStyles();
  const { tableState, dataGridRef, sideDrawerRef } = useProjectContext();

  const [cell, setCell] = useState<SelectedCell>(null);
  const [open, setOpen] = useState(false);
  sideDrawerRef.current = { cell, setCell, open, setOpen };
  
  ...
}
```

즉, ref 값을 ContextAPI로 끌어올린 것이다.
