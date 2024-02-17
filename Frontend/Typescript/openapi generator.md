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
- [swagger-typescript-api](https://www.npmjs.com/package/swagger-typescript-api)
- [@openapitools/openapi-generator-cli](https://www.npmjs.com/package/@openapitools/openapi-generator-cli)

<img width="1317" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/676bbd31-7995-41bf-a7e1-7d95b9184a30">

타입 추출 옵션, 그리고 단순히 API 메서드들만 정의되는 것이 아니라, 가능하면 API 호출을 수행할 HTTP 클라이언트 인스턴스도 자동 생성 조건이다. 그리고 fetch 대신 axios를 주로 사용하기에 axios 인스턴스와의 호환성도 좋아야 함.

이 지점에서 swagger-typescript-api가 좋다. ts에 최적화 되어있는 이 제너레이터는 모든 기능을 갖고 있다. 옵션을 조함해 작성한 스크립트는 아래와 같음.swagger-typescript-api

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

## swagger-typescript-api가 편한 이유

매우 친절함. 사용할 수 있는 옵션과 설명, 사용 예시까지 자세하게 안내하고 있음. 여러 언어가 아닌 타입스크립트만을 위한 라이브러리로, 사용자 니즈를 잘 반영한 것 같다. 명시적인 타입 추출이 매우 잘된다. 인풋이 되는 json, YAML 파일에 정의되어 있는 타입들을 API 제너레이팅 시점에 원하는 타입으로, 직접 정의해 새로 매핑할 수 있는 기능도 있음. 확장성 면에서 만족스럽고, 이 기능 역시 설명 및 예시가 자세하다.

Axios 와의 호환도 매우 좋다. Axios 옵션을 이용한 경우, Axios 클래스를 상속받은 HTTP 클라이언트가 만들어지고, 사용자 관점에서 Axios 인스턴스를 생성하는 것과 별반 다르지 않게 쓸 수 있다. 하지만, 관점에 따라 필수적인 기능이라 보긴 어렵다.

[openapi-typescript](https://www.npmjs.com/package/openapi-typescript) 는 자체적 HTTP 클라이언트를 제공하고 인터페이스도 매우 직관적이다. API 를 호출할 때 꼭 fetch API나, Axios 클라이언트를 고집할 필요는 없으므로, 라이브러리에서 자체적으로 제공하는 클라이언트를 사용하는 것도 나쁘지 않다.

swagger-typescipt-api 라이브러리를 통해 만들어지는 결과물은 매우 단순하고 직관적이다. 사실상 단 하나의 파일이 만들어진다. 규모가 작은 프로젝트에서 API 정의를 위해 많은 양의 파일을 만들고 싶지 않기 때문. 다른 라이브러리중 데이터 모델 인터페이스나 BaseAPI에 대한 파일이 별도로 만들어지는 경우도 있어 이쪽을 선호할 수도 있다.

<br/>

## 나에게 맞는 API 제너레이터

단순함을 선호한다면 swagger-typescript-api다.

- [openapi-typescript](https://www.npmjs.com/package/openapi-typescript)
- [swagger-typescript-api](https://www.npmjs.com/package/swagger-typescript-api)
- [@openapitools/openapi-generator-cli](https://www.npmjs.com/package/@openapitools/openapi-generator-cli)

중에, `@openapitools/openapi-generator-cli` 같은 경우 java runtime에서 동작하기 때문에 JVM을 mac에 또 따로 설치해주어야 한다. 그래서 무척 번거로움. 그렇기 때문에 그냥 openapi-typescript, swagger-typescript-api 둘 중 하나를 선택하는 것이 좋다. 이유는 node.js 기반으로 동작하는 것이기 때문이다.

다만, openapi-typescript 같은 경우는 옵션이 swagger-typescript-api에 비해 많지 않고, 단순히 type 정의만 해주는 목적으로 사용하는 것이 좋을 듯 하다. 물론, fetch api에 관련된 client도 따로 패키지로 제공하고 있다.

swagger-typescript-api는 fetch-api에 대한 client도 제공하고, axios 모듈에 관련한 client도 제공하고 있다. 그리고 다양한 옵션을 가지고 있다. 그래서 우선 swagger-typescript-api를 쓰는 것이 좋다고 생각할 수 있는데, 문제는 최근 라이브러리 업데이트가 없다는 점이 꽤나 아쉽다.

디스커션 또한 없으며, issue및 pr도 굉장히 많은 상태로 관리가 안되고 있다.
