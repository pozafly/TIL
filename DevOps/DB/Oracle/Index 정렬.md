# Index 정렬

INDEX 정렬은 WHERE절에 PRIMARY KEY의 칼럼 > 0 조건을 주면, PRIMARY KEY에 INDEX가 있기 때문에 ORDER BY 절 보다 빠른 속도로 정렬시켜준다.

<br/>

[ex] TEMP에 속한 수습사원만 순번을 붙여 출력하시오. 단 5번까지만 출력하시오

```sql
SELECT ROWNUM, EMP_ID, EMP_NAME 
FROM TEMP 
WHERE LEV='수습'AND ROWNUM <= 5; 
```

```sql
SELECT ROWNUM, EMP_ID, EMP_NAME 
FROM TEMP 
WHERE EMP_ID > 0 AND LEV='수습'AND ROWNUM <= 5;
```

결과가 다름. 밑에 로직에 정렬이 들어갔기 때문.
