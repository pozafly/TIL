# Script postbuild 다른 명령어 사용해야 할 때

Next.js에 sitemap을 next.js의 라우팅 과정이 있으므로 자동으로 생성해주고 싶었다. 찾았던 라이브러리는 `next-sitemap`

[next-sitemap](https://next-sitemap.iamvishnusankar.com/docs/documentation/installation)은, build 시 build 된 파일을 기준으로 url에 따라 sitemap을 생성해준다.

따라서, build 이후 알아서 cli로 sitemap 파일을 생성해주는데, package.json에

```jsx
{
  "build": "next build",
  "postbuild": "next-sitemap"
}
```

이와 같이 입력하라고 되어있다.

하지만, 배포 환경이 다양한 경우, `build` 명령어 하나 만으로 해결할 수 없다. staging, test, dev 계가 있을 수 있기 때문이다.

그렇다면 `post${스크립트명}` 이렇게 해주기만 해도, 해당 명령어 이후에 스크립트를 실행시켜준다.

```jsx
{
  "prd.build": "next build",
  "postprd.build": "next-sitemap"
}
```

이렇게하면, [prd.build](http://prd.build) 명령어 이후, next-sitemap 명령어가 동작하게 된다.
