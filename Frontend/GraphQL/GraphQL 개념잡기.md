# GraphQL 개념잡기

> [출처](https://tech.kakao.com/2019/08/01/graphql-basic/)

페북에서 만든 쿼리 언어임. Structed Query Language(이하 SQL)과 마찬가지로 쿼리 언어. gql, sql의 언어적 구조 차이는 매우 크다. 실전에서 쓰이는 방식도 매우 크다.

- sql
  - **데이터베이스 시스템**에 저장된 데이터를 효율적으로 가져오는 것이 목적
  - sql의 문장(statement)은 주로 백엔드 시스템에서 작성, 호출
- gql
  - **웹 클라이언트**가 데이터를 서버로부터 효율적으로 가져오는 것이 목적
  - 클라이언트 시스템에서 작성하고 호출

```sql
SELECT plot_id, species_id, sex, weight, ROUND(weight / 1000.0, 2) FROM surveys;
```

```graphql
{
  hero {
    name
    friends {
      name
    }
  }
}
```

서버 사이드 gql 어플리케이션은 gql로 작성된 쿼리를 입력 받아 쿼리를 처리한 결과 다시 클라이언트로 돌려준다. HTTP API 자체가 특정 DB나 플랫폼에 종속적이지 않은 것처럼 gql도 어떤 특정 DB나 플랫폼에 종속적이지 않다. 심지어 네트워크 방식에도 종속적이지 않다. 일반적으로 gql의 인터페이스간 송수신은 네트워크 레이어 L7 HTTP POST 메서드와 웹 소켓 프로토콜을 활용한다.

필요에 따라 얼마든지 L4 TCP/UDP를 활용하거나 심지어 L2 형식의 이더넷 프레임을 활용할 수 있음.

![image](https://github.com/pozafly/TIL/assets/59427983/a83bbced-bbc7-479d-84ad-4f3cad99fa13)

파이프라인이다.

<br/>

## REST API와 비교

REST API는 URL, METHOD등을 조합하기 때문에 다양한 Endpoint가 존재한다. 반면, gql은 단 하나의 EndPoint가 존재함. gql API에서는 불러오는 데이터의 종류를 쿼리 조합을 통해 결정 한다. 예를 들면 REST API 에서는 각 EndPoint 마다 DB SQL 쿼리가 달라지는 반면, gql API는 gql 스키마의 타입마다 DB SQL 쿼리가 달라진다.

![image](https://github.com/pozafly/TIL/assets/59427983/0c849874-784a-43d4-84f8-57ca50c6f4ad)

![image](https://github.com/pozafly/TIL/assets/59427983/389d337b-9a80-4a15-9c53-c03e08d4c117)

위 그림처럼, gql API를 사용하면 여러 번 네트워크 호출을 할 필요 없이, 한 번의 네트워크 호출로 처리할 수 있다.

<br/>

## GraphQL 구조

### 쿼리/뮤테이션(query/mutation)

쿼리와 뮤테이션 그리고 응답 내용의 구조는 상당히 직관적이다. 요청하는 쿼리 문의 구조와 응답 내용의 구조는 거의 일치한다.

![image](https://github.com/pozafly/TIL/assets/59427983/83958702-491f-483c-87e2-b0becf829514)

gql에서 굳이 쿼리와 뮤테이션을 나누는데 내부적으로 들어가면 사실상 별 차이가 없음. 쿼리는 데이터를 읽는데(R) 사용하고, 뮤테이션은 데이터를 변조(CUD) 하는데 사용한다는 개념 적인 규약을 정해 둔 것임.

```graphql
{
  human(id: "1000") {
    name
    height
  }
}

query HeroNameAndFriends($episode: Episode) {
  hero(episode: $episode) {
    name
    friends {
      name
    }
  }
}
```

gql을 접했을 때 **일반 쿼리**와 **오퍼레이션 네임 쿼리**의 구분이 어려웠음. 하나는 앞에 query가 붙어있고, 다른 하나는 처음 시작이 중괄호 (`{`) 문자가 붙어 있다. 이것을 이해하는데 오퍼레이션 네임과 변수의 쓰임새를 살펴보는 것이 도움이 된다. 

일반적인 경우 클라이언트에서 static한 쿼리문을 작성하지는 않을 것이다. 주로 정보를 불러올 때 id 값이나 다른 **인자** 값을 가지고 데이터를 불러올 것이다. gql에는 쿼리에 변수라는 개념이 있는데, 이 개념은 이런 용처를 위해 존재하는 것이다. gql을 구현한 클라이언트에서는 이 변수에 프로그래밍으로 값을 할당할 수 있는 함수 인터페이가 존재한다. react apollo client 경우 variables 란는 파라미터에 원하는 값을 넣어주면 된다.

```graphql
query getStudentInfomation($studentId: ID){
  personalInfo(studentId: $studentId) {
    name
    address1
    address2
    major
  }
  classInfo(year: 2018, studentId: $studentId) {
    classCode
    className
    teacher {
      name
      major
    }
    classRoom {
      id
      maintainer {
        name
      }
    }
  }
  SATInfo(schoolCode: 0412, studentId: $studentId) {
    totalScore
    dueDate
  }
}
```

**오퍼레이션 네임 쿼리**는 매우 편라하다. 굳이 비유하자면 쿼리용 함수다. DB에서의 프로시저(procedure) 개념과 유사하다고 생각하면 된다. 이 개념 덕분에 REST API 호출할 때와 다르게, 한 번의 인터넷 네트워크 왕복으로 우리가 원하는 모든 데이터를 가져올 수 있다. DB의 프로시저는 DBA 혹은 백엔드 프로그래머가 작성하고 관리했지만, gql 오퍼레이션 네임 쿼리는 클라이언트 개발자가 작성하고 관리한다.

gql이 제공하는 추가 기능 덕분에 벡엔드 개발자와 프엔 개발자의 협업 방식에도 영향을 준다. 이전 협업 방식 (REST API) 에서는 프론트엔드 개발자가 백엔드에게 작성하여 전달하는 API의 request / response의 형식에 의존하게 된다. 그러나, gql을 사용한 방식에서는 이런 의존도가 많이 사라진다. 다만, 여전히 데이터 schema에 대한 협업 의존성은 존재한다.

<br/>

## 스키마 / 타입(schema / type)

DB 스키마를 작성할 때의 경험을 SQL 쿼리 작성으로 비유한다면 gql 스키마를 작성할 때의 경험은 C, C++의 헤더 파일 작성에 비유된다. 그러므로, 프로그래밍 언어(C, C++, JAVA 등)에 익숙한 개발자라면 스키마 정의 또한 쉽게 배울 수 있다.

### 오브젝트 타입과 필드

```ts
type Character {
  name: String!
  appearsIn: [Episode!]!
}
```

- 오브젝트 타입 : Character
- 필드 : name, appearsIn
- 스칼라 타입 : String, ID, Int 등
- 느낌표(!) : 필수 값을 의미 (non-nullable)
- 대괄호([, ]) : 배열을 의미(array)

<br/>

## 리졸버(resolver)

DB 사용 시, 데이터를 가져오기 위해 sql을 작성했다. 또한 DB에는 DB 어플리케이션을 사용해, 데이터를 가져오는 구체적인 과정이 구현되어 있다. 그러나 gql에서는 데이터를 가져오는 구체적인 과정은 직접 구현해야 한다. gql 쿼리문 파싱은 대부분의 gql 라이브러리에서 처리를 하지만, gql에서 데이터를 가져오는 구체적인 과정은 resolver(리졸버)가 담당하고, 이를 직접 구현해야 한다.

개발자는 리졸버를 직접 구현해야 한다는 부담은 있지만, 이를 통해 데이터 source의 종류에 상관 없이 구현이 가능하다. 예를 들어, 리졸버를 통해 데이터를 DB에서 가져올 수 있고, 일반 파일에서 가져올 수 있고, 심지어 http, SOAP와 같은 네트워크 프로토콜을 활용해 원격 데이터를 가져올 수 있다. 덧붙이면, 이런 특성을 이용하면 legacy 시스템을 gql 기반으로 바꾸는 데 활용할 수 있다.

gql 쿼리에선는 각각의 필드마다 함수가 하나씩 존재한다고 생각하면 된다. 이 함수는 다음 타입을 반환한다. 이런 각각의 함수를 리졸버라고 한다. 만약 필드가 스칼라 값(문자열이나 숫자와 같은 primitive 타입)인 경우에는 실행이 종료된다. 즉 더 이상의 연쇄적인 리졸버 호출이 일어나지 않는다. 하지만 필드의 타입이 스칼라 타입이 아닌 우리가 정의한 타입이라면 해당 타입의 리졸버를 호출되게 된다.

이런 연쇄적 리졸버 호출은 DFS(Depth First Search)로 구현되어 있을 것으로 추측한다. 이 점이 바로 gql이 Graph라는 단어를 쓴 이유가 아닐까 생각한다. 연쇄 리졸버 호출은 여러모로 장점이 있다. 연쇄 리졸버 특성을 잘 활용하면 DBMS의 관계에 대한 쿼리를 매우 쉽고, 효율적으로 처리할 수 있다. 예를 들어 gql의 query에서 어떤 타입의 필드 중 하나가 해당 타입과 1:n 관계를 맺고 있다고 가정해보자.

```graphql
type Query {
  users: [User]
  user(id: ID): User
  limits: [Limit]
  limit(UserId: ID): Limit
  paymentsByUser(userId: ID): [Payment]
}

type User {
	id: ID!
	name: String!
	sex: SEX!
	birthDay: String!
	phoneNumber: String!
}

type Limit {
	id: ID!
	UserId: ID
	max: Int!
	amount: Int
	user: User
}

type Payment {
	id: ID!
	limit: Limit!
	user: User!
	pg: PaymentGateway!
	productName: String!
	amount: Int!
	ref: String
	createdAt: String!
	updatedAt: String!
}
```

여기서는 User와 Limit는 1:1의 관계이고, User와 Payment는 1:n의 관계다.

```graphql
{
  paymentsByUser(userId: 10) {
    id
    amount
  }
}
```

```graphql
{
  paymentsByUser(userId: 10) {
    id
    amount
    user {
      name
      phoneNumber
    }
  }
}
```

위 두 쿼리는 동일한 쿼리 명을 가지고 있지만, 호출 되는 리졸버 함수의 갯수는 아래가 더 많다. 각각의 리졸버 함수에는 내부적으로 데이터베이스 쿼리가 존재한다. 이 말인즉, 쿼리에 맞게 필요한 만큼만 최적화하여 호출 할 수 있다는 의미다. 내부적으로 로직 설계를 어떻게 하느냐에 따라 달라질 수 있겠지만, 이러한 재귀형의 리졸버 체인을 잘 활용 한다면, 효율적 설계가 가능하다.(기존 REST API 시대에는 정해진 쿼리는 무조건 전부 호출이 되었다)

리졸버 함수는 다음과 같이 총 4개의 인자를 받는다.

```js
Query: {
  paymentsByUser: async (parent, { userId }, context, info) => {
      const limit = await Limit.findOne({ where: { UserId: userId } })
      const payments = await Payment.findAll({ where: { LimitId: limit.id } })
      return payments        
  },  
},
Payment: {
  limit: async (payment, args, context, info) => {
    return await Limit.findOne({ where: { id: payment.LimitId } })
  }
}
```

- 첫번째 인자는 parent로 연쇄적 리졸버 호출에서 부모 리졸버가 리턴한 객체. 이 객체를 활용해서 현재 리졸버가 내보낼 값을 조절 할 수 있다.
- 두번째 인자는 args로 쿼리에서 입력으로 넣은 인자다.
- 세번째 인자는 context로 모든 리졸버에게 전달 된다. 주로 미들웨어를 통해 입력된 값들이 들어 있다. 로그인 정보 혹은 권한과 같이 주요 컨텍스트 관련 정보를 가지고 있다.
- 네번째 인자는 info로 스키마 정보와 더불어 현재 쿼리의 특정 필드 정보를 가지고 있다. 잘 사용하지 않는 필드임.



........



---

## 정리

gql은 퍼포먼스적 장점이 분명 존재한다. 하지만 더 관심이 가능 장점은 바로 생산성 향상이다. gql은 기존 백앤드-프론트엔드 협업 문화를 많이 바꿀 것으로 예상한다. gql의 협업 구조상 프론트엔드 쪽에 조금 더 할일이 많아지고 힘이 실리는 느낌. 애자일 하게 웹사이트 프로젝트를 진행하는데 gql이 많은 도움이 될 것.

react를 혼합해 사용하려면 또 다시 가파른 고개를 넘어서야 제대로 사용할 수 있을 것임. react는 flux 아키텍쳐가 점령했다.  flux 아키텍쳐는 아름답지만,  flux와 함께  gql을 구현하는 일은 또 다른 숙제가 될 것. apollo client의 local statement management 기능은 flux 아키텍쳐를 구체화해 구현한 redux를 완전히 대처 가능하다고 생각됨.

