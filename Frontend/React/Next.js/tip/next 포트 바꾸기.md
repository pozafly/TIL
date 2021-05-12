# next 포트 바꾸기

package.json 파일에서,

```json
"scripts": {
  "dev": "next -p 3060"
},
```

이렇게 `scripts` 쪽에 실행 명령어에 원래는 next 하나만 적혀있었지만, -p [포트번호] 이걸 달아주면 포트를 바꿀 수 있다.