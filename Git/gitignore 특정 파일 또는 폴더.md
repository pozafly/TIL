# Gitignore 특정 파일 또는 폴더

> 출처: https://kcmschool.com/194

프로젝트에서 특정 파일이나 폴더 push를 하지 말아야 할 경우가 생긴다. 예를 들면 노출되면 안되는 설정 파일이라든지, secret key라든지.

`.git` 폴더가 있는 같은 선장에 `.gitignore` 파일을 생성해 특정 파일이나, 폴더를 적어주면 git에서 파일 자체를 관리하지 않게 해줄 수 있다.

<br/>

<br/>

## 작성법

```
# 파일 무시
test.txt

# 다음과 같은 확장자는 전체 무시
*.text
*.exe
*.zip

# 폴더 무시
test/
```

폴더와 같은 경우 무시하는 폴더 하위의 파일이나 폴더 또한 ignore(무시) 된다. 참고로 위의 `#` 은 주석처리 된다.

`.gitignore` 파일에 작성했다면 add > commit > push 까지 해야 ignore가 적용된다.

<br/>

## 주의사항

기존에 git으로부터 관리받고 있던(commit 된 것들) 파일이나 폴더를 다시 추가로.gitignore 파일에 작성하고 add, commit, push 해도 ignore 되지 않는다.

 이럴 때는 기존에 가지고 있던 cached 를 치워야 함.

```shell
# 파일이라면
$ git rm --cached test.txt

# 전체 파일이라면
$ git rm --cached *.txt

# 폴더라면
$ git rm --cached test/ -r
```

`git rm --cached` 명령어는 Staging Area 에서 파일을 제거하고 Working Directory 에서는 파일을 유지하는 명령어다. 명령어를 실행하고 반드시 다시 commit을 해주어야 파일이 제거된다.
