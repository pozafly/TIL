# p6spy 설치 및 세팅

## 1. Build.gradle의 dependencies에 implements

```properties
implementation("com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.11.0")
```

springboot를 위한 p6spy를 넣기 위함임.
p6spy 라이브러리가 따로 있긴 하지만, `p6spy-spring-boot-starter` 이건, application.yml에서 datasource를 따로 건들이지 않아도 설정이 가능하다.

## 2. application.yml 건드릴 필요 없음

```yml
spring:  
  datasource:  
    url: jdbc:h2:tcp://localhost/~/Documents/dev/backend/data-jpa  
    username: sa  
    password:  
    driver-class-name: org.h2.Driver  
  jpa:  
    hibernate:  
      ddl-auto: create  
    properties:  
      hibernate:  
        # show_sql: true  
        format_sql: true  
  
logging.level:  
  org.hibernate.SQL: debug  
  org.hibernate.type: trace
```

- org.hibernate.SQL: debug
- org.hibernate.type: trace
이 두개는

```sql
insert 
    into
        member
        (username, id) 
    values
        (?, default)
```

이런식으로 쿼리문이 찍히는데 보기 위함임. 없애고 `?` 에 값이 들어간 로그만 보고 싶다면 둘 다 `OFF` 로 하면 된다.

## 3. spy.properties

~~src/main/resources에 spy.properties 파일 생성~~

```properties
logMessageFormat=com.p6spy.engine.spy.appender.MultiLineFormat  
appender=com.p6spy.engine.spy.appender.Slf4JLogger  
multiline=true
```

properties 파일을 만들지 않고 application.yml에 아래와 같이 넣으면 된다.

```yml
decorator:  
  datasource:  
    p6spy:  
      enable-logging: true      # JDBC 로그 활성화  
      multiline: true           # MultiLineFormat 자동 적용  
      logging: slf4j            # Slf4JLogger 사용
```

이렇게만 넣어두어도 p6spy 동작 잘함. 하지만,

```
insert into member (username,id) values (?,default)
insert into member (username,id) values ('memberA',default);
```

이렇게 한 줄로만 포매팅 되지 않은 상태로 나오므로 config 파일을 직접 설정해주어야 함.

## 4. config, formatter class 생성

```java
import com.p6spy.engine.logging.Category;  
import com.p6spy.engine.spy.P6SpyOptions;  
import com.p6spy.engine.spy.appender.MessageFormattingStrategy;  
import jakarta.annotation.PostConstruct;  
import org.hibernate.engine.jdbc.internal.FormatStyle;  
  
import java.util.Locale;  
  
import static org.springframework.util.StringUtils.hasText;  
  
public class P6spyPrettySqlFormatter implements MessageFormattingStrategy {  
  
    private static boolean isDdl(String trimmedSQL) {  
        return trimmedSQL.startsWith("create") || trimmedSQL.startsWith("alter")  
                || trimmedSQL.startsWith("comment");  
    }  
  
    private static String trim(String sql) {  
        return sql.trim().toLowerCase(Locale.ROOT);  
    }  
  
    private static boolean isStatement(String category) {  
        return Category.STATEMENT.getName().equals(category);  
    }  
  
    @PostConstruct  
    public void setLogMessageFormat() {  
        P6SpyOptions.getActiveInstance().setLogMessageFormat(this.getClass().getName());  
    }  
  
    @Override  
    public String formatMessage(int connectionId, String now, long elapsed, String category,  
                                String prepared, String sql, String url) {  
        String formattedSql = formatSql(category, sql);  
        return formatLog(elapsed, category, formattedSql);  
    }  
  
    private String formatSql(String category, String sql) {  
        if (hasText(sql) && isStatement(category)) {  
            String trimmedSQL = trim(sql);  
            if (isDdl(trimmedSQL)) {  
                return FormatStyle.DDL.getFormatter().format(sql);  
            }  
            return FormatStyle.BASIC.getFormatter().format(sql); // maybe DML  
        }  
        return sql;  
    }  
  
    private String formatLog(long elapsed, String category, String formattedSql) {  
        return String.format("[%s] | %d ms | %s", category, elapsed,  
                formatSql(category, formattedSql));  
    }  
}
```

```java
import com.p6spy.engine.spy.P6SpyOptions;  
import jakarta.annotation.PostConstruct;  
import org.springframework.context.annotation.Configuration;  
  
@Configuration  
public class P6spyConfig {  
  
    @PostConstruct  
    public void setLogMessageFormat() {  
        P6SpyOptions.getActiveInstance()  
                .setLogMessageFormat(P6spyPrettySqlFormatter.class.getName());  
    }  
}
```

그리고 돌리면

```sql
insert      
    into
        member
        (username, id)      
    values
        ('memberA', default)
```

이렇게 이쁘게 잘 뜬다.

## 멀티모듈(MSA)일 경우 설정

```
├── common
│   ├── logging
│   │   ├── build.gradle
│   │   └── src
│   │       └── main
│   │           ├── generated
│   │           └── java
│   │               └── kuke
│   │                   └── board
│   │                       └── common
│   │                           └── p6spy
│   │                               ├── P6spyConfig.java
│   │                               └── P6spyPrettySqlFormatter.java
├── service
│   ├── article
│   │   ├── build.gradle
│   │   └── src
│   │       ├── main
│   │       │   ├── generated
│   │       │   ├── java
│   │       │   │   └── kuke
│   │       │   │       └── board
│   │       │   │           └── article
│   │       │   │               ├── ArticleApplication.java
│   │       │   │               ├── controller
│   │       │   │               │   └── ArticleController.java
│   │       │   │               ├── entity
│   │       │   │               │   └── Article.java
│   │       │   │               ├── repository
│   │       │   │               │   └── ArticleRepository.java
│   │       │   │               └── service
│   │       │   │                   ├── ArticleService.java
│   │       │   │                   ├── request
│   │       │   │                   │   ├── ArticleCreateRequest.java
│   │       │   │                   │   └── ArticleUpdateRequest.java
│   │       │   │                   └── response
│   │       │   │                       └── ArticleResponse.java
│   │       │   └── resources
│   │       │       └── application.yml
├── build.gradle
└── settings.gradle
```

이런 구조라 가정하자.
1. 먼저 root의 build.gradle에 디펜던시를 설치
2. common 경로 잡기
	- common/loggin/build.gradle에 JPA 디펜던시 추가 -> formatter에서 jpa 디펜던시 사용함
3. common에 P6spyConfig.java, formatter.java 를 넣는다.
4. settings.gradle
	- `include 'common:logging'` 추가
5. 사용처(service/article)의 build.gradle에 `implementation project(':common:logging')` 추가
6. 사용처(service/article)에서 application.yml에 p6spy 설정 코드 추가(decorator…)
7. 사용처(service/article)의 xxxApplication.java에 `@SpringBootApplication(scanBasePackages = "[패키지]")` 등록
	- P6spyConfig.java에는 `@Configuration` 어노테이션 사용중인데, 이게 각 패키지마다 컴포넌트 스캔이 안되므로 넣어줌.
	- 따라서 common이 포함되는 패키지 명을 넣어주어야 함.
