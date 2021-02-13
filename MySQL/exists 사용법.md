# Exists 사용법

```sql
EXISTS(서브쿼리)
```

서브쿼리의 결과가 한 건이라도 존재하면 **True**, 없으면 **False**를 리턴한다.

주로 안에 들어가는 서브쿼리는 select `1` ... 형식으로 작성해서 데이터의 존재여부를 확인한다.



```sql
select
    a.id,
    a.title,
    a.bg_color,
    a.public_yn,
    a.hashtag,
    a.like_count,
    a.created_at,
    a.created_by,
    EXISTS
      (
         select 1
           from board_has_like
          where board_id = a.id and user_id = #{userId}
      ) as own_like
from board a
where a.created_by = #{userId}
order by created_at desc
```

이런 식으로 사용했다.

