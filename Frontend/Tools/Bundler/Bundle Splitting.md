# Bundle Splitting

> [출처](https://patterns-dev-kr.github.io/performance-patterns/bundle-splitting/)

webpack이나 rollup과 같은 번들러들은 앱의 소스 코드를 받고 이를 하나 이상의 번들로 만들어준다. 사용자가 웹 사이트에 접속하면 만들어진 번들들이 다운로드 되며 사용자의 화면에 데이터를 출력하게 된다.

[V8](https://v8.dev/docs)과 같은 자바스크립트 엔진들은 로드되는 동안 사용자가 요청 데이터에 대해 구문을 분석하고 컴파일할 수 있다. 최신 브라우저들은 코드들을 성능 좋고 빠르게 구문 분석하고 컴파일하도록 발전되어 왔지만, 요청 데이터와 관련된 코드에 대한 로딩 시간과 실행 시간은 아직 개발자가 직접 최적화 해야 한다. 이 '실행 시간'을 가급적 짧게 유지하여 메인 스레드를 블록하지 않도록 해야 한다.

최신 브라우저들은 번들들을 `스트리밍` 할 수 있지만, 사용자의 화면에 첫 픽셀을 그려내기까지 시간이 꽤 걸릴 수 있다. 만들어진 번들의 크기가 클 경우 화면에 무언가를 그려내는 코드까지 실행되는데 걸리는 시간이 많이 걸릴 수 있다. 그 전까지는 사용자는 빈 화면에 머무르게 되고 이 시간이 길어지면 서비스를 사용할 마음이 사라질 수 있음.

> 📌 스트리밍이란, 파일 전체를 한 번에 받는게 아니라 부분부분씩 받아 적용하는 것을 뜻한다.

사용자가 보는 화면에 데이터를 빨리 보여줄 필요가 있다. 번들 크기가 클수록 로딩 타임, 프로세스 타임, 실행 시간이 늘어난다. 따라서 번들 크기를 줄이는 것이 이런 시간들을 줄이는 방법이 되겠다.

<img width="952" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/0c987b6e-2180-4bae-be59-758b44ac1129">

번들 코드를 나누어 로드 및 실행 시간에 걸리는 시간을 줄임. 로드 및 실행시간을 줄여 사용자의 화면에 무언가 그리기 가지 걸리는 시간을 짧게 만들었다. 이는 성능 측정 항목 중 FCP(First Contentful Paint)를 향상시킨 것이며 이어서 화면에서 큰 영역을 차지하는 컴포넌트의 렌더링 속도도 향상시켰다. LCP(Largest Contentful Paint)를 향상시킨 셈이다.

화면에 무언가 빨리 나타나서 좋긴 하지만, 앱이 제대로 기능하려면 사용자의 인터렉션도 빠르게 가능해져야 함. 번들 파일들이 로드되고 실행 완료 되어야만 UI가 인터렉트가 가능해짐. TTI 라고 함.

<img width="924" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/8d8d982c-3e4b-4a16-8dab-78dac16e185e">

번들이 크다고 해서 실행시간이 항상 오래걸리는 것은 아닐 수 있음. 사용자가 실제로 사용하지 않는 코드들을 로드하더라도, 번들의 일부분은 특정 상호작용을 통해서만 실행될 수 있고, 해당 상호작용은 사용자가 할 수도 있고 안할 수도 있다.

JavaScript 엔진은 사용자의 화면에 아무것도 나오지 않는 시점이며 실제로 사용하지 않을지도 모르는 코드들도 구문분석하고 컴파일 해야 함. 브라우저 성능이 좋아 이 과정에 드는 비용은 높지 않을 수 있지만, 크기가 큰 번들을 다운로드 하도록 하는 것은 여전히 성능에 문제를 일으킬 수 있다. 사용자의 단말 성능이 떨어지거나 네트워크 상태가 불안정한 경우 사용자는 번들이 다운로드되기까지 꽤 많은 시간을 기다려야 할 수 있음.

<img width="844" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/36ed3f9b-373a-47f5-b6a5-483e0653485f">

실제로 사용하는 코드는 번들의 뒷 부분에 있지만, 첫 번째 부분도 로드되고 처리되어야 한다. 현재 페이지 탐색에서 별로 중요하지 않은 코드의 일부분들은 최초 렌더링 시 필요한 부분과 분리할 수 있음.

<img width="669" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/3d8e23c4-1695-44c2-8fde-4389a58ebf3f">

크기가 큰 번들을 `main.bundle.js`, `emoji-picker.bundle.js` 두 개의 작은 번들로 나누었다. 이로써 최초 로딩 시점에 더 작은 데이터만 받아도 되도록 하여 성능을 향상시켰다.

<img width="669" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/08bca7d0-d92b-4453-8d0a-b2b03d3c3c2e">
