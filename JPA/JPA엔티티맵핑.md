## JPA에서 객체와 엔티티를 어떻게 맵핑하나?

---

JPA는 DB의 테이블을 객체로 맵핑하는 ORM 기술의 JAVA 진영 인터페이스 이다.
JPA가 객체와 DB사이의 패러다임의 불일치를 해소하고 객체로 테이블을 관리할 수 있게 한다.
그럼 엔티티로 맵핑하는 방법이 어떤 것이 있는지 한번 알아보자

### 객체와 테이블의 맵핑

**_@Entity_** 어노테이션을 통해서 객체가 JPA가 관리하는 엔티티라고 명명한다.  
즉 해당 어노테이션을 통해서 해당 객체가 JPA 가 관리하는 엔티티다 라고 알려주는 것

✅ **체크 사항**

- **_기본 생성자가 필수_** (파라미터 없는 public or protected)
- final 클래스 , enum, interface, inner 클래스에서는 사용 불가
- 저장할 필드에 final 설정 X

👉 **example code**

```java
@Entity
@Getter
@RequiredArgsConstructor
public class PostEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "post_id")
    private Long id;

    private String title;

    @Lob
    private String content;

    private int viewCount;

    private int likeCount;
}
```

#### @Entity 속성

◼ name  
JPA 에서 사용하는 엔티티 이름  
Default 값은 클래스 이름을 그대로 사용

#### @Table 속성

만약 DBA가 정한 테이블 네이밍 컨벤션은 무조건 앞에 tb\_ 를 사용해야 한다.  
그럼 엔티티 이름과 테이블 이름의 불일치가 발생  
이때 @Table의 name 속성을 통한 이름 설정

> 데이터 베이스 스키마 자동 생성 기능 제공  
> JPA는 기본적으로 엔티티를 통해서 테이블을 관리하기 때문에 스키마 자동 생성 기능을 제공  
> 하지만 여기서 주의해야하는 부분이 존재한다  
> 여러 옵션들이 있지만 **운영** 장비에서는 절대 **_create, create-drop, update 옵션 사용 ❌_**
>
> 개발 초기 단계 : create / update  
> 테스트 서버 : update / validate  
> 스테이징 or 운영 : validate / none

#### @Column

해당 어노테이션을 컬럼의 이름 또는 길이 제한, 유니크 등 컬럼에 대한 속성을 설정  
여기서 unique 옵션 같은 경우에는 오직 DB에 테이블의 ALTER 문을 추가하는데 사용되고
이는 DDL 생성 기능이라고 한다.

이는 JPA의 실행 로직에는 영향을 주지 않는다.

> 나의 경우에는 보통 컬럼 명 특히 PK같은 경우 테이블\_id 로 컬럼명을 명명하지만 객체에서는 id로만 명명해서 컬럼 이름을 불일치 해소  
> 그리고 유니크 옵션을 명시하는 정도로만 사용한다.
