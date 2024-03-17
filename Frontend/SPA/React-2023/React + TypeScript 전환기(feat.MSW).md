# React + TypeScript 전환기(feat.MSW)

> 출처: https://saramin.github.io/2024-03-12-comment-for-typescript-react-conversion/

학습한 것들을 조금씩 서비스에 반영하며 점진적으로 개선해 가는 것이 아니라, 제한 된 시간 안에 학습해온 것을 한 순간에 쏟아내어 온통 새로운 기술 스택으로 서비스를 완성해 가야 할 때는 상당히 적지 않은 부담을 느끼며 새로운 기술 스택에 적절한 설계와 방법론을 고민해야 하기 때문에 도전이 된다.

<br/>

## TS 도입

문제점

- 팀 내 개발 프로세스에서 테스트가 작성되고 있지 않고, 테스트를 도입하기에는 아직 역량이 충분하지 않다.
- 코드 리뷰 단계에서 예외 처리 누락이나 오탈자로 인한 오류가 종종 발견되고, 자주 발경되는 만큼 코드 리뷰에서 검증해야 할 사항이 많아진다.
- 컴포넌트, 함수 등에 대한 문서화가 이루어지지 않아 코드 레벨에서 사용법을 확인하거나 스토리북에서 사용할 컴포넌트를 찾아 다니느라 낮은 생산성이 지속된다.

이런 문제를 해결하기 위해서는 TS가 가장 좋은 대안.

### Type 관리 전약

Type을 크게 두 가지 카테고리로 나누어 관리하도록 작성 되었다.

컴포넌트 props 및 컴포넌트에 종속되어 있는 types는 각 컴포넌트 디렉토리 안에 위치하여 local types로 관리하고, 그 외 Router, API fetcher, React Query, utils, 3rd party 등에 필요한 types는 types 디렉토리 안에 위치하여 global types로 관리하도록 작성되었다.

![image](https://github.com/pozafly/TIL/assets/59427983/5ef68cce-5683-4344-9f13-2d52c0633a91)

좌: Badge 컴포넌트 디렉토리, 우: types 디렉토리

초기에는 type 정의를 각 컴포넌트 및 모듈 파일에 직접 작성하고 있었는데, 다음의 상황들을 만나게 됨.

- import 되는 컴포넌트 및 모듈 개수 만큼 type import 문이 작성되면서 import 문이 화면 절반 이상을 차지하는 상황이 발생한다.
- 문서화를 위해 porps type에 작성된 JSDoc 어노테이션으로 인해 type 정의 부분이 화면 한 페이지를 차지할 만큼 길어지는 상황이 발생한다.
- 단일 파일에 여러 개의 함수 등이 작성되는 경우, 연관된 type과 코드를 가까이 배치시키면 경계를 명확히 하기 위한 추가 장치가 필요해지는 문제가 발생하고, type 정의와 코드 모음을 분리하여 배치시키면 수정 시 종종 위치 찾기를 위해 스크롤 이동으로 인한 스위칭 비용이 높아지는 문제가 발생한다.

첫 번째 문제를 해결하기 위해 모든 type을 global type으로 변경해봤는데,

이번에는 storybook에서 정의된 props들을 표현하지 못하는 문제가 발생함. storybook github issue에서 local types만 지원하고 있다는 내용이 발견되어 컴포넌트에 한하여 local로 사용하게 되었다.

```ts
import type { HTMLAttributes } from 'react';

let VALID_HEADING_LEVEL: readonly [1, 2, 3, 4, 5, 6];

declare global {
  type HeadingLevel = (typeof VALID_HEADING_LEVEL)[number];
}

export interface HeadingProps extends HTMLAttributes<HTMLHeadingElement> {
  /**
    * the level of heading
    */
  level: HeadingLevel;
}
```

Heading 컴포넌트에 정의한 type 사례. 컴포넌트르 벗어나 다른 곳에서 쓰일 소지가 있는 것들은 global로 정의된다.

두 번째와 세 번째 문제를 해결하기 위해 타입 정의 부분만 별도 파일로 분리하여 types 디렉토리에서 관리하고 있는데, 스크린샷에서 보이듯 관심사별로 묶어 타입을 모아두는 방식을 택하고 있다.

```ts
// types/services.d.ts
declare interface SaraminApiError {
  message: string;
  code: string;
}

declare interface SaraminApiResponse<T> {
  success: boolean;
  error: SamainApiError | null;
  data: T;
}

// src/services/CalendarApis.ts
export const getThemes = (
	params?: object,
): Promise<SaramApiResponse<ThemesResponse>> => { ... };

// src/hooks/useCalendarApis.ts
export const useGetThemes = (
  params?: object,
  config?: Omit<
    UseQueryOptions<
      SaraminApiResponse<ThemesResponse>,
      AxiosError<SaraminApiResponse<null>>,
      ThemesTransform
    >,
    "queryKey"
  >,
) => { ... }
```

API 응답 type 사례. 이렇게 정의해두면 어느 API Fetcher, React Query hooks 등에서든 import 없이 바로 사용할 수도 있고, 인텔리센스에서드 즉시 확인할 수 있었다.

컴포넌트도 types 디렉토리에 위치시켜 관리하도록 시도했는데, 컴포넌트 별로 작성하게 되면 types 내의 파일이 방대해지기도 하고 일부 파일은 3~4줄로 작성이 그치는 경우도 있다. 아토믹 시스템에서의 수준별로 묶어도 봤지만, 특정 컴포넌트에서만 요구되는 literal type이나, 네이밍이 겹치는 것을 피하기 위한 네이밍 규격을 추가해야 하는 등의 고려사항이 더 발생해 결국 아직까지는 썩 와닿는 좋은 방법을 찾지 못해 결국 컴포넌트에 필요한 types는 컴포넌트 디렉토리에서 관리 하도록 구성되었다.

### TS를 도입하면서 어려웠던 점

내부에서 TS 관련 린트를 타이트하게 잡으면 TS에 제법 많은 시간을 할애하게 되어 일정에 영향을 줄 수 있을거란 조언도 있었지만, 의외로 TS 도입으로 인해 일정이 영향을 받는 일은 없었다.

TS를 도입하게 되면 type 정의하느라 더 많은 키보드 타이핑을 해야하고 더 많은 시간이 소비된다는 이야기들이 지금도 있는 것으로 아느넫, 오히려 자잘한 오타로 인한 오류나 이벤트 핸들러 속에 숨어 발견되기 어려운 오류를 방지하기 위해 눈을 부릅뜨고 코드 리뷰해야 하거나 뒤늦게 오류를 해결하느라 소비하는 시간 등을 따지면 더 많은 시간이 절약 된다.

다만, 어려운 것은 외부 라이브러리 사용할 때다.

![image](https://github.com/pozafly/TIL/assets/59427983/0fcefa3e-55b2-4f0c-99cd-0928dd26f9db)

타입을 정의해 가는 중, 처음에는 어마무시한 양으로 느껴지는 오류 메시지.

React Query를 사용하며 Custom hook을 만들어 적용할 때 요구되는 type을 작성하느라 진땀이 남. 췩에는 type이 일치하지 않다는 메시지를 올바르게 해석해내지 못하고 해멤.

### TS 도입으로 얻은 좋은 점

- 코드 리뷰에서 불필요한 리소스가 조금 더 줄었다.

정적 타입을 사용하지 않던 이전 환경에서 코드리뷰를 하면 혹시나 숨어 있을지 모를 누락된 예외 처리, 오탈자로 인한 오류까지도 함께 검사해야 하거나 꼼꼼히 살펴보지 않으면 리뷰어 조차도 놓치기 쉽기 때문에 제법 많은 리소스가 발생되었는데 정적 검사가 이를 대신하여 경고를 띄워 리뷰 전 단계에서 해결 할 수 있게 되어 더 본질적인 것에 집중하여 리뷰가 가능해졌음.

때로는 컴포넌트 먼저 개발 이후 API fetcher를 개발하면서 가공된 예외가 발생하거나 타입의 불일치로 일어나는 문제를 실제로 구동해볼 때에야 확인할 수 있거나 리뷰 시 관련 파일을 열어두고 배교해야 했지만, TS에서 미리 API 응답 데이터의 타입을 정의해두고 시작하면서 이런 문제를 사전 방지.

- 개발 속도가 더 빨라짐

컴포넌트 가져다 쓰려면 필수 props가 문지 확인하기 위해 한쪽에 컴포넌트 코드를 열어두거나 storybook을 띄워두고 찾아다니거나 해야 했지만, TS도입 이후로 IDE 인텔리센스로 손 쉽게 처리할 수 있게 되었음.

![image](https://github.com/pozafly/TIL/assets/59427983/2ca8a3b0-90af-4993-bc23-ba857125b5ed)

- 가독성이 좋아짐.

함수를 어떻게 사용해야할 지 이해가 더욱 쉬워짐.

 overloading을 사용해 함수를 작성할 경우가 있는데, 파라미터 순서에 따라 타입이 다른 경우, 파라미터 이름만으로 가독성을 지원하는데 한계가 있는 경우, TS 통한 overloading 표현으로 IDE가 적절하게 지원하며 함수를 이해시키기 위한 문서를 추가로 작성하거나 문서가 없는 함수를 이해하기 위해 코드 내부를 들여다보아야 하는 일을 줄일 수 있게 되었음.

```ts
/**
  * date format의 문자열, 날짜 객체로도 비교 가능하고,
  * 주어진 연월 또는 연월일로도 비교 가능한 함수를 만들다보니,
  * 시간이 흐른 뒤, 이게 뭐하던 건지 파악하려면 작성한 나도 코드를 따라가며 해석해야 했다.
  */
function isSameDay(other, year, month, date) {
  if (typeof year === `string`) {
    return other.setHours(0, 0, 0, 0) === new Date(year).setHours(0, 0, 0, 0);
  }
  if (year instanceof Date) {
    return other.setHours(0, 0, 0, 0) === year.setHours(0, 0, 0, 0);
  }
  if (typeof year === `number` && typeof month === `number`) {
    return (
      other.setHours(0, 0, 0, 0) === new Date(year, month, date || 1).setHours(0, 0, 0, 0)
    );
  return false;
}
    
/**
  * type과 함께 overload로 표현하면서 평안이 찾아왔다
  */
function isSameDay(other: Date, date: string | Date): boolean;
function isSameDay(other: Date, year: number, month: number): boolean;
function isSameDay(other: Date, year: number, month: number, date: number): boolean;
function isSameDay(
  other: Date,
  year: number | string | Date,
  month?: number,
  date?: number,
) { ... }
```

<br/>

## MSW

API 모킹. service worker가 HTTP 요청을 가로채 등록해둔 모의 응답을 전달해주는 기능을 함.

![image](https://github.com/pozafly/TIL/assets/59427983/7bdf5972-db70-40fd-9312-61ae566fa910)

Storybook과도 통합 가능하고, request에 따라 응답을 변경할 수도 있기 때문에 FE 개발에서는 좋음.

2월 기준으로 msw-storybook-addon이 아직 msw 1.x 버전만 지원하고 있기 때문에 msw 역시 1.x 버전을 사용해야 함. 적용 방법 및 Storybook과의 통합 방법은 [msw 공식 문서](https://v1.mswjs.io/docs/)와 [storybook addon 문서](https://storybook.js.org/addons/msw-storybook-addon/)에 잘 정리되어 있어 그대로 따라하기만 해도 됨.

이미 만들어진 API를 사용하는 부분도 있고, 앞으로 수정 될 응답을 사용해야 하는 경우도 있었는데요, msw를 통해 실제 API를 호출하여 작동하는 것과 같은 효과를 가질 수 있기 때문에 저희처럼 API에서 응답되는 실제 데이터를 미리 넣어두고 사용하거나 앞으로 생성 될 혹은 변경 될 규격에 맞춰서 데이터를 넣어두고 개발하면 API 개발자의 개발 완료 시점을 기다리지 않아도 되게 되었습니다.

```ts
const recruitSchedule: SaraminApiResponse<ScheduleRecruitsResponse> = {
  success: true,
  error: null,
  data: {
    list: {
      "20240119": [
        {
          company_nm: "사람인",
          title: "사람인 FE개발팀 신입 채용",
          ...,
          company_logo_url: "",
          meta_tag: "서울 구로구 · 무관", // 추가될 데이터 
        },
        {
          company_nm: "사람인",
          title: "사람인 FE개발팀 신입 채용 2",
          ...,
          company_logo_url: "",
          meta_tag: "", // 빈 문자열로 올지도 모르니까 일단...
        },
        {
          company_nm: "사람인",
          title: "사람인 FE개발팀 신입 채용 3",
          ...,
          company_logo_url: "",
          meta_tag: "", // 혹시 property를 빼고 오는 경우가 있을 수도 있으니...
        },
```

msw 도입의 가장 큰 효과는 API의 실패 케이스를 미리 대응할 수 있는 점. API 응답에 따른 처리가 필요한 기능을 만들 경우, 성공 케이스에만 대응하고 실패 케이스를 놓치게 되거나 혹은 실패 상황을 만들기 어려워 상상 속에서 개발해야 하는 상황이 있음. msw를 도입하며 응답이 늦는 경우, 특정 HTTP status를 대응해야 하는 ㄴ경우, 오류 상황을 재현하는 스토리를 작성해야 하는 경우 등을 모두 과거에 비해 손쉽게 만들어내어 개발할 수 있게 되었음.

```ts
import { rest } from "msw";
import { ..., recruitSchedule } from "@mocks/data/calendar";

const handlers = [
  ...,
  // request를 가로채 parameter에 따라 다른 데이터를 줄 수도 있다
  rest.get(`/calendar/recruits/schedule`, (req, res, ctx) => {
    const monthParam = req.url.searchParams.get("month");
    const start = req.url.searchParams.get("startDate");
    const end = req.url.searchParams.get("endDate");
    const getMonthlyResult = (month: string) =>
      Object.entries(recruitSchedule.data.list).filter(([date]) =>
        date.includes(month),
      ) || [];
    const getWeeklyResult = (startDate: string, endDate: string) =>
      Object.entries(recruitSchedule.data.list).filter(
        ([date]) => date >= startDate && date <= endDate,
      ) || [];
    if (!monthParam && !(start && end)) return res(ctx.status(500), ...);
	
    return res(
      ctx.status(200),
      ctx.json({
        ...recruitSchedule,
        data: {
          ...recruitSchedule.data,
          list: Object.fromEntries(
            monthParam ? getMonthlyResult(monthParam) : getWeeklyResult(start, end),
          ),
        },
      }),
    );
  }),
  ...
];
```

<br/>

## HoC 패턴 적용

월간 달력이나 주간 달력이냐를 제외하면 나머지 부분은 동일한 형태를 유지함.

```tsx
// 월간뷰 페이지 구조 
fetchDatas...

handle user interaction...

<Themes themes={...} onClickXXX={ ... } />
<DateController date={...} onClickXXX={...} />
<NavigationTabBar date={...} subscribeList={...} />
<MonthlyCalendar  date={...} scheduleRecruitList={...} />
<Toolbar onClickXXX={...} />
<Recruitment scheduleRecruitmentMap={...} />
<Notice />

// 주간뷰 페이지 구조

fetchDatas...

handle user interaction...

handle weeklyCalendar interaction...

<Themes themes={...} onClickXXX={ ... } />
<DateController date={...} onClickXXX={...} />
<NavigationTabBar date={...} subscribeList={...} />
<WeeklyCalendar date={...} scheduleRecruitList={...} />
<Toolbar onClickXXX={...} />
<Recruitment scheduleRecruitmentMap={...} />
<MonthController />
<Notice />
```

각각 템플릿을 작성하고, 주간 템플릿에만 요구되는 추가 UI 기능들을 작성하는 방향을 떠올림.

하지만, 이내 나머지 영역이 모두 동일 비즈니스 로직을 가지고 동일한 기능을 가지는데 굳이 분리해 관리해야할 이유가 있을까 의문이 생김. 동일 부분을 각각 작성하는 순간 이후 유지보수가 발생 할 경우 양 파일 모두에 반영하고 코드 리뷰 시 리뷰어도 양쪽의 로직이 동일하게 적용되었는지 혹 빠뜨린 부분은 없는지도 리뷰해야하기 때문에 오히려 품질을 유지하는 것을 더 어렵게 할 것이라는 생각이 들었고, 변경 사항을 추적하는 것 역시 다소 복잡해질 소지가 있다는 판단이 들었음.

재사용 문제를 해결하기 위한 장치로 hook이 있지만, 이 경우에서는 하나의 hook이 서로 다른 관심사를 가지는 여러 컴포넌트를 반환하게 하는 형태가 되고 오히려 유지보수를 더 어렵게 만들 소지가 높을 것 같아 해결 방법에서 제외하였고, 이전에 글로만 봐왔던 High Order Component(HOC) 패턴을 사용하기로 했습니다.

```tsx
// src/hocs/withCalendarCommon.tsx
const CalendarCommon = <P extends ComponentProp>(Component) => {
  return (props: P) => {
    fetchDatas...
	  
    handle use interaction...
	  
    return (
      <>
        <Themes themes={...} onClickXXX={ ... } />
        <DateController date={...} onClickXXX={...} />
        <NavigationTabBar date={...} subscribeList={...} />
        <Component {...props} />
        <Toolbar onClickXXX={...} />
        <Recruitment scheduleRecruitmentMap={...} />
        <MonthController />
        <Notice />
      </>
    );
  };
}
```

```tsx
// src/pages/theme.tsx

import MonthlyTemplate from "@components/Templates/Monthly";
import WeeklyTemplate from "@components/Templates/Weekly";
import withCalendarCommon from "@hocs/withCalendarCommon";

const ThemePage = () => {
  const { week } = useParams();
  // something to do only on theme page ...

  const Component = week
    ? withCalendarCommon(WeeklyTemplate)
    : withCalendarCommon(MonthlyTemplate);

  return <Component />;
};

export default ThemePage;
```

상이한 부분만 템플릿 화 하여 동일 로직을 사용하는 부분은 HoC 적용

이런 방법으로 동일 비지니스 로직과 동일 UI 기능 및 동일하게 사용되는 컴포넌트를 더 이상 두 벌에서 관리하는 문제를 해결할 수 있을 것으로 보여짐. 다만 HoC의 사용이 많아질 경우 props 추적이 복잡해짐. 중첩되는 일 없도록 규칙을 정해야 함.

<br/>

## 마무리

완전히 새로운 스택을 이용해 프로덕트를 만드는 것은 처음 FE 개발자로 발걸음 을 떼었던 때와 유사한, 학습한 것만으로 충분히 해결 될 수 있을까? 새로운 것 옆에 새로운 것들의 조합에서 문제가 발생했을 때 빠르게 원인을 찾아서 해결할 수 있을까? 등 묵직한 부담을 어깨에 올려두고 낮섦과 마주함.

막연한 부담은 우려했던 것보다 더 많이 어렵지 않게 다가와 부담을 지워낼 수 있었고, 뒤로 갈 수록 '이렇게 하면 더 좋겠구나' 싶은 부분도 보임.

