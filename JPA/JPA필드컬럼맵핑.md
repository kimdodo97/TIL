## JPA에서 필드와 컬럼의 맵핑 방법은?

---

만약 우리는 아래과 같은 요구사항이 있다고 하고 필드와 컬럼을 JPA에서 어떤 식으로 맵핑하는지 알아보자

### 요구 사항

1. 회원은 일반, 관리자로 구분된다.
2. 회원 가입일과 수정일이 존재해야한다.
3. 회원을 설명할 수 있는 필드는 존재해야 한다. (해당 필드 길이 제한 X)

이렇게 3가지의 요구 조건을 만족하는 최소 사용자 엔티티를 한번 설계해보자

```java
@Entity
public class User{
    @Id
    private Long id;

    @Column(name="name")
    private String username;

    private Integer age;

    @Enumerated(EnumType.STRING)
    private Role roleType;

    @Temporal(TemporalType.TIMESTAMP)
    private Date craeteDate;

    @Temporal(TemporalType.TIMESTAMP)
    private Date lastModifiedDate

    @Lob
    private String desc;

    public User(){

    }
    ...
}
```

### 매핑 Annotation

hibernate.hbm2ddl.auto

1. @Column : 컬럼 맵핑
2. @Temporal : 날짜 맵핑
3. @Enumerated : enum 타입 매핑
4. @Lob : BLOB, CLOB 매핑
5. @Transient : 특정 필드 컬럼 매핑 제외

위 5가지가 필드 값과 컬럼을 매핑하는 대표적인 어노테이션이다.

### @Column 어노테이션 속성

- **name**  
  필드와 매핑할 테이블의 컬럼 이름  
  Default는 객체의 필드 이름

- **insertable, updateable**  
  등록, 변경 가능 여부 설정  
  기본값은 참

- **nullable**  
  null 값의 허용 여부를 설정한다
  false로 설정하면 DDL 생성 시에 not null 제약조건이 붙는다.

- **unique**  
  @Table 의 uniqueConstraints 와 동일한 효과를 주지만 해당 컬럼에만 적용 가능

- **columnDefinition**  
  데이터베이스 컬럼 정보를 직접 줄 수 있다.  
  해당 컬럼의 정의를 사용자가 직접 정의할 수 있다. SQL DDL 쿼리문을 직접 설정

- **length**  
  문자 길이 제약 조건, String 타입에만 사용 한다.

- **precision, scale**  
  BigDecimal 타입에서 사용  
  precision은 소수점을 포함하는 전체 자릿수, scale 은 소수의 자릿수  
  해당 2개의 어노테이션은 정밀 소수를 다룰떄 일반적으로

✅ **_주의 사항_**

> Enum 타입을 사용할 떄 STRING 과 ORDINARY 라는 2가지의 옵션이 존재한다.  
> **ORINARY는** 만약 Enum이 A,B,C 가 있으면 해당 결과가 **INDEX 번호인 0,1,2로 DB에 저장**된다.  
> 이렇게 되면 만약 새로운 Enum 타입이 추가 될때 개발자가 기존의 Enum 타입 컬럼이 어떤 값이 이었는지 알 수 없다.  
> 그렇기 때문에 **_반드시 STRING으로 설정해서 Enum 타입_**을 사용해야한다.

📌 추가 사항

> Java 8 버전 이후에는 LocalDate, LocalDateTime 2가지의 타입을 통해서 날짜, 날짜+시간 을 타입으로 명시한다.  
> JPA도 과거 버전과 다르게 Java 8 이후 개발된 버전에서는 자동으로 LocalDate와 LocalDateTime 을 지원하기 때문에 @Temporal 어노테이션 생략 가능
