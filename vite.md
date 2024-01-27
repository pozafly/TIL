# Vite

회사에서 vue-cli로 만들어진 프로젝트를 Vite로 마이그레이션을 진행 중이다. [vue-cli](https://github.com/vuejs/vue-cli)는 2년 전부터 maintenance mode에 들어갔고, webpack을 기반으로 만들어졌기 때문에 번들링 및 빌드할 때 무척 느리다. vue-cli 내부에 묶여있는 eslint, prettier 등의 도구도 latest 버전과 호환이 되지 않았다.

예전부터 사용해보고 싶었던 번들러는 Vite다. Vite의 가장 큰 특징은 devServer를 ESM을 통해 제공하기 때문에 번들링을 진행하지 않고 코드를 제공한다. 그렇기 때문에 devServer가 cold-start될 때 현재 페이지에서 필요한 정보만 아주 빠르게 실행시킬 수 있다. Vite 가이드에서 가장 먼저 이야기 하고 있는 것이 ESM을 통한 devServer start 속도이다.

오늘 글은, 번들러 및 개발 환경을 vue-cli에서 Vite로 전환하며 궁금했던 개념을 약간 정리해보는 차원으로 적어보고자 한다.

<br/>

## Vite 환경 변수

Vite는 환경 변수가 정의된 파일(`.env`)에서 여타 번들러와 마찬가지로 어플리케이션에 필요한 변수를 가져와 사용할 수 있다. 

### Node.js

대부분의 bundler는 Node.js 환경 위에서 동작한다. Node.js에서는 `process` 라는 특별한 이름의 예약어를 사용해 환경 변수에 접근할 수 있다. `process` 예약어는 현재 실행되고 있는 Node.js 프로세스에 대한 정보를 가지고 있고 이를 제어할 수 있는 객체다.





그리고 대부분은 `dotenv` 라는 라이브러리를 내부적으로 래핑해서 사용하고 있다. vue-cli는 @vue/cli-services 패키지 내에서 사용하고 있고, CRA 같은 경우 react-scripts 라이브러리 내부 package.json에 `dotenv` 을 사용하고 있으며 Next.js도 내부적으로 dotenv 라이브러리를 이용해 `.env` 파일을 가져오고 있다.

dotenv는 node.js 환경에서 동작하도록 되어 있다.

Vite의 경우 ESM 환경에서 코드 조각을 들고오는데, 





그리고 환경 변수는 `process.env` 로 접근할 수 있다.





Node.js 모듈 환경과 브라우저 모듈 환경은 각기 CJS, ESM으로 나뉘어 있고, 







> ※ [--env-file](https://nodejs.org/dist/latest-v20.x/docs/api/cli.html#--env-fileconfig)
>
> node.js 20 버전 부터는 `--env-file=config` 를 통해 `.env` 파일을 `dotenv` 라이브러리 없이 사용할 수 있도록 개발 중이다. 실제로 터미널에서 node 명령어로 `.evn` 파일을 지정해줄 수 있다. 아직은 `실험용`.
>
> ```sh
> $ node --env-file=.env --env-file=.development.env index.js
> ```
>
> ```
> // .env
> PORT=3000
> ```
>
> 













import.meta로 node.js와 browser 모두 meta 정보에 접근할 수 있다.

근데 vite에서는 왜 dotevn 를 쓰는걸까? env를 단순히 읽어들이기 위해서일까?

vite로 플젝을 실행해서 source 탭에 트랜스파일링 된 파일 자체를 보면

```javascript
import.meta.env = {
    "VITE_TEST": "test!",
    "BASE_URL": "/",
    "MODE": "development",
    "DEV": true,
    "PROD": false,
    "SSR": false
};
```

이렇게 되어 있다. 즉, 브라우저 외부로 노출되어 있다. 따라서 어떤 변환 없이 브라우저 ESM으로 접근할 수 있다. 하지만, 서로 모듈이 다를 경우도 그렇게 되는가? 서로 어떻게 접근하게 되는가?

아래를 참조하자.

- import.meta가 proposal에 올라간 사유 : https://www.proposals.es/proposals/import.meta

- How to Access ES Module Metadata using import.meta : https://dmitripavlutin.com/javascript-import-meta/

- https://blog.bitsrc.io/understanding-import-meta-your-key-to-es-module-metadata-e2911b0d3c00

