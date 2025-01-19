# Image

> [출처](https://oliveyoung.tech/blog/2023-06-09/nextjs-image-optimization/), [공홈](https://nextjs.org/docs/pages/api-reference/components/image-legacy)

<br/>

## 공홈

### Src

- 정적 파일만 가능하다. (local 이미지. import 문으로 가져온)
- 외부라면, next.config.js 파일에 경로를 넣어주어야 한다.

### Width

- layout, sizes 속성에 따라 달라지는 원래 너비.
- `layout="intrinsic"` 또는 `layout="fixed"` 사용시 `width` 는 렌더링 된 너비를 픽셀 단위로 나타내어, 크기에 영향을 준다.
- `layout="responsive"`, `layout="fill"` 이라면, width는 원래 너비를 픽셀 단위로 나타내기 때문에 가로, 세로 비율에만 영향을 준다.
- width 속성은 정적으로 가져온 이미지 또는 `layout="fill"` 이미지를 제외하고 필수다.

### Height

- width와 동일하게 layout 속성에 따라 다르다.

### Layout

| layout              | 행동                                                 | srcSet                                                       | sizes     | has wrapper and sizer |
| ------------------- | ---------------------------------------------------- | ------------------------------------------------------------ | --------- | --------------------- |
| intrinsic (default) | 컨테이너 너비에 맞게 축소, 최대 이미지 크기까지 축소 | `1x`, `2x` (이미지 크기 기준)                                | 해당 없음 | Y                     |
| fixed               | 크기는 width, height에 정확히 맞음                   | `1x`, `2x` (이미지 크기 기준)                                | 해당 없음 | Y                     |
| responsive          | 컨테이너 너비에 맞게 크기 조정                       | `640w`, `750w`, … `2048w`, `3840w` (기준: [imageSizes](https://nextjs.org/docs/pages/api-reference/components/image-legacy#image-sizes) 및 [deviceSizes](https://nextjs.org/docs/pages/api-reference/components/image-legacy#device-sizes)) | `100vw`   | Y                     |
| fill                | 컨테이너를 채우기 위해 X축과 Y축 모두에서 늘림       | `640w`, `750w`, … `2048w`, `3840w` (기준: [imageSizes](https://nextjs.org/docs/pages/api-reference/components/image-legacy#image-sizes) 및 [deviceSizes](https://nextjs.org/docs/pages/api-reference/components/image-legacy#device-sizes)) | `100vw`   | Y                     |

- intrinsic: 작은 뷰포트 경우 작아지게 만듦. 큰 뷰포트는 원래 이미지를 보여줌.
- fixed: 뷰포트가 변경되어도 이미지 크기가 변경되지 않는다.(반응성 없음)
- responsive: 이미지 크기가 작은 뷰포트에서는 크기가 축소되고 큰 뷰포트에서는 크기가 확대된다.
- fill: 부모 요소가 상대적인 경우 너비, 높이 모두 부모 요소의 크기에 맞게 늘어난다. [objectFit](https://nextjs.org/docs/pages/api-reference/components/image-legacy#objectfit) 속성과 함께 사용된다.

<br/>

## Sizes

breakpoint 기준 브라우저에서 다운로드 할 이미지 크기를 결정하는데 사용된다. 디폴트 값은 `100vw` 다. layout이 `responsive`, `fill` 일 경우 성능에 영향을 미치며, `intrinsic` 또는 `fixed` 를 사용하는 이미지의 경우 무시된다. sizes는 srcset을 가공하게끔 도와준다.

<br/>

## Quality

`1`, `100` 사이 정수다. 기본값은 `75`.

<br/>

## Priority

boolean 값이고, true 면 우선순위가 높기 때문에 `preload` 한다. 사용하면 lazy loading이 자동으로 비활성화 된다. LCP로 감지된 모든 이미지에 priority 속성을 넣어주어야 한다. 서로 다른 이미지가 서로 다른 뷰포트 크기에 대한 LCP 요소일 수 있으므로 우선 순위 이미지를 갖는 것이 적절하고, 스크롤 없이 볼 수 있는 부분위 표시되는 경우에만 사용해라.

<br/>

### Placeholder

`blur` 또는 `empty(default)`
