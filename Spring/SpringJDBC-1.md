> 💻 스프링 입문 - 코드로 배우는 스프링 부트, 웹 MVC, DB 접근 기술
# Spring 순수 JDBC & 통합테스트

[실제 강의 링크](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%9E%85%EB%AC%B8-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8)

[이전강의](https://velog.io/@kimdodo/%EC%8A%A4%ED%94%84%EB%A7%81-MVC-%EA%B8%B0%EC%B4%88) 는 메모리 리포지토리를 이용해서 MVC 웹 개발에서 배웠다.
이번 시간에서는 메모리 리포지토리를 데이터베이스 연동을 통해서 DB 정보를 유지하고 어떻게 커넥션 해서 사용하는지 알아보자

## 순수 JDBC
솔루션 회사에 잠시 재직할때는 파이썬 Flask 통한 API 서버 개발에서 JDBC로 db와 연동하기 했다.
하지만 순수 JDBC가 아니라 Python에서 제공하는 JDBC 템플릿을 이용한거 였다. 그럼 과연 20년전 JPA도 사용하지 않던 시절에는 어떻게 JDBC 를 사용하였는지 알아보자

순수 jdbc 환경 설정

build.gradle
```
implementation 'org.springframework.boot:spring-boot-starter-jdbc'
runtimeOnly 'com.h2database:h2'
```

resource/application.properties
```
spring.datasource.url=jdbc:h2:tcp://localhost/~/test
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.username=sa
```

두가지 환경 설정을 사전에 시작해야한다.
>Spring 2.4 이후 부터는 username도 반드시 추가해야함

초기 개발에서 인터페이스를 통한 리포지토리 구현 목록을 만들었고 이를 jdbc를 활용해서 변경하면 된다.
개발 소스는 현업에서 사용하지 않으며 어떤 문제점이 있어 현대의 jdbc 템플릿 또는 JPA를 사용하게 되었는지를 확인

문제점
>1. 코드 굉장히 복잡
>2. 반환하는 exception이 다양해서 try catch 필요

여기서 추가사항은 
연결을 DB에 바로 하는게 아니라 Datasourceutils을 통해서 해야한다는 것이다.
트랜잭션 걸리는 상황에서 **커넥션을 이전과 동일**하게 유지하기 위해서 이렇게 해야함
```java
private Connection getConnection() {
 return DataSourceUtils.getConnection(dataSource);
 }
 
private void close(Connection conn) throws SQLException {
 DataSourceUtils.releaseConnection(conn, dataSource);
 }
```

이 메모리가 아닌 데이터베이스를 통한 리포지토리로 변경 시 핵심은 다음과 같다.
조립하는 코드 이외의 나머지 서비스 코드, 컨트롤러 코드를 변경할 필요가 없다.
이것은 인터페이스를 통해서 객체 지향 언어의 **다형성** 덕분이다.

인터페이스를 기반으로 구현체만 변경하면 이외의 코드를 수정할 필요가 없다.

![](https://velog.velcdn.com/images/kimdodo/post/7495091b-75cc-4433-9252-03677eea46cb/image.png)

인터페이스 기반 구현체인 jdbc 멤버 리포지토리를 추가하면 위 그림과 같이 패키지 구조가 구성

![](https://velog.velcdn.com/images/kimdodo/post/5b01cb6f-217d-4dbd-9c84-67ca7491d411/image.png)
여기서 기존의 메모리 리포지토리에서 구현체를 jdbc로 변경만 하면 이외에는 수정 X

객체지향개발의 다형성의 중요함을 잘 보여준다.
이런 원리를
> 개방-폐쇠의 원칙(OCP) 라고 한다.
이는 확장에는 열려있고, 수정 또는 변경에는 닫혀있다고 한다.

그리고 스프링의 **DI**를 잘 사용하면 기존 코드를 수정하지 않고 설정만으로 **구현 클래스를 변경 가능**
```java
@Configuration
public class SpringConfig {
    private DataSource dataSource;

    @Autowired
    public SpringConfig(DataSource dataSource){
        this.dataSource = dataSource;
    }
    @Bean
    public MemberService memberService(){
        return new MemberService(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository(){
        return new JdbcMemberRepository(dataSource);
    }
}

```

## Spring 통합 테스트
기존의 서비스 테스트에 **@SpringBootTest**을 통해서 스프링 컨테이너를 활용한 테스트로 변경 

테스트를 수행하는 것은 기존과 동일하다

다만 foreach, beforeeach와 같은 사전, 사후 작업이 사라지고 회원가입과 같은 테스트 기능 반복이 안된다.
테스트의 중요 사항 중 하나인 **반복성 위배** 👿

반복 불가 원인은 DB를 사용하기 때문이다. 회원가입은 Insert 쿼리가 DB에 커밋되었기 때문!!
DB에 쿼리를 날려주고 커밋을 하지 않으면 DB에 적용 안됨 

즉 커밋을 안하거나 롤백을 하거나로 문제 해결 필요

>이런 문제를 해결하는게  **@Transactional**

Transactional을 통하면 인서트 쿼리를 하고 트랜젝션을 수행하고 테스트가 끝나면 롤백해준다

이렇게 필요한 부분을 설정하고 스프링 통합 테스트 진행을 하면 정상 처리 된다.

>👀 통합 테스트가 얼마나 중요한가?

영한님의 강의에서는 통합 테스트가 이루어지는 경우도 종종 있고 이런 테스트를 잘 짜는 것도 필요하다.
그러나 단위 테스트 단순하게 자바 문법만을 활용한 **단위 테스트**가 잘 짜여지는게 **좋은 테스트일 확률이 높다**

즉 통합 테스트도 필요하지만 단위 테스트를 잘 짜고 확인하며 개발 하는게 중요하다!

다음 강의에서는 jdbc 템플릿, JPA를 활용하면 어떻게 DB 접근 방식이 변화하는지 알아보겠다.

