# Rollback

rebase시 잘못 합쳤을 때, 코드 전체를 돌리고 싶다면, `rollback` 하려면,

```shell
$ git reset --hard origin/master
```

이러면 완전 원복임. clone 받았을 때로 돌아가게 된다.
