# Bundle analyzer size(stat, parsed, gzip)

> [출처 issue](https://github.com/webpack-contrib/webpack-bundle-analyzer/issues/61#issuecomment-290634958)

`stat size`, `parsed size`, `gzip size` 3개가 있다.

## Stat size

통계(stat) 크기는 webpack의 통계 객체에서 직접 가져온다. 모듈의 실제 소스코드를 그대로 사용하고, minification(축소)나 gzip으로 압축 전 크기를 말한다.

이 크기는 소스 맵이나 추가적으로 처리되지 않은 기타 파일들을 제외한, 파일 시스템에 저장될 때의 크기입니다. 즉, 파일을 압축하지 않은 상태의 크기를 말합니다. 이 크기는 종종 개발 중에 빠른 참조로 사용되며, 최적화 전의 원시적인 크기를 이해하는 데 유용합니다.

## Parsed size

파싱된 크기는 컴파일된 번들 파일을 읽고 웹팩 통계 파일에서 모듈에 대한 링크를 다시 만들어 계산한다. 따라서 UglifyJS와 같은 압축기를 사용하는 경우 구문 분석된 크기는 축소 후의 크기를 보여줌. (웹팩이 트리쉐이킹을 마친 결과물이다.)

## Gzip size

컴파일된 번들 파일을 읽고 각 모듈 소스에 대해 개별적으로 gzip을 실행해 계산한다. gzip크기는 축소 및 압축 후 크기를 표시하지만, 각 모듈을 개별적으로 압축하면 개별 소스가 함께 압축될 기회가 적기 때문에 압축 측면에서 '이익'이 적기 때문에 실제 파일 크기와 1:1 매핑이 아니다.

## 활용 방안

- **Gzip Size**가 예상보다 크게 나온다면, 추가적인 압축 옵션을 고려하거나, 불필요한 코드를 제거하는 등의 최적화 작업이 필요할 수 있습니다.
- **Parsed Size**는 최적화 작업(예: 트리 쉐이킹이나 코드 분할)이 실제로 브라우저에서 처리해야 하는 코드의 양을 얼마나 줄였는지를 평가하는 데 유용합니다.
- **Stat Size**는 개발 초기에 참고할 수 있는 기본적인 크기지만, 실제 성능 최적화를 위해서는 Parsed Size와 Gzip Size에 더 주목해야 합니다.

<br/>

> ※ gzip이란?
>
> 파일을 압축하기 위한 SW임. 웹 개발에서 웹 서버와 브라우저 사이에서 전송되는 데이터의 크기를 줄이기 위해 사용된다. HTML, CSS, JavaScript 파일과 같은 텍스트 기반 자원들을 압축해 네트워크를 통해 데이터 전송 시간을 단축시키고 웹페이지 로딩 시간을 개선하는데 도움이 된다.
>
> ```nginx
> gzip on;
> gzip_disable "msie6";
> gzip_vary on;
> gzip_proxied any;
> gzip_comp_level 6;
> gzip_buffers 16 8k;
> gzip_http_version 1.1;
> gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
> ```
>
> nginx에서 위와 같이 사용할 수 있음.
>
> 이미 압축된 파일(예: 이미지)에 대해서는 gzip 압축을 하지 않는게 좋다. 오히려 파일 크기가 커지는 경우도 있음.
