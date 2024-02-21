# **RDB 연관관계**

김영한님의 JPA 강의를 듣던 중 RDB를 설계하는 과정에서 가지는 일대다, 다대일,일대일 등 여러 연관관계에 대한 명확한 이해가 필요했고 이 글을 통해서 다시 한번 정리하고 실무에 필요한 정보도 추가로 정리하려고 한다.

<script>(adsbygoogle = window.adsbygoogle || []).push({});</script>

## **1\. RDB 설계**

#### ◼ **스키마(Schema)**

스키마란 DB를 구성하는 레코드의 크기, Key, 레코드간 관계, 검색 방법 등을 정의한 것을 의미

즉 데이터베이스에서 **데이터가 구성되는 방식과 엔티티간의 관계에 대한 설명**을 말한다.

#### ◼ **엔티티(Entitiy)**

실제, 객체라는 의미를 가지고 실무에서 엔티티라고 명명된다.

업무 즉 개발(비지니스 로직) 등에 **필요한 정보를 저장하고 관리하기 위한 집합적일 요소**들이라고 생각하면 된다.

> 학생이라는 엔티티가 있으면 여기에는 학번, 이름, 나이, 입학일 등 여러 속성이 존재한다.

#### **◼ 관계형 DB 구성 요소**

구조화된 데이터는 하나의 테이블로 표현 가능

사전에 정의된 테이블을 Relation이라고 보르기 때문에 관계형 DB라고 부른다.

| **키워드** | **설명** |
| --- | --- |
| 데이터 | 각 항목에 저장되는 값 |
| 테이블(Table/Relation) | 사전에 정의된 타입으로 작성된 데이터가 row로 저장 |
| 컬럼(Column/Field) | 테이블의 한 열을 가리킴(속성별 데이터) |
| 레코드(Record/Tuple) | 데이터의 한 행에 저장된 데이터 |
| 키(Key) | 테이블의 각 레코드를 구분하는 값   레코드마다 고유한 값을 가짐 |

#### **◼ 관계 종류**

-   1:1 관계
-   1:N 관계 (일대다)
-   N:M 관계 (다대다)

**👉 1:1 관계**

**하나의 레코드가 다른 테이블의 레코드 한개와 연결된 관계**

아래 예시를 통해서 확인하면 Users의 phon\_id는 외래키로 Phonebook 테이블의 ID와 연결

Phonebook의 테이블은 ID와 phon\_number로 이루어져있고 각 전화번호는 오직 한명의 유저와 연결되어 있다. 

실무에서는 1:1 관계도 많이 사용하지 않는다.

사실 아래와 같은 경우라면 Phonebook의 Phon\_number가 users의 속성으로 들어가도 무방하기 때문이다

![](https://velog.velcdn.com/images/kimdodo/post/e626b36e-6028-4acd-9914-6667f9ea89ff/image.png)


**👉 1:N 관계**

**하나의 레코드가 서로 다른 여러개의 레코드와 연결**

아래 예시를 통해서 한번 확인해보자

Users 테이블의 ID를 Phonebook에서 외래키로 참조한 상태이다.

한명의 유저가 여러개의 전화번호를 가질 수 있지만 여러명의 유저가 하나의 전화번호를 가질 순 없다.

즉 이런 경우가 일대다의 관계를 가진다.

일반적으로 실무에서 가장 많이 사용하는 관계

![](https://velog.velcdn.com/images/kimdodo/post/21b60eed-2414-4e6a-873b-63d7b67b94d1/image.png)


**👉 N:M 관계**

여러개의 레코드가 다른 테이블의 **여러개의 레코드와 관계**를 가지는 경우

다대다의 관계를 가지며 스키마 디자인을 할때는 J**oin 테이블을 만들어 관리**

예시를 통해서 알아보자

여러개의 여행 상품과 여러개의 고객이 존재한다.

고객 한명이 여러개의 상품을 구매할 수도 있고 여행 상품하나는 여러개의 고객이 구매도 가능하다.

즉 양쪽 모두 1:N의 관계를 가지기 때문에 다대다의 관계이다.

![](https://velog.velcdn.com/images/kimdodo/post/c51875fd-c45b-4f2a-a478-28b7cf2cc507/image.png)

**다대다의 관계에서는 조인 테이블을 가진다.**

아래 처럼 cutomer\_pakage라는 테이블을 만들고 이는 고객 한명이 여러개의 여행 상품을 가질 수 있고 여행 상품 하나가 여러개의 고객을 가질 수 있게 해준다.

조인 테이블 즉 다대다 관계를 직접 형성하면 안되는 이유는 따로 설명하겠다.
![](https://velog.velcdn.com/images/kimdodo/post/7324017e-fbff-44f1-8147-ea2b6c6ac3cd/image.png)


## **2\. 다대다 관계**

RDB를 설계할때 이론적으로는 다대다의 관계도 존재한다.

하지만 실무에서 다대다의 관계를 일반적으로 사용하지 않는다.

> 이 글을 포스팅하게된 핵심 이유중 하나이다.

관계형 DB는 정규화된 테이블 2개로 다대다 관계를 표현할 수가 없다.

연결 테이블 앞서 설명한 조인 테이블을 추가해서 이를 일대다, 다대일의 관계로 표현해야한다.

즉 아래 그림처럼 멤버라는 테이블과 프로덕트이라는 테이블은 여러개의 다대다의 관계를 가지지만 이를 풀어낼 방법이 없다. 그래서 중간 연결 테이블을 사용한다.

![](https://velog.velcdn.com/images/kimdodo/post/bde4a5e2-d2a3-483c-b6fd-7a6379d21256/image.png)


객체는 컬렉션을 사용해서 객체 2개로 다대다 관계 가능

ORM 즉 JPA를 사용하는 상황에서는 @ManyToMany 어노테이션을 사용하고 JoinTable로 연결 테이블을 지정가능

다대다 단방향

```
@Entity
public class Member {
    @ManyToMany    
    @JoinTable(name = "member_product")    
    private List<Product> products = new ArrayList<>();        
}
```

```
@Entity
public class Product{
    @Id@Generated(strategy=GenerationType.INDENTITIY)
    private Long id;
    
    private String name;
    
    @ManyToMany(mappedBy="products")
    private List<Member> members = ArrayList<>();
}
```

두개의 예제 샘플과 같이 각각 ManyToMany 어노테이션을 사용하고 각각 리스트를 만들어 연결 테이블의 역할을 수행해준다.

다대다 한계점

어려운 방법은 아니지만 **실무에서는 사용을 하지 않는다.**

복잡한 비지니스 로직을 개발하면, **연결 테이블이 단순한 연결만 수행하지 않는 경우**도 있다.

조인 테이블 자체에 주문 시간, 수량과 같은 데이터가 들어갈 수 있다.

이렇게 컬럼이 추가되면 ManyToMany를 사용할 수 없다. 왜냐면 **새로 추가될 컬럼 들은 엔티티에 맵핑이 안되기 때문이다.**

극복 방법

단순한 조인 테이블이 아닌 **조인 테이블을 엔티티로 바꾸면 된다**.

그렇게 변경하여 일대다, 다대일 어노테이션을 사용하면 되는 것이다.

즉 **@ManyToOne의 관계 2개**로 만드는 것이다.

JPA가 만들어서 숨겨서 사용하던 연결 테이블을 실제 테이블로 관리함으로 한계점을 극복하는 방법이다.

아래 예시 처럼 단순하게 멤버아이디, 프로덕트 아이디만 들고 있던 연결 테이블에서 오더 ID라는 PK를 사용해서 독립적인 Key를 가지며 Member와 오더는 일대다의 관계를 가지고, 오더와 프로덕트는 일대다의 관계를 가지게 된다.

![](https://velog.velcdn.com/images/kimdodo/post/56bcc2b7-7d17-4a50-8feb-2d75c205d9a3/image.png)


## **3\. 양방향 연관관계**

양방향 맵핑은 JOIN을 통해서 양방향으로 필요한 값을 가져오는 것처럼 객체 또한 양방향으로 통신 할 수 있는 구조

DB의 테이블은 방향이라는 개념이 없고 조인 연산을 통해서 워하는 값을 가져온다.

여기서 문제는 객체를 다룰때 발생한다.

**◾ 단방향**
![](https://velog.velcdn.com/images/kimdodo/post/246fcbff-8105-4131-90c8-94dc5edacfb0/image.png)


단방향 예시를 보개되면 멤버에서 Team을 참조 가능하지만 팀에서 멤버를 참조할 방법은 존재하지 않는다.

**◾ 양방향**
![](https://velog.velcdn.com/images/kimdodo/post/ba5cde34-7ba8-4e51-91fc-8f391165a7cf/image.png)


양방향 객체 연관관계에서는 팀에서도 멤버가 참조가 가능하다

이를 기반해서 예시 코드를 작성해보자

```
@Entity
public class Team {

    @Id @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;
    private String name;

    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>(); // ArrayList로 초기화 하면 add() 할 때 null관련 에러가 발생하지 않음.
}

@Entity
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;

    @Column(name = "USERNAME")
    private String name;

}
```

이렇게 엔티티를 만들면 각 팀에서도 멤버를 확인가능하고 멤버에서도 속한 팀을 확인 가능하다.

즉 양방향으로 자유롭게 객체 참조가 가능하다.

```
Member findMember = entityManager.find(Member.class, member.getId());

List<Member> members = findMember.getTeam().getMembers();
```

여기서 중요한 것은 바로 **연관관계의 주인과 mappedBy 인자값**이다.

mappedBy를 이해하기 위해 객체와 테이블이 연관관계를 맺는 차이를 이해해보자

**객체는 단방향 연관관계가 2개** 있다.

멤버 -> 팀

팀 -> 멤버

객체의 양방향 관계는 실제로 앙뱡향이 아닌 단방향 2개를 조합해 양방향 처럼 보이게 하는 것이다.

즉 객체를 양방향 참조하기 위해서 실제로는 단방향의 연관관계 2개를 만들어야한다.

하지만 테이블의 연관관계는 1개이다.

멤버 <-> 팀 1개의 연관관계이며 테이블은 외래 키 하나로 두 테이블의 연관관계를 관리한다.

외래키 조인을 통해서 관리가 가능하기 때문이다.

#### **◼ 양방향 맵핑 규칙**

-   객체의 두 관계 중 하나를 연관관계 주인으로 설정
-   주인만이 외래키를 관리 (등록, 수정)
-   주인이 아닌 쪽은 읽기만 가능
-   주인은 mappedBy쓰지 않음
-   주인이 아니면 mappedBy 속성으로 주인 지정

🔥 주인은 **외래 키가 있는 곳을 주인**이라고 한다.

자 그럼 예시로 확인해보자

지금까지 사용한 멤버와 팀의 경우에 외래키는 어디에 있는가

멤버의 TEAM\_ID가 외래키이다. 즉 이 연관관계의 주인은 MEMBER가 되는 것이다.

![](https://velog.velcdn.com/images/kimdodo/post/4de3df7b-3e7b-4358-bd02-fd41b2bc8c0d/image.png)


#### **❗ 실수 포인트**

연관관계의 주인은 멤버이다. 그런데 연관관계의 주인에 값을 입력하지 않는 실수를 흔히 범한다.

이게 무슨말인지 한번 보자

```
Member member = new Member();
member.setName("Member1");
entityManager.persist(member);

// 팀 저장
Team team = new Team();
team.setName("TeamA");

// 역방향(주인이 아닌 방향)만 연관관계 설정
team.getMembers().add(member);
entityManager.persist(team);
```

이렇게 코드를 작성하면 멤버는 Member1이라는 이름은 등록한다. 

팀에 새로운 팀을 만들고 TeamA라고 등록한다.

이렇게 하면 결과는 어떨까 멤버 Member1의 Team은 당연하게도 null 이다.

팀을 등록한 적이 없기 때문이다.

```
// 팀 저장
Team team = new Team();
team.setName("TeamA");
entityManager.persist(team);

// 회원 저장
Member member = new Member();
member.setName("Member1");
member.setTeam(team);
entityManager.persist(member);
```

이런식으로 팀을 먼저 생성하고 멤버 생성시 Team을 입력하면 정상적으로 값이 들어간다.

즉 양방향 맵핑시 연관관계의 주인에 값을 입력하는 걸 까먹지 말자

> 순수 자바 유닛테스트에서는 주인과 아닌 쪽 모두에 값을 입력해야한다.

많은 예시 코드를 보면서 이해하는 과정이 앞으로 필요할 것 같다.

현재 읽고 있는 AWS를 활용한 스프링 부트 프로젝트 관련 책, 영한님의 JPA 활용 편 강의를 통해서 연관관계를 어떤 식으로 사용하는지 알아보고 정리해보려 한다.
