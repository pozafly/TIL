# 댓글 서비스 Utterances(feat.react)

[Utterances](https://utteranc.es/) 여기서 설정해줄 수 있다. 하지만, 공식적으로는 `<script>` 태그에 Utterances를 다는데 react에서는 script 태그를 직접 달수 없다.

```jsx
import React, { createRef, useLayoutEffect } from 'react';

const src = 'https://utteranc.es/client.js';

export interface IUtterancesProps {
  repo: string;
}

const Utterances: React.FC<IUtterancesProps> = React.memo(({ repo }) => {
  const containerRef = createRef<HTMLDivElement>();
  // isDark, theme은 테마에 따라서 어떤걸 넣어줄지 정해준다.
  const isDark = window.matchMedia("(prefers-color-scheme: dark)").matches;
  const theme = isDark ? 'dark-blue' : 'github-light';

  useLayoutEffect(() => {
    const utterances = document.createElement('script');

    const attributes = {
      src,
      repo,
      'issue-term': 'pathname',
      label: 'comment',
      theme,
      crossOrigin: 'anonymous',
      async: 'true',
    };

    Object.entries(attributes).forEach(([key, value]) => {
      utterances.setAttribute(key, value);
    });

    containerRef.current.appendChild(utterances);
  }, [repo]);
  return <div ref={containerRef} />;
});

Utterances.displayName = 'Utterances';

export default Utterances;
```

그리고, 사용할 때는

```jsx
<Utterances repo="pozafly/blog-comments" />
```

이렇게 넣어서 사용해줄 수 있다. 나는 이걸 Gatsby 블로그에 달았음.