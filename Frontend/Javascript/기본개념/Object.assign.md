# Object.assign

> 출처: https://pro-self-studier.tistory.com/21

```javascript
Object.assign(target, ...sources)
```

- target: 타겟 오브젝트
- sources: 소스 오브젝트

소스 오브젝트는 타겟 오브젝트에 병합된다. 리턴 값으로 `타겟 오브젝트를 반환`한다.

```javascript
var obj = { a: 1 };
var copy = Object.assign({}, obj);
console.log(copy);   // { a: 1 }
```

카피 되었다. obj의 객체를 복사했고, 타겟 오브젝트인 {} 에다가 복사하고 return 값으로 {} 이곳에 복사된 obj를 return 한 것. 결론적으로는 {} 이녀석이 리턴된 것이다. 빈 객체 안에 병합 된 것.

```javascript
var obj1 = { a: 1 };
var obj2 = { b: 2 };
var obj3 = { c: 3 };
var newObj = Object.assign({}, obj1, obj2, obj3);
console.log(newObj);   // { a: 1, b: 2, c: 3 }
console.log(obj1);     // { a: 1 }
```

위의 예제와 마찬가지로 target 오브젝트를 {} 빈 배열을 주고 병합했다. 즉, obj1, obj2, obj3 가 {} 에 병합되어 리턴되어진 것. 하지만 obj1을 찍어보면 그대로 값이 나오는 걸 볼 수 있다. 다음은 {} 빈 배열이 아니라 target 오브젝트에 obj1을 줘보자.

```javascript
var obj1 = { a: 1 };
var obj2 = { b: 2 };
var obj3 = { c: 3 };
var newObj = Object.assign(obj1, obj2, obj3);
console.log(newObj);   // { a: 1, b: 2, c: 3 }
console.log(obj1);     // { a: 1, b: 2, c: 3 }
```

이번엔 target 오브젝트에 obj1을 넣고 병합을 시도했다. newObj처럼 obj1도 모두 병합된 값이 나왔다. 따라서 Object.assign의 메서드의 결과가 새로운 객체를 반환해야 한다면, 타겟으로 빈 객체를 주는 걸 잊지 말아야 함.
