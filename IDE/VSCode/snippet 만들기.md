# snippet 만들기

> [출처](https://www.youtube.com/watch?v=umeqCopb96w)

- Cmd + Shift + P로 팔레트를 열어라.
- `snippet` 이라고 입력하면, Snippets: Configure User Snippet 을 선택해라.
- '새 전역 코드 파일'을 선택해라.
- 이름을 만들면 스니팻을 작성할 수 있는 파일이 생성된다.

```json
{
  "Console log": {
		"prefix": "cl",
		"body": "console.log($1)"
	}
}	
```

처음은 이름이고

- prefix: 어떤 단축키 쓸껀가?
- body: 어떤 문구 표시할건가?
  - `$1` 커서 위치 지정

[https://snippet-generator.app/](https://www.youtube.com/redirect?event=video_description&redir_token=QUFFLUhqbnRiV0VpRUQxVFNtS2VvR3NZR1ptZXZ2TDNKUXxBQ3Jtc0tsTWF4RHNQVk9vY1d4RmRFdmN0NnA1elJiRUs3Q0p1RjBZbHZQMU82OGlSb05LVzhnZm95dTR1UWtYanpvWTNlRzVRWHdyaXhaVUtKUW5SbVZyQVFoQWxFVW9yUFF3X1ZTT19ENFdCSUV5MThqMW9vbw&q=https%3A%2F%2Fsnippet-generator.app%2F&v=umeqCopb96w) 여기서 쉽게 만들 수 있다.

그리고, https://code.visualstudio.com/docs/editor/userdefinedsnippets 이곳은 VSCode 스니펫 문서임.

아래는 react 펑셔널 컴포넌트 자동 스니펫임.

```json
	"React Function Components": {
		"prefix": "rfc",
		"body": [
			"export default function ${TM_FILENAME_BASE/(.*)/${1:pascalcase}/}() {",
			"  ",
			"  return (",
			"    <>",
			"      <div>${1:${TM_FILENAME_BASE}}</div>",
			"    </>",
			"  )",
			"}"
		],
		"description": "React Function Components"
	}
```

