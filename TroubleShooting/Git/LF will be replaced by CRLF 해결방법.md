# LF will be replaced by CRLF 해결방법

warning: LF will be replaced by CRLF 해결방법 

문서의 끝을 처리하는데 있어서 OS마다 약간의 차이가 있기 때문에 발생하는 에러임.

 

core.autocrlf 기능 꺼주기 

```shell
$ git config --global core.autocrlf false
```

이 명령어 하나면 됨.

