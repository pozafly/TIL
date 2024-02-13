# openapi generator

> [출처](https://yozm.wishket.com/magazine/detail/2387/)

## 적합한 라이브러리 찾기

### 라이브러리 기준

첫 번째 기준은 사용자 수, 깃허브 star 또는 npm 위클리 다운로드 수 추측 가능. 사용자가 많은 것은 그만큼 사용성이 높다는 의미로 해석할 수 있다.

만약 위클리 다운로드 횟수가 꽤 많아도 마지막 업뎃이 몇년 전이라면 제대로 관리되지 않는 상태일 수도 있다. 이 경우 라이브러리를 사용하는 도중 버그가 발생했을 때 패치가 되지 않을 수 있음. 이럴 땐 직접 라이브러리를 고쳐 쓰거나, 새로운 라이브러리를 다시 찾아야 하므로 번거로워진다. swagger-typescript-api는 후보 중 위클리 다운 횟수가 가장 높진 않았다. 그러나 10만회 이상으로 절대적 수치가 꽤 높다고 판단, 후보에서 제외하지 않았음.

두 번째 기준은 제공되는 옵션을 활용해 사용하고 있는 언어의 장점을 잘 살릴 수 있는지다. TypeScript 사용하는 입장에서, 제너레이터가 타입을 잘 추출해주는지가 중요함. 서버에서 정의해준 request body, response body, enum, error 타입 등을 제너레이터가 잘 추출해주고, 그렇게 생성된 파일을 클라이언트에서 쉽게 사용할 수 있기를 원했다. 재가공할 필요 없이 명령어 하나에 바로 호출해서 쓸 수 있는 api.ts 파일이 생성되는 것.

### 옵션

명시적 타입 추출을 위한 옵션 찾기가 어려움. 많은 OAS 라이브러리 스펙 명세서를 보면, ts에 관한 스펙을 비교적 최신에 추가했다. ts는 다른 개발 언어에 비해 역사가 짧기 때문. 또 한가지는 복잡성이다. 대부분 라이브러리에서 제공하는 아웃풋 형태가 예상보다 복잡함.

### 후보

- [openapi-typescript](https://www.npmjs.com/package/openapi-typescript)
- [@openapitools/openapi-generator-cli](https://www.npmjs.com/package/@openapitools/openapi-generator-cli)
- [swagger-typescript-api](https://www.npmjs.com/package/swagger-typescript-api)

타입 추출 옵션, 그리고 단순히 API 메서드들만 정의되는 것이 아니라, 가능하면 API 호출을 수행할 HTTP 클라이언트 인스턴스도 자동 생성 조건이다. 그리고 fetch 대신 axios를 주로 사용하기에 axios 인스턴스와의 호환성도 좋아야 함.

이 지점에서 swagger-typescript-api가 좋다. ts에 최적화 되어있는 이 제너레이터는 모든 기능을 갖고 있다. 옵션을 조함해 작성한 스크립트는 아래와 같음.

```js
// api generator using swagger-typescript-api

import { exec } from 'shelljs';
const swaggerPath = `${__dirname}/../packages/api/swagger.json`;
const targetPath = `${__dirname}/../packages/front/src/api`;

async function generateApi() {
    exec(`npx swagger-typescript-api \
    -p ${swaggerPath} \                   // json 또는 yaml 파일
    -o ${targetPath} \                    // api 파일을 만들 목적지 디렉토리
    -n api.ts \                           // 파일 이름
    -r \                                  // 요청과 응답에 대한 메타정보를 자세하게 생성
    --axios \                             // 내장된 axios 클라이언트 인스턴스 사용
    --extract-request-params \
    --extract-request-body \
    --extract-response-body \
    --extract-response-error \
    --extract-enums \                      // 타입 추출
    --unwrap-response-data                 // response 객체에서 data 객체를 꺼내어 해줌
    `)
}

generateApi();
```

<br/>

