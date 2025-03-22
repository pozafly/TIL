# application.yml 설정

```yaml
spring:
  output:
    ansi:
      enabled: ALWAYS
  datasource:
    url: jdbc:h2:tcp://localhost/~/Documents/dev/backend/jpashop
    username: sa
    password:
    # DB 연결을 위해 JDBC 드라이버의 클래스 이름을 지정함.
    driver-class-name: org.h2.Driver

  jpa:
    hibernate:
      # 자동으로 table을 만들어주는 모드
      ddl-auto: create
    properties:
      hibernate:
        # show_sql: true -> system.out으로 로그를 찍음
        format_sql: true

logging:
  level:
    # 하이버네이트가 생성하는 SQL을 디버그 모드로 콘솔에 출력해줌 (Logger를 통해 찍어줌)
    org.hibernate.SQL: debug
```

- ddl-auto: create 옵션을 주면 DB의 table을 매번 drop하고 새로 생성해준다.

```java
@Entity
@Getter @Setter
public class Member {
    @Id @GeneratedValue
    private Long id;
    private String username;
}

```

```java
@Repository  
public class MemberRepository {  
  
    @PersistenceContext  
    private EntityManager em;  
  
    public Long save(Member member) {  
        em.persist(member);  
        return member.getId();  
    }  
  
    public Member find(Long id) {  
        return em.find(Member.class, id);  
    } 
}
```

1. 레파지토리에서는 `@PersistenceContext` 가 엔티티 매니저의 역할을 하도록 SpringBoot에서 자동으로 코드를 생성해준다.
2. 왜 `return member.getId()` 를 하느냐? -> 커맨드(명령) 형이기 때문에 사이드 이펙트가 발생하고, 사이드 이펙트가 발생하면 전체 엔티티를 반환하기 보다는, id를 return해서 다시 조회가 가능하므로.

이런 Entity가 있고, 테스트를 아래와 같이 작성함.

```java
@ExtendWith(SpringExtension.class)
@SpringBootTest
class MemberRepositoryTest {

    @Autowired
    MemberRepository memberRepository;

    @Test
    @Transactional
    void testMember() throws Exception {
        // given
        Member member = new Member();
        member.setUsername("memberA");

        // when
        Long savedId = memberRepository.save(member);
        Member findMember = memberRepository.find(savedId);

        // then
        assertThat(findMember.getId()).isEqualTo(member.getId());
        assertThat(findMember.getUsername()).isEqualTo(member.getUsername());
    }
}
```

실행하면 로그는 아래와 같음.

```
drop table if exists member cascade 

drop sequence if exists member_seq

create sequence member_seq start with 1 increment by 50

create table member (
    id bigint not null,
    username varchar(255),
    primary key (id)
)
```

이런 식으로 처음 시작할 때 table이 존재하면 drop 해줌.

그런데, DB에 select 해보면 insert 했던 정보가 없다. `@Transactional` 어노테이션은 테스트 환경에서는 롤백을 자동으로 해주기 때문.

만약 test case에서 생성된 데이터를 직접 보고싶다면

```java {3}
@Test  
@Transactional  
@Rollback(false)  
void testMember() {
  // ...
}
```

이렇게 `@Rollback(false)` 를 붙여 돌리면 select 해볼 수 있다.

```java
Assertions.assertThat(findMember).isEqualTo(member);
```

이러는 어떻게 나올까? `toString`, `equals` 를 만들지 않았기 때문에 기본 `==` 비교임.
하지만 테스트가 잘 통과한다.

왜냐면 영속성 컨텍스트에서 꺼내왔기 때문임.

## 쿼리 파리미터 로그 남기기

```
select .... (?, ?)
TRACE 10847 --- [    Test worker] org.hibernate.orm.jdbc.bind              : binding parameter (1:VARCHAR) <- [memberA]
TRACE 10847 --- [    Test worker] org.hibernate.orm.jdbc.bind              : binding parameter (2:BIGINT) <- [1]
```

이런식으로 어떤 값이 들어갔는지 확인해야 할 때가 많음.

```yml {4}
logging:  
  level:  
    org.hibernate.SQL: debug  
    org.hibernate.orm.jdbc.bind: trace
```

이런식으로 넣어주면 표현이 된다.

### 외부 라이브러리 사용

https://github.com/gavlyukovskiy/spring-boot-data-source-decorator
스프링 부트를 사용하면 이 라이브러리만 추가하면 된다. (스프링 부트 3.0 이상을 사용하면 라이브러리 버전을 1.9.0 이상을 사용해야 한다.)

```
implementation 'com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.9.0'
```

참고: 쿼리 파라미터를 로그로 남기는 외부 라이브러리는 시스템 자원을 사용하므로, 개발 단계에서는 편하게 사용해도 된다. 하지만 운영시스템에 적용하려면 꼭 성능테스트를 하고 사용하는 것이 좋다.
