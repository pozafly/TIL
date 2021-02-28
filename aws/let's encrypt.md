# let's encrypt

-> 무료 인증서. 일단 갱신하는 부분만 적어본다. 나중에 다시 정리해보자



- 만료일 확인.

```shell
$ echo "" | openssl s_client -connect localhost:443 | openssl x509 -noout -dates
```

- 갱신 테스트(단 nginx 끄고 해야햄)

```shell
$ sudo certbot renew --dry-run
```

- nginx 상태확인

```sh
$ systemctl status nginx.service
```

- nginx 끄기

```sh
$ sudo service nginx stop
```

- nginx 켜기

```shell
$ sudo service nginx start
```

- certbot 갱신

```sh
$ sudo certbot renew
```