## 문제 설명
두 문자열 `s`와 `skip`, 그리고 자연수 `index`가 주어질 때, 다음 규칙에 따라 문자열을 만들려 합니다. 암호의 규칙은 다음과 같습니다.

-   문자열 `s`의 각 알파벳을 `index`만큼 뒤의 알파벳으로 바꿔줍니다.
-   `index`만큼의 뒤의 알파벳이 `z`를 넘어갈 경우 다시 `a`로 돌아갑니다.
-   `skip`에 있는 알파벳은 제외하고 건너뜁니다.

예를 들어 `s` = "aukks", `skip` = "wbqd", `index` = 5일 때, a에서 5만큼 뒤에 있는 알파벳은 f지만 `[b, c, d, e, f]`에서 'b'와 'd'는 `skip`에 포함되므로 세지 않습니다. 따라서 'b', 'd'를 제외하고 'a'에서 5만큼 뒤에 있는 알파벳은 `[c, e, f, g, h]` 순서에 의해 'h'가 됩니다. 나머지 "ukks" 또한 위 규칙대로 바꾸면 "appy"가 되며 결과는 "happy"가 됩니다.

두 문자열 `s`와 `skip`, 그리고 자연수 `index`가 매개변수로 주어질 때 위 규칙대로 `s`를 변환한 결과를 return하도록 solution 함수를 완성해주세요.

## 제한사항
-   5 ≤ `s`의 길이 ≤ 50
-   1 ≤ `skip`의 길이 ≤ 10
-   `s`와 `skip`은 알파벳 소문자로만 이루어져 있습니다.
    -   `skip`에 포함되는 알파벳은 `s`에 포함되지 않습니다.
-   1 ≤ `index` ≤ 20
---

| s       | skip   | index | result  |
| ------- | ------ | ----- | ------- |
| "aukks" | "wbqd" | 5     | "happy" |

---

<br/>

## 나의 답

1. s 문자열을 하나하나 돌면서 'index' 만큼 땡기며 검사한다.
2. for문 내부에 if 문을 사용해서 skip 단어가 있다면 Index를 유지하고 skipCount 1을 더해준다.
3. answer에 차곡차곡 쌓기

```js
function solution(s, skip, index) {
    var answer = '';
    let sArr = s.split('');
    let skipArr = skip.split('');
    let tempArr = [];
    
    // 문자열을 하나하나 돌면서 'index' 만큼 땡겨야 한다.
    for(let i = 0; i < sArr.length; i++) {
        let targetString = sArr[i];
        let numberCode = targetString.charCodeAt();
        let skipCount = 0;
        
        for (let k = 1; k < index + 1; k++) {
            for(let j = 0; j < skipArr.length; j++) {
                if (skipArr[j] === String.fromCharCode(numberCode)) {
                    // skipArr의 단어와 겹치면 skipCount++
                    skipCount++;
                }
            }
            numberCode += 1;
        }
        let resultNumberCode = numberCode + skipCount;
        // z이후에 다시 a로 돌아가야 하므로.
        if (resultNumberCode > 122) {
            resultNumberCode = resultNumberCode - 122 + 96;
        }
        
        tempArr.push(String.fromCharCode(resultNumberCode));
        // skipCount 초기화
        skipCount = 0;
    }
    answer = tempArr.join('');
    
    return answer;
}
```

- charCodeAt : 영어 문자를 number 코드화
- String.fromCharCode : number 코드를 문자화
를 사용했다. 

<br/>

## 다른 분의 답

```js
function solution(s, skip, index) {
    const alphabet = ["a", "b", "c", "d", "e", "f", "g", "h", "i", "j", 
                      "k", "l", "m", "n", "o", "p", "q", "r", "s", "t", 
                      "u", "v", "w", "x", "y", "z"].filter(c => !skip.includes(c));
    return s.split("").map(c => alphabet[(alphabet.indexOf(c) + index) % alphabet.length]).join("");
}
```

- charCodeAt, String.fromCharCode 같은거 안썼음.
- a  ~ z 까지 나열 후 처음부터 알파펫에서 skip안에 들지 않은 것을 제거하고 시작했다.
- `alphabet[(alphabet.indexOf(c) + index) % alphabet.length]` 이 코드는, z 코드 이후에 a로 다시 돌아가게 하는 코드다. 즉, `%` 를 사용해 나머지 값의 index를 가지면 다시 alphabet 배열을 처음부터 돌기 때문이다.









