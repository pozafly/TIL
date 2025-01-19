# FOUC, FOUT

> [출처](https://tooo1.tistoryㅇ.com/608)

## FOUC(Flash Of Unstyled Content)와 FOUT(Flash of Unstyled Text)

웹 페이지 로딩 과정에서 일시적으로 스타일이 적용되지 않은 콘텐츠와 텍스트를 사용자에게 보여주는 현상이다. 두 현상 모두 스타일 로드 및 적용 시점과 브라우저 렌더링 시점 간 차이 때문에 발생한다.

- FOUS: 일반적인 콘텐츠
- FOUT: 웹 폰트

에 대한 스타일 적용 지연 때문에 발생한다.

### FOUC

![image](https://github.com/pozafly/TIL/assets/59427983/c2f7af5a-8cb0-4304-ace6-26dbe9770920)

브라우저가 HTML 문서를 파싱하고 DOM 트리를 구성하는 동안, 외부 CSS 파일이 아직 로드되지 않았을 때 발생한다. 이로 인해 브라우저는 초기에 스타일이 적용되지 않은 콘텐츠를 화면에 표시하게 된다. 외부 CSS 파일이 로드되고 파싱되어 CSSOM 트리가 만들어지만, 렌더 트리가 업데이트 되고 스타일이 적용된 콘텐츠가 표시된다. 이과전에서 사용자는 잠시 스타일이 적용되지 ㅇ낳은 콘텐츠를 볼 수 있게 된다.

![img](https://github.com/pozafly/TIL/assets/59427983/e2da72c2-c24f-40e7-9867-c29994960193)

### FOUT

![img (1)](https://github.com/pozafly/TIL/assets/59427983/5eeeedc6-a61c-4349-a94b-3f11bf5f6f37)

웹 폰트를 사용하는 경우, 브라우저가 웹 폰트 파일을 다운로드하고 파싱하는데 시간이 걸릴 수 있음. 웹 폰트가 아직 로드되지 않았을 때, 브라우저는 시스템 기본 폰트를 사용하여 텍스트를 먼저 렌더링한다. 웹 폰트가 로드되면, 브라우저는 렌더 트리를 업뎃 해 웹 폰트를 적용한 텍스트를 표시하게 된다. 이 과정에서 사용자는 잠시 웹 폰트가 적용되지 않은 텍스트를 볼 수 있게된다.

<br/>

## FOUC, FOUT 발생 단계(브라우저 렌더링)

```
HTML 파싱: 브라우저는 HTML 파일을 파싱하여 DOM(Document Object Model) 트리를 생성합니다.
CSS 파싱: 브라우저는 CSS 파일을 파싱하여 CSSOM(CSS Object Model) 트리를 생성합니다.
렌더 트리 생성: DOM 트리와 CSSOM 트리를 합쳐서 렌더 트리를 생성합니다. 이 트리는 페이지의 시각적 요소를 나타냅니다.
레이아웃: 렌더 트리의 요소들에 대한 위치와 크기를 계산합니다.
페인팅: 최종적으로 스타일과 레이아웃 정보를 바탕으로 화면에 요소를 그립니다.
```

![image](https://github.com/pozafly/TIL/assets/59427983/0553adb4-bbd9-4223-a778-8519525c853e)

FOUC, FOUT 현상은 주로 렌더 트리 생성 및 레이아웃 단계에서 발생한다. 스타일 정보가 늦게 도착하거나, 웹 폰트가 로드되는 동안 브라우저가 기본 폰트로 렌더링 하기 때문이다.

<br/>

## FOUC와 FOUT 현상을 극복하기 위한 최적화 전략

### CSS를 HTML 문서의 `<head>` 태그에 배치

HTML을 순차적으로 해석하므로, 스타일을 가능한 빨리 로드하려면 `<head>` 태그 내 `<link>` 태그를 사용해 CSS 파일을 연결하는 것이 좋다. 이렇게 하면 브라우저가 렌더링을 시작하기 전 스타일을 먼저 로드하고 적용할 수 있음.

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>퉁이리</title>
    <link rel="stylesheet" href="styles.css"> <!-- 여기 -->
</head>
<body>
    <!-- Your content here -->
</body>
</html>
```

### 인라인 CSS 사용

인라인 CSS는 별도 HTTP 요청 없이 페이지와 함께 로드되기 때문에 렌더링 속도가 빨라질 수 있다. 하지만 이 방법은 파일 크기가 커질 수 있으므로, 필수적 스타일 정보만 인라인으로 삽입하는 것이 좋다. 이를 부분적으로 적용하기 위해 크리티컬 CSS 라는 기법을 사용할 수 있음.

```
크리티컬 CSS란 웹 페이지의 초기 렌더링에 영향을 미치는 중요한 스타일을 인라인으로 삽입하는 방법입니다.
```

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>퉁이리</title>
    <style>
        /* Critical CSS: 페이지 초기 렌더링에 필요한 중요한 스타일 */
        body {
            font-family: Arial, sans-serif;
            background-color: white;
            margin: 0;
            padding: 0;
        }
        header {
            background-color: #333;
            color: white;
            padding: 1rem;
        }
    </style>
    <link rel="stylesheet" href="styles.css"> <!-- 나머지 스타일은 여기서 로드 -->
</head>
<body>
    <header>
        <h1>퉁이리</h1>
    </header>
    <!-- Your content here -->
</body>
</html>
```

### CSS 파일 크기 줄이기

CSS 파일 크기를 최소화 하면 FOUC, FOUT 현상을 줄 일 수 있음. 수동으로 할 수 있지만, CSS 압축 도구를 사용하면 더 효율적이다.

### 웹 폰트 로드 최적화

`<link>` 의 preload 속성을 사용해 웹 폰트를 미리 로드할 수 있음. CSS font-display 속성을 사용해 웹 폰트의 로드 및 렌더링 동작을 조절할 수 있음.

### JavaScript 로드 순서 조절

JavaScript 가 페이지 스타일 적용을 차단하거나 렌더링을 지연시키는 경우, FOUC, FOUT 현상이 발생할 수 있음. script를 body 태그 끝에 배치하거나 async, defer 속성을 사용해 로드 순서를 변경할 수 있음.

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>퉁이리</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <!-- Your content here -->

    <!-- JavaScript를 문서 끝부분에 위치시키기 -->
    <script src="your-script.js"></script>
    <!-- 또는 `async` 또는 `defer` 속성 사용 -->
    <script async src="your-script.js"></script>
    <script defer src="your-script.js"></script>
</body>
</html>
```

### 콘텐츠 숨겨놓고 로드 완료 후 표시하기

초기 로딩 시 콘텐츠를 숨기고, 페이지의 스타일과 스크립트가 완전히 로드된 후 콘텐츠를 표시하는 것이다. 예를 들어, 페이지 최상위 요소에 `opacity: 0` 또는 `display: none` 같은 스타일을 적용해 콘텐츠를 숨길 수 있음. 그런 다음 JavaScript를 사용해 페이지 로드가 완료되면 스타일을 제거 후 콘텐츠를 표시할 수 있음. 이 방법은 FOUC 현상을 완전히 제거할 수 있지만, 사용자가 빈 화면을 잠시 볼 수 있으므로 적절한 로딩 인디케이터를 제공하는게 좋다.

### SSR 사용
