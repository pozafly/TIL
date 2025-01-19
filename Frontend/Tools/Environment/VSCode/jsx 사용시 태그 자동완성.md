# Jsx 사용 시 태그 자동 완성

jsx 안에 태그 없이 div만 적고 탭키나 엔터키 누르면 태그를 자동으로 만들어줌.

vscode setting -> emmet 검색 -> Edit in settings.json 열어서 아래 내용 추가

```json
{ 
  ... 
  "emmet.includeLanguages": { 
    "javascript": "javascriptreact"
  }
}
```
