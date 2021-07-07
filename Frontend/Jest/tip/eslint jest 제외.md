# eslint jest 제외

jest 관련 함수를 eslint에서 에러를 보내줄 때가 있다. `.eslintrc.js` 파일에  아래와 같이 

env 부분에 `'jest/globals': true,` 이렇게 넣어주면 된다.

```js
module.exports = {
  env: {
    browser: true,
    es2021: true,
    'jest/globals': true,  // 요부분
  },
  parser: '@babel/eslint-parser',
  extends: ['airbnb-base', 'prettier'],
  parserOptions: {
    ecmaVersion: 2021,
    sourceType: 'module',
  },
  plugins: [],
  rules: {
    'no-console': 'off',
  },
};
```

