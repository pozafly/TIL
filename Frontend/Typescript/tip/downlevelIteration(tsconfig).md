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

문자열을 풀면 안되는 이유는 아래와 같다.

```jsx
'💩'.length === 2
```

이처럼, 몇몇 이모지는 length가 2로 잡힌다. 이모지는 유니코드로 표현될 수 있고, UTF-16을 사용한다면, 16진수로 되어 있다. 따라서 length가 무척 헷갈릴 수 있는 상황이기 때문에 문자열을 spread 혹은, for of 와 같은 iterator로 풀려고 하면 경로를 내뱉는 것이다.

for of 구문이나, 배열에 대한 spread opertator 사용시 downlevelIteration 플래그를 사용하라는 메시지가 출력 된다.

tsconfig의 target을 확인해보면 es6 이전 버전일 것이다. 이 문법들은 es6 부터 지원되기 때문이다.

`--downlevelIteration` 옵션은 반복에 대한 (iteration) 타입스크립트 코드를 비교적 오래된 (down level) 자바스크립트 런타임에서도 문제 없이 수행되도록 컴파일 하는 옵션이다.

------

만약 위에서 언급한 문법들은 단순히 인덱스만을 가지고 반복하는 for문 코드로 변환한다면, 그들의 명세와는 다르게 변환되는 드문 예외 케이스가 존재한다.

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

