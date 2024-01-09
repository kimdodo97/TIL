# 관심사 분리(AppConfig)

> 💻 스프링 핵심 기술 - 기초
해당 강의는 김영한님의 스프링 핵심 원리- 기초 강의를 수강하여 공부한 내용은 간략하게 기록하는 기록장
[스프링 핵심 원리 기초 강의](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8#)

## 관심사 분리
어플리케이션이 하나의 공연이라고 보자
각 인터페이스는 배역이다. 근데 이 배역에 맞는 배우를 선택하는 건 누가 하는가?
인터페이스 즉 배역은 공연의 기획하는 PD가 설정하는 것이다.
이전에 DIP, OCP 위배가 되는 코드는 A 배역의 배우가 B 배역의 배우를 섭외하는 그런 기이한 상황인 것이다.

그럼 A 배우는 공연을 해야하면서 동시에 배역 할당이라는 **여러책임**을 가진다.

> 관심사를 분리해야 한다.

즉 배우는 연기만, 디렉터를 디렉팅만 해야한다.
즉 A배우는 B배우와 상관없이 A배역의 역할만 연기해야함
지금까지 없었던 **공연의 기획자**를 초빙해서 관심사를 분리해보자


## AppConfig
애플리케이션의 전체 동작 방식을 구성하기 위해서 **구현객체를 생성**하고, **연결하는** 책임을 가지는 별도의 클래스가 필요하다.
```java
public class AppConfig {
    public MemberService memberService(){
        return new MemberServiceImpl(new MemoryMemberRepository());
    }

    public OrderService orderService(){
        return new OrderServiceImpl(new MemoryMemberRepository(),new FixedDiscountPolicy());
    }
}
```
AppConfig는 동작에 필요한 **구현객체를 생성**
MemberServiceImpl
MemoryMemberRepository
OrderServiceImpl
FixDiscountPolicy
4개의 구현 객체를 모두 생성

객체의 인스턴스의 참조를 **생성자를 통해서 주입**

이렇게 되면 기존의 new 를 통해서 구현 객체 인스턴스를 생성하는 코드는 생성자를 통해서 의존 주입이 되고
이는 오직 인터페이스에만 의존하게 된다.

>MemberServiceImpl은 의존관계에 대한 고민은 없고 오직 실행만 수행

```java
private final MemberRepository memberRepository;

    public MemberServiceImpl(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
```

#### 클래스 다이어그램
![](https://velog.velcdn.com/images/kimdodo/post/ce57b865-160c-4bb5-b7d6-a7b5d5ff2b17/image.png)
이를 통해서 AppConfig가 객체 생성과 연결을 담당하고 구현 객체들은 인터페이스에만 의존하게 된다.
이를 통해서 관심사 분리를 수행한다.

#### 객체 인스턴스 다이어그램
![](https://velog.velcdn.com/images/kimdodo/post/79f76c39-fd5d-4aff-9f0f-d5033b4bc808/image.png)
AppConfig객체는 메모리멤버리포지토리를 생성하고 그 참조값을 전달
클라이언트인 memberService의 입장에서는 멤버리포지토리는 전혀 모르지만 외부에서 이 의존관계를 주입하는 형태
>**의존관계 주입 Dependency Injection(DI) **라고 한다


## Code
```java
AppConfig appConfig = new AppConfig();
MemberService memberService = appConfig.memberService();
```

멤버 서비스든 오더 서비스든 결국 이전처럼 new를 통해서 할당 받는게 아니라
AppConfig 객체를 생성하고 이를 통해서 뭔지 알수는 없지만 **AppConfig가 생성한 구현 객체**를 참조받아 사용하는 것이를
이를 통해서 **DIP, OCP의 위배**를 방지했다.

```java
//TEST
MemberService memberService;
OrderService orderService;

@BeforeEach
public void beforeEach(){
	AppConfig appConfig = new AppConfig();
	memberService = appConfig.memberService();
	orderService = appConfig.orderService();
}
```

테스트도 위 처럼 추상 클래스 선언만 하고 구현 객체는 **BeforeEach** 를 통해서 테스트 실행전에 AppConfig를 통해서 DI 받아서 사용한다.

현재 AppConfig를 통한 관심사 분리를 했지만 이를 어떻게 수정해서 더 좋은 코드로 바꿀지 다음 시간에 알아본다