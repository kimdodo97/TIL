
# 스프링 JDBC 템플릿 & JPA란

> 💻 스프링 입문 - 코드로 배우는 스프링 부트, 웹 MVC, DB 접근 기술
[실제 강의 링크](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%9E%85%EB%AC%B8-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8)

## Spring JDBC 템플릿

순수 JDBC와 동일한 환경 설정은 기반으로 진행된다.
스프링 JDBC 템플릿을 이용한다.
MyBatis 같은 라이브버리는 JDBC API의 반복 코드를 제거한다.
그러나 SQL 쿼리는 직접 작성해야한다.

순수 jdbc를 사용할때 보다 코드 자체가 간결해지고 중복되는 부분들이 많이 제거 되었다.
```java
public class JdbcTemplateMemberRepository implements MemberRepository {
    private final JdbcTemplate jdbcTemplate;
    @Autowired
    public JdbcTemplateMemberRepository(DataSource dataSource) {
        jdbcTemplate = new JdbcTemplate(dataSource);
    }
    @Override
    public Member save(Member member) {
        SimpleJdbcInsert jdbcInsert = new SimpleJdbcInsert(jdbcTemplate);
        jdbcInsert.withTableName("member").usingGeneratedKeyColumns("id");
        Map<String, Object> parameters = new HashMap<>();
        parameters.put("name", member.getName());
        Number key = jdbcInsert.executeAndReturnKey(new
                MapSqlParameterSource(parameters));
        member.setId(key.longValue());
        return member;
    }
    @Override
    public Optional<Member> findById(Long id) {
        List<Member> result = jdbcTemplate.query("select * from member where id = ?", memberRowMapper(), id);
        return result.stream().findAny();
    }
    @Override
    public List<Member> findAll() {
        return jdbcTemplate.query("select * from member", memberRowMapper());
    }
    @Override
    public Optional<Member> findByName(String name) {
        List<Member> result = jdbcTemplate.query("select * from member where name = ?", memberRowMapper(), name);
        return result.stream().findAny();
    }
    private RowMapper<Member> memberRowMapper() {
        return (rs, rowNum) -> {
            Member member = new Member();
            member.setId(rs.getLong("id"));
            member.setName(rs.getString("name"));
            return member;
        };
    }
}

```

## JPA 맛보기
JDBC 템플릿 또는 MyBatis와 같은 방식을 사용하는 경우도 아직까진 많다.
그러나 국외 동향을 확인하면 JPA를 사용하는 방식이 절대적으로 많다.
또한 JPA 점유율도 점점 상승하는 추세이기 때문에 RDBMS를 사용한다면 JPA 사용은 불가피하다.

그럼 JPA 사용이 뭔지 간단하게 한번 알아보자

JPA를 사용하기 위해서 도메인의 멤버를 Entity로 설정해줘야한다.

```java
package hello.hellospring.domain;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity
public class Member {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id; // 시스템 저장 ID
    private String name;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

그리고 리포지토리를 생성한다.

```java
package hello.hellospring.repository;
import hello.hellospring.domain.Member;
import javax.persistence.EntityManager;
import java.util.List;
import java.util.Optional;
public class JpaMemberRepository implements MemberRepository {
    private final EntityManager em;
    public JpaMemberRepository(EntityManager em) {
        this.em = em;
    }
    public Member save(Member member) {
        em.persist(member);
        return member;
    }
    public Optional<Member> findById(Long id) {
        Member member = em.find(Member.class, id);
        return Optional.ofNullable(member);
    }
    public List<Member> findAll() {
        return em.createQuery("select m from Member m", Member.class)
                .getResultList();
    }
    public Optional<Member> findByName(String name) {
        List<Member> result = em.createQuery("select m from Member m where m.name = :name", Member.class)
                .setParameter("name", name)
                .getResultList();
        return result.stream().findAny();
    }
}
```

JPA에 뭔지 현 시점에서는 무엇인지 정확히 인지하지는 못한다.
ORM을 위한 어떤 것이라는 것을 알고 생산성을 비약적으로 상승시키는 도구 정도라고 알고있다.

코드만 보더라고 PK를 사용하는 것은 쿼리를 작성하지도 않았으며 이외의 기능도 쿼리 작성이 간단하다

중요한 점은 **서비스 계층에서 트랜젝션을 추가**해야하는 것

```java
import org.springframework.transaction.annotation.Transactional
@Transactional
public class MemberService {}
```

JPA를 활요한 모든 **데이터 변경은 트랙젝션 내부**에서 실행

JPA를 활용하는 것은 현업에서 이제 필수적이며 JPA 기술은 Spring 만큼 방대한 기술이다.
그렇기 때문에 타 강의 수강 이후 JPA 강의 또는 서적을 통해서 추가 공부를 해보려 한다.