# scss

vue에 scss를 입힐 때 필요한 모듈은, `node-sass`, `sass-loader` 다.

이녀석들은 devDependencies로 넣어주어야 함.

```shell
$ npm i -D node-sass sass-loader
```

하지만, 이렇게 넣어주고, `<style>` 태그에 lang="scss" 를 붙여주게 되고 컴파일하면,

`Syntax Error: TypeError: this.getOptions is not a function` 이라는 문구를 볼 수 있다. 버전 호환성 문제다.

따라서 버전을 명시해주어 깔아주는 것으로 해결 할 수 있음.

```shell
$ npm i -D node-sass@5 sass-loader@10
```

이렇게 하면 vue2와 호환이 잘 된다.