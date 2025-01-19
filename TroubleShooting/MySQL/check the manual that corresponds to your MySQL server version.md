# Check the manual that corresponds to your MySQL server version

MySQL에서 계속 뱉어내는 Syntax 에러가 있었다. 이클립스 console 창에 뜨는 쿼리문을 보면,

 ```sql
 insert into member (id, name) 
 values (?, ?)
 ```

WARN 8080 --- [nio-1112-exec-1] o.h.engine.jdbc.spi.SqlExceptionHelper: SQL Error: 1064, SQLState: 42000

ERROR 8080 --- [nio-1112-exec-1] o.h.engine.jdbc.spi.SqlExceptionHelper: You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'member (id, name) values ('hst', '황선태')' at line 1

이렇게 계속해서 뜨는거다. 그래서 별짓을 다해봤지. 스키마를 새로 생성한다던지, 테이블을 지웠다 새로 한다던지. 구글링도 ㅈㄴ 해보고. 답을 찾았다.

우선,

```sql

```

이런식으로 스키마.member MySQL workbench에서 실제 쿼리문을 돌리니까 들어가는거다.

 그래서 VO객체에 @Entity(name="member") 를 @Entity(name="example.member") 라고 주고 돌려보니 Table not exist.

```sql
insert into example_member (id, name) 
values(?, ?)
```

이렇게 테이블 명에 언더바가 생긴걸 알 수 있다. 아마 JPA에서 자동으로.을 _로 바꿔서 매핑시켜주나보다. 그래서 생각에 JPA DDL 을 CREATE를 주면 테이블을 알아서 생성해서 넣어주겠지 생각해 application.properties 에서 spring.jpa.hibernate.ddl-auto=create 이렇게 주고 돌리니, example_member 라는 테이블이 생기긴했는데 실제적인 member 테이블에 들어가지 않는거다.

휴, 결국 찾아냈는데,

ERROR 8080 --- [nio-1112-exec-1] o.h.engine.jdbc.spi.SqlExceptionHelper: You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'member (id, name) values ('hst', '황선태')' at line 1

이 스프링부트 오류문을 자세히 보면, syntax 오류이긴 하지만, check the manual that corresponds to your MySQL server version이라 적혀있다. 'MySQL의 서버 버전' mysql 버전업 리스트에 보면 어떤 버전 이상에는 더이상 member 테이블을 생성할 수 없다고 안내가 있다고 한다. 즉 MySQL에서 이제 member 라는 이름을 예약어를 걸려고 하는 듯 하다. 따라서 member 라는 테이블을 생성할 수는 있지만, member 테이블에 실제적으로 데이터를 집어넣을 때는 스키마.member를 사용해야하고, jpa에서는 이를 모르기 때문에 MySQL 의 서버 오류(신텍스 오류)만 뱉어냈을 뿐이다.

https://okky.kr/article/640767

테이블 drop 시키고 이름 바꿔서(e_member) 하니 잘 된다.
