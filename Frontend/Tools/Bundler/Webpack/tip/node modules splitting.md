# Node modules splitting

> [출처](https://so-so.dev/tool/webpack/node-modules-splitting/)

web 어플리케이션을 만들 때 여러 기능을 빠르고 쉽게 지원하기 위해 libaray를 사용함. 이걸 어떻게 build 하는지에 따라 초기 페이지 로딩 속도가 좌우 된다.

webpack의 splitChunk 옵션을 수정해 node_modules를 여러 개의 bundle file로 나눌 수 있는 방법에 대해 알아보자.

<br/>

## cacheGroups

[cacheGroups](https://webpack.js.org/plugins/split-chunks-plugin/#splitchunkscachegroups)는 특정 조건에 따라 청크 파일을 생성한다는 규칙. cacheGroups object의 key 값이 각 '규칙의 이름'이 된다. 이 글에서 설명할 cacheGroups 규칙은 [Next.js의 webpack-config.ts](https://github.com/vercel/next.js/blob/ed0820f763e74d0071625030aed70b3b21184aef/packages/next/build/webpack-config.ts) 를 일부 수정한 내용.

<br/>

## Framework, lib, commons 3가지 규칙

cacheGroups의 default 설정은 모두 false로 '사용하지 않음'으로 명시해주고, 커스텀 한 규칙을 3가지 정의한다.

```javascript
cacheGroups: {
  default: false,
  vendors: false,
  framework: {},
  lib: {},
  commons: {}
}
```

> 여기서 이름이 꼭 위 예시와 같을 필요는 없다. [webpack splitChunk 문서](https://webpack.js.org/plugins/split-chunks-plugin/#splitchunkscachegroups)에서는 `splitChunks.cacheGroups.{cacheGroup}.priority` 형태로 안내하고 있다.

- framework: react, react-router-dom 등 프로젝트가 사용하는 core framework를 분리한 chunk
- lib: 기준 크기를 넘어가는 노드 모듈에 대한 별도 chunk
- commons: 그 외의 모듈 chunk

위 규칙을 담은 config는 다음과 같다.

```javascript
cacheGroups: {
  default: false,
  vendors: false,
  framework: {
    chunks: 'all',
    name: 'framework',
    test: /(?<!node_modules.*)[\\/]node_modules[\\/](react|react-dom|react-router-dom)[\\/]/,
    priority: 40,
    enforce: true,
  },
  lib: {
    test(module) {
      return (
        module.size() > 80000 &&
        /node_modules[/\\]/.test(module.identifier())
      );
    },
    name(module) {
      const hash = crypto.createHash('sha1');

      hash.update(module.libIdent({ context: __dirname }));

      return hash.digest('hex').substring(0, 8);
    },
    priority: 30,
    minChunks: 1,
    reuseExistingChunk: true,
  },
  commons: {
    name: 'commons',
    minChunks: 1, // entry points length
    priority: 20,
  }
}
```

위 규칙에 따라 나눠진 bundle이다. framework, commons 그리고 lib에서 정의한 [hash] 이름으로 분리된다.

![[assets/images/d615b65f09eff00a65f0b8598537cf0f_MD5.png]]

이제 각 규칙을 이루고 있는 속성들에 대해 알아보자.

### Priority

cacheGroups의 `test` 규칙에 따라 여러 cacheGroups에 속할 수 있다. 모듈이 두 가지 이상의 그룹에 속할 수 있을 때, priority를 보고 높은 규칙에 속하게 된다.

### Chunks: all

chunks에 줄 수 있는 옵션은 `initial`, `async`, `all`.

``` js
// a.js
import React from 'react'
import('lodash') // dynamic load

console.log('hello a')

export default a;
```

`a.js` 에서 lodash는 dynamic import 되었고, react는 즉시 import 되었다.

```javascript
// webpack.config.js
splitChunks: {
  cacheGroups: {
    defaultVendors: {
      test: /[\\/]node_modules[\\/]/,
      chunks: 'async'
    },
  }
}
```

위 설정은 defaultVendors라는 chunk 그룹을 생성하고 이 그룹에서는 node_modules 폴더 아래에 있는 파일 중 비동기로 import 된 파일들을 포함시키는 설정이다.

만약 chunks 옵션이 `all` 인 경우 import 방식에 상관 없이 node_modules 아래에 있는 파일은 모두 defaultVendors.js로 모듈이 나눠지게 된다.

웹팩 node_modules splitting의 전체 자체가 browser에서 리소스를 병렬적으로 로드할 수 있으니, 번들을 작게 쪼개 전체 로딩 타임을 감소시킨다는 컨셉이다.

### Enforce: true?
