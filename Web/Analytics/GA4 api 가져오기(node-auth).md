# GA4 api 가져오기(node-auth)

## 동기

- Google Analytics에서 통계 자료를 가져오고 싶었다.
- 블로그에 top 게시물을 보여주고 싶었기 때문.

<br/>

## 방법

node.js 클라이언트로 인증을 받고 api 호출을 하면된다. 이 과정에서 인증 과정이 여러 개이고, 조금 까다롭기 때문에 이 문서를 기록한다.

### GA4 Query Explorer에서 정보 가져오기 실험

- [GA4 Query Explorer](https://ga-dev-tools.google/ga4/query-explorer/)에서 내 계정과 GA에 연결되어 있는 프로젝트를 설정한다.
- start date, end date 설정
- metrics, dimensions 설정
- (options) limit 설정
- `MAKE REQUEST` 버튼 클릭

아래와 같이 호출했을 때,

```json
{
  "dimensions":[
    {
      "name":"unifiedScreenName"
    }
  ]
  "metrics":[
    {
      "name":"totalUsers"
    }
  ]
  "dateRanges":[
    {
      "startDate":"120daysAgo"
      "endDate":"yesterday"
    }
  ]
  "limit":"10"
}
```

<img width="806" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/1eb130b6-b173-4892-bfac-ab38047c5666">

이런 결과를 가져올 수 있다. 그럼, 소스코드에서 Analytics api를 호출하면 위와 같은 결과를 받아볼 수 있겠다.

<br/>

## Node.js 라이브러리 설정

먼저 GA4 기준으로 어떤 경로를 호출해야 하는지 알아보자.

이전에 GA3 기준으로는 [Reporting API v4](https://developers.google.com/analytics/devguides/reporting/core/v4?hl=ko)를 호출했어야 했다. 하지만, 2023/7/1 부터 표준 유니버셜 애널리틱스 속성의 데이터 처리가 중단됨에 따라 2024/7/1 부터 GA4 기준으로 api를 사용해야만 한다. 즉, [Analytics Data API v1](https://developers.google.com/analytics/devguides/reporting/data/v1?hl=ko)로 대체되었다.

클라이언트(브라우저) 에서 호출하는 것은 [이곳](https://velog.io/@slight-snow/React.js-GA4Google-Analytics-API-%ED%98%B8%EC%B6%9C%ED%95%98%EA%B8%B0)에서 확인할 수 있다.

```jsx
import { useEffect } from 'react';
import axios from 'axios';

export default function Example() {
  useEffect(() => {
    axios
      .post(`https://analyticsdata.googleapis.com/v1beta/properties/${GA4_PROPERTY_ID}:runReport`,
            {
             "dimensions": [{ "name":"date" }],
             "metrics": [{ "name":"activeUsers" }, { "name":"screenPageViews" }, { "name":"sessions" }],
             "dateRanges": [{ "startDate":"2024-01-01", "endDate":"today" }],
             "keepEmptyRows": true
            }
      )
      .then((response) => {
        console.log(response);  
      })
      .catch((error) => {
        console.log(error);
      })
  })
}
```

하지만, Node.js에서는 조금 다르다.

Google Analytics의 [Analytics API 빠른 시작](https://developers.google.com/analytics/devguides/reporting/data/v1/quickstart-client-libraries?hl=ko#node.js_1)에서 살펴볼 수 있는데, 순서대로 하지말고 먼저 클라이언트부터 설치해보자.

### Google Analytics API를 호출할 Node.js 클라이언트 설치

> ※ Node.js 클라이언트는, 브라우저(클라이언트)가 아니다. Google Analytics API를 호출할 '클라이언트'로, Node.js 환경에서 동작하는 클라이언트를 말한다. 클라이언트(브라우저)와 헷갈리지 말자.

```sh
$ npm i -D @google-analytics/data
```

npm으로 패키지를 설치한다. 빠른 시작 페이지의 가장 하단에는 아래와 같은 코드가 있다.

```js
  /**
   * TODO(developer): Uncomment this variable and replace with your
   *   Google Analytics 4 property ID before running the sample.
   */
  // propertyId = 'YOUR-GA4-PROPERTY-ID';

  // Imports the Google Analytics Data API client library.
  const {BetaAnalyticsDataClient} = require('@google-analytics/data');

  // Using a default constructor instructs the client to use the credentials
  // specified in GOOGLE_APPLICATION_CREDENTIALS environment variable.
  const analyticsDataClient = new BetaAnalyticsDataClient();

  // Runs a simple report.
  async function runReport() {
    const [response] = await analyticsDataClient.runReport({
      property: `properties/${propertyId}`,
      dateRanges: [{ startDate: '2020-03-31', endDate: 'today' }],
      dimensions: [{ name: 'city' }],
      metrics: [{ name: 'activeUsers' }],
    });

    console.log('Report result:');
    response.rows.forEach(row => {
      console.log(row.dimensionValues[0], row.metricValues[0]);
    });
  }

  runReport();
```

여기에서 잘 봐야 할 부분은, `propertyId` 다. propertyId는 내가 입혀둔 Google Analytics의 프로젝트의 ID이다.

- Google Analytics의 우측 탭 하단의 '관리' 버튼을 누른다.
- 속성 -> 속성 세부정보로 들어간다.

<img width="589" alt="스크린샷 2024-05-05 오후 7 18 44" src="https://github.com/pozafly/TIL/assets/59427983/1ccd4fff-5f5e-4c01-8ade-f8d06a1103b6">

위에 가려진 부분이 `propertyId` 다. 복사해서 코드에 넣어보자.

<img width="1021" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/da92e188-9f23-464e-834b-347eba0496b5">

제대로 넣었음에도, propertyId가 권한이 없다고 나타난다. Google Cloud Console에서 Analytics API 사용 설정을 해주어야 한다.

### Google Cloud Console에서 Analytics API 사용 설정

1. [google cloud console](https://console.cloud.google.com/welcome?project=analytics-api-422407)에서 새로운 프로젝트를 만든다. (나는 analytics-api)
2. API 및 서비스 클릭

   <img width="1172" alt="스크린샷 2024-05-05 오후 7 08 43" src="https://github.com/pozafly/TIL/assets/59427983/614be949-bc05-445f-abf5-cc955b5546de">

3. 우측 탭에서 `라이브러리` 클릭
4. 검색에 'google analytics api' 검색

   <img width="595" alt="스크린샷 2024-05-05 오후 7 11 11" src="https://github.com/pozafly/TIL/assets/59427983/dade997c-bd18-4668-b1be-fe136b9b7837">

5. Google Analytics Data API 클릭 (다른거도 있는데 Data 붙은 녀석으로 해야 함)

   <img width="1022" alt="스크린샷 2024-05-05 오후 7 12 01" src="https://github.com/pozafly/TIL/assets/59427983/ae94ac32-405d-46a1-a3ad-4496925300ee">

6. '사용' 버튼 클릭

그럼 이제, 내 코드에서 이곳을 호출해, Analytics 정보를 가져올 것이다. 위 코드를 다시 한 번 호출해보자.

<img width="1007" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/0f5571e6-270f-48d1-ad2c-a9a0fed544a0">

오류구문이 조금 변했다. 이번엔 Permission Denied가 아니라, credentials 오류다. 이젠 접근을 위한 인증이 필요한 상황이다.

[Google Analytics Admin: Node.js Client](https://github.com/googleapis/google-cloud-node/tree/main/packages/google-analytics-admin)에 따르면 authentication이 필요하다고 나와있다.

<img width="797" alt="스크린샷 2024-05-05 오후 7 26 03" src="https://github.com/pozafly/TIL/assets/59427983/60dfd5f9-1437-424e-a298-d8fb193733e3">

링크를 타고 들어가보면, 클라이언트 라이브러리를 사용하기 위해서는 애플리케이션 기본 사용자 인증 정보(ADC)를 지원해야 한다고 한다. ADC란, Application Default Credentials라고 하는데, 사용자 인증 정보를 자동으로 검색하기 위해 Google 인증 라이브러리에 사용되는 전략이다.

즉, 로컬 컴퓨터에 인증 정보를 저장하고 (Mac의 키체인과 비슷) 인증이 필요한 서비스 즉, Google API 클라이언트 라이브러리를 동작시킬 때 자동으로 컴퓨터에서 인증 파일을 찾아 인증을 한 후 API를 호출시키는 것을 말한다.

이 [ADC](https://cloud.google.com/docs/authentication/provide-credentials-adc?hl=ko)를 설정해주기 위해서는 로컬 컴퓨터에서 `gcloud CLI` 를 통해 인증을 받아야만 한다. `$ gcloud auth application-default login` 이런 식의 cli가 계속해서 들어가기 때문에 배포된 서버 컴퓨터에서는 한 번만 인증하면 되지만, 내가 사용하려는 환경은 GitHub Actions의 러너 환경이기 때문에 이와 적합하지 않겠다고 생각했다.

왜냐하면 GitHub Actions는 실행 할 때마다 runner 환경이 새롭게 만들어지고 매번 gcloud cli를 통해 인증 정보를 매번 새롭게 만들어주어야 하기 때문이다.

### 인증 JSON 파일 만들기

그럼 gcloud 명령어가 아닌, 다른 방법으로 인증을 거쳐야 하는데, 이는 인증 JSON 파일을 통해 할 수 있다.

가장 먼저 '[서비스 계정](https://cloud.google.com/docs/authentication?hl=ko#service-accounts)'을 만들어주어야 한다. 서비스 계정은 어플리케이션에 Google Cloud 리소스 액세스가 필요할 때와 같이 '사람이' 직접 관련되지 않은 경우의 인증 및 승인을 관리하기 위한 방법이다. 따라서, GitHub Actions와 같이 사람이 직접 인증하지 않고 자동화 된 프로그램으로 인증 받을 수 있는 방법에 적합하다 볼 수 있다.

[서비스 계정 키 만들기](https://cloud.google.com/iam/docs/keys-create-delete?hl=ko#creating)를 참고하여 키를 만들어보자. [이곳](https://console.cloud.google.com/projectselector2/iam-admin/serviceaccounts?walkthrough_id=iam--create-service-account-keys&start_index=1&hl=ko&_ga=2.7475762.1639366599.1714905346-1092588540.1714895173&supportedpurview=project#step_index=1)에 들어가면 서비스계정을 만들 수 있는데 아까 만들어두었던 analytics-api 프로젝트를 선택하자.

<img width="402" alt="스크린샷 2024-05-05 오후 7 37 49" src="https://github.com/pozafly/TIL/assets/59427983/e570dbe3-8414-42f8-8131-7408de9891b5">

서비스 계정 만들기에 들어가서 계정 이름과 ID를 적어주면 iam.gserviceaccount.com 도메인으로 된 이메일 주소를 내어준다.

<img width="537" alt="스크린샷 2024-05-05 오후 7 40 27" src="https://github.com/pozafly/TIL/assets/59427983/6b9b45e9-a224-4dda-9878-64e489e11f8b">

나머지 두 개는 선택 사항이므로 나는 아무것도 선택하지 않고 완료했다.

만들어진 서비스 계정을 클릭하고 키 탭으로 넘어가면 아래와 같이 '키 추가' 버튼을 클릭해 키를 만들어준다.

<img width="509" alt="스크린샷 2024-05-05 오후 7 43 56" src="https://github.com/pozafly/TIL/assets/59427983/b8d0b100-f4e0-4db5-97c2-a06369648b61">

JSON 형태로 키를 만들면, 비공개키가 만들어지고, `analytics-api-,,,-,,,,d2,,,.json` 와 같은 형태의 json 파일이 자동으로 다운로드 받아졌을 것이다.

```
{
  "type": "service_account",
  "project_id": "analytics-api-댜3ㅜ3ㅓㄹㅎ93",
  "private_key_id": "어ㅑ덕리ㅏㅂㅈㄷ러ㅑ",
  "private_key": "-----BEGIN PRIVATE KEY-----\nMIIEvgI암호문암호문암호문암호문암호문",
  "client_email": "analytics@a,,,,.gserviceaccount.com",
  "client_id": ",,,,,9",
  "auth_uri": "https://accounts.google.com/o/oauth2/auth",
  "token_uri": "https://oauth2.googleapis.com/token",
  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
  "client_x509_cert_url": "https://www.googleapis.co...,
  "universe_domain": "googleapis.com"
}
```

이런 형태의 키가 생성되었다. 이제 Google API를 호출할 때 이 json파일을 참조하여 호출하면 API가 credentials 오류 없이 정상적으로 가져올 수 있다.

이제 2가지 설정만 더 해주면 된다.

Google Analytics의 관리에 '계정 액세스 관리' 탭으로 들어가자.

<img width="310" alt="스크린샷 2024-05-05 오후 7 49 02" src="https://github.com/pozafly/TIL/assets/59427983/d6713390-4eae-4f74-9a28-5ed408e18b15">

우측 상단의 + 버튼을 클릭해 Google Analytics에 접근할 수 있는 접근권한을 방금 만든 서비스 계정에 부여하는 과정이다.

<img width="219" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/77cbafe5-4ab4-47d1-9d3a-4225d8a78120">

`+` 버튼을 누르고 '사용자 추가' 를 누르면 '이메일 주소'를 입력하라고 나온다. 그러면 Google Cloud에서 만들었던 서비스 계정의 email 주소를 복사해 붙여넣어주자.

<img width="288" alt="스크린샷 2024-05-05 오후 7 51 41" src="https://github.com/pozafly/TIL/assets/59427983/44ff3882-36e8-4c14-9039-cd0435b411c9">

그리고 아래 사진과 같이 `뷰어` 권한만 주도록 하자. API 호출로 정보를 받아보기만 하면 되기 때문이다.

<img width="960" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/dd2db614-0556-4e67-873c-cb5bb0c4f7bd">

그러면 이제 정상적으로 아까 코드가 호출될 것이다.

```ts
const { BetaAnalyticsDataClient } = require('@google-analytics/data');
const analyticsDataClient = new BetaAnalyticsDataClient({
  keyFilename: './scripts/analytics/analytics-api-아까받은json파일임.json',
});

const propertyId = '프로퍼티아이디 넣어주라';

export async function runReport() {
  const [response] = await analyticsDataClient.runReport({
    property: `properties/${propertyId}`,
    dateRanges: [
      {
        startDate: '120daysAgo',
        endDate: 'yesterday',
      },
    ],
    metrics: [{ name: 'totalUsers' }],
    dimensions: [{ name: 'unifiedScreenName' }],
    limit: 10,
  });

  response.rows.forEach((row: unknown) => {
    console.log(row);
  });
}

runReport();
```

`analyticsDataClient()` 생성자에 객체 형태로 `keyFilename` key에 json 파일을 경로로 잡아주고 호출해보자. analyticsDataClient에 대한 정보는 [nodejs-analytics-data](https://github.com/googleapis/nodejs-analytics-data/blob/3f7ec96ce5530cbe8636bfd0aa50b7e57667801c/samples/quickstart_json_credentials.js#L40-L53) 레파지토리를 참고했다.

호출해보자.

<img width="870" alt="image" src="https://github.com/pozafly/TIL/assets/59427983/1871b2a6-c2ec-4960-8957-d69b528f0c76">

이와 같은 형태로 잘 출력된 것을 볼 수 있다.

그러면 이걸 활용해 블로그에 조회수 Top3의 정보를 매주, 매달마다 cron으로 GitHub Actions에서 돌려, ISR 형태로 사용자에게 제공할 수 있는 것이다. Good!
