# Operator(연산자)

> 출처 : https://youtu.be/YBjufjBaxHo

<br/>

## Logical operator(논리 연산자)

- || (or)

  - 조건 중 하나라도 true 라면 return true.
  - 단, 첫번 째 값이 true라면 뒤는 계산하지 않고 바로 true를 뱉음.

  ```javascript
  consoel.log(`or: ${value1 || value2 || check()}`);
  
  function check() {
    for (let i = 0; i < 10; i++) {
      console.log('😂');
    }
    return true;
  }
  ```

  - 여기서 보면 😂 이녀석이 출력이 안된다.
  - 따라서 연산이 많은 작업을 하는 녀석은 뒤에 두어야 함.

- && (and)

  - 조건 중 하나라도 false 라면 false 를 return.
  - 이 녀석도 마찬가지로 연산이 많은 작업을 하는 녀석을 뒤에 두는게 좋다.
  - null 체크 할 때도 사용됨. object가 null 일 때 false를 return 하기 때문.

- ! (not)

  - 값을 반대로 바꿔줌.

>`정리`
>
>|| && 연산자에서 연산 단위가 높은 녀석은 **뒤로 보내자**.  앞에서 true, false로 걸러지게 되면 뒤까지 계산하지 않기 때문이다. 메모리를 아낄 수 있는 꿀팁!