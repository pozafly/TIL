# downlevelIteration(tsconfig)

TypeScript에서 아래와 같은 오류가 발생했다.

`--downlevelIteration` 설정을 하라는 문구다.

<img width="879" alt="스크린샷 2023-06-30 오후 6 34 42" src="https://github.com/pozafly/TIL/assets/59427983/61837a1c-95aa-4a3c-9e28-5774c1b6be81">

코드를 자세히 보면 아래와 같다.

```jsx
const title = [...file.name].join('');
```

이렇게 spread 연산자로 string 형식을 풀고, 가공해주려고 했다.

📌 잘 보면, `file.name` 은 **문자열**이고, 문자열을 풀어 가공해주려고 했는데, 문자열을 풀 때 해당 오류가 TypeScript 측면에서 오류를 발생시킨 것이다.

for of 구문이나, 배열에 대한 spread opertator 사용시 downlevelIteration 플래그를 사용하라는 메시지가 출력 된다. tsconfig의 target을 확인해보면 es6 이전 버전일 것이다. 이 문법들은 es6 부터 지원되기 때문이다.

target이 ES5 이하일 경우, 문자열을 spread로 풀기를 권장되지 않는 이유는 아래와 같다.

```jsx
console.log('⛳'.length); // 1
console.log('💩'.length); // 2
```

이처럼, 몇몇 이모지는 length가 2로 잡힌다. 이모지는 유니코드로 표현될 수 있고, UTF-16을 사용한다면, 16진수로 되어 있다. 따라서 length가 무척 헷갈릴 수 있는 상황이다. ([이곳](https://blog.jonnew.com/posts/poo-dot-length-equals-two)에서 자세한 내용을 참고할 수 있다)

그러면, TypeScript는 폴리필을 통해 ES5와 같은 버전의 JavaScript 코드로 변환할 때, `Symbol.iterator` 구현이 없다면 index기반으로 변환하는데, 위의 length 문제가 있기 때문에 `--downlevelIteration` 옵션을 확인하면, 더 길어진 폴리필을 삽입하게 된다.

폴리필이 상당히 길어지게 되고, target 옵션이 ES6 이상일 수 있기 때문에 기본적으로 default 값은 `false` 로 지정되어 있다.

즉, `--downlevelIteration` 옵션은 반복에 대한 (iteration) 타입스크립트 코드를 폴리필을 이용해, 비교적 오래된 (down level) 자바스크립트 런타임에서도 문제 없이 수행되도록 컴파일 하는 옵션이다.

만약 위에서 언급한 문법들은 단순히 인덱스만을 가지고 반복하는 for문 코드로 변환한다면, 이모지와 같은 경우, ECMAScript 명세와는 다르게 변환되는 드문 예외 케이스가 존재한다.

## 결론

tsconfig.json 파일에서

```jsx
{
  "compilerOptions": {
    "target": "es6", // es5 -> es6
		// ...
	}
}
```

혹은,

```jsx
{
  "compilerOptions": {
    "downlevelIteration": "true",
		// ...
	}
}
```

아니라면, 문자열을 for of 또는 spread로 풀지 않고 가공하도록 하자.
