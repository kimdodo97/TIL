# **Spring 의존관계 자동 주입**

이전 포스팅에서 Spring 컴포넌트 스캔을 사용해서 @Bean을 통한 수동등록이 아닌 자동으로 빈을 등록하는 것을 보았다.

실제 현업에서는 수동/자동 빈 등록 모두 사용하는 경우도 있으며 의존 관계 주입도 자동 주입, 수동 주입 두가지의 경우가 각각 혼합되어 사용된다. \[[2024.02.09 - \[Spring\] - Spring 컴포넌트 스캔\]](https://developer-dodo.tistory.com/entry/Spring-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8-%EC%8A%A4%EC%BA%94 "Spring 컴포넌트 스캔")

<script>(adsbygoogle = window.adsbygoogle || []).push({});</script>

## **1\. 의존관계 자동 주입 방법**

의존관계 자동 주입은 총 4가지 케이스를 가지고 있다.

#### **◼ 생성자 주입**

생성자에 @Autowired를 통해서 의존관계를 주입한다. 2가지의 특징을 가지고 있다.

✅ **특징**

\- 생성자 호출 시 딱 **1번만 호출**되는 것이 보장

\- **불변, 필수** 의존관계에서 사용

생성자에서 처음 DI를 할때 이외에는다른 메서드나 멤버 변수 자체에 접근 할 수 없게끔 되어 불변하거나 필수적으로 필요한 관계를 정의할 때 주로 사용한다.

셋팅된 값을 바꾸면 안되는 상황에서 이렇게 생성자를 주입하는 상황에서 사용한다. (필수 의존 관계)

private final 로 하면 일단 이러나 저라나 무조건 값이 있어야한다고 지정한 것

생성자를 통해서 주입하기 때문에 값을 null로 주는 경우는 문서로 정의한 경우 이외에는 없다.

> 생정자의 인자가 1개만 있으면 @Autowired 어노테이션 생략도 가능하다(스프링빈 인 것만!)

#### **◼ 수정자 주입**

수정자 주입, Setter 주입이라고 부른다. **setter**를 만들어서 @Autowired 어노테이션을 통해서 DI를 주입한다.

스프링 빈을 다 등록하고 @Autowired가 있는 의존관계를 모두 주입하는 것

스프링 빈 등록 ---> 스프링 의존관계 주입 이렇게 두가지 단계를 순차적으로 수행하는 것

> 🛑 생성자 주입의 경우에는 2단계가 아닌 빈 등록과 동시에 의존관계 주입  
> 이유는 자바 코드 기반이기 떄문에 생성자를 무조건 호출해야하고 null로 비워둘수 없기에 자동으로 등록

**선택이나 변경**이 가능한 의존관계에서 주로 사용한다.

required 인자값을 어노테이션에서 사용해서 선택 설정도 가능하다. 이러면 의존관계에 사용하는 컴포넌트 즉 빈이 비어있어도 그냥 에러가 안나게 함

**📖 자바 setter, getter 프러퍼티 규약**

```
class Data{
	private int age;
    public void setAge(int age){
    	this.age = age;
    }
    
    public int getAge(){
    	return age;
	}
}
```

위 예시처럼 값에 접근 및 변경 시 직접 변수 접근 보다 이를 권장

#### **◼ 필드 주입**

```
@Component
public class OrderServiceImpl implements OrderService{
    @Autowired private MemberRepository memberRepository;
    @Autowired private DiscountPolicy discountPolicy;

    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }

    @Override
    public Order createOrder(Long memberId, String itemName, int itemPrice) {
        Member member = memberRepository.findById(memberId);
        int discountAmount = discountPolicy.discount(member,itemPrice);

        return new Order(memberId,itemName,itemPrice,discountAmount);
    }

    //테스트 용
    public MemberRepository getMemberRepository(){
        return memberRepository;
    }
}
```

위 예시를 보면 변수 자체에 @Autowired를 통해서 의존관계를 직접 주입하는 경우가 있음

굉장히 쉽고 코드가 간결해지는 장점이 있다. 하지만 **테스트가 어려운** 문제 발생

순수 자바로 하는 단위 테스트를 수행하는 경우에서는 스프링 컨테이너가 사용되지 않기 때문에 NullPointException이 발생

테스트를 하기 위해서 굳이 setter를 만들어서 값을 넣어줘야하는데 이것 보다는 그냥 setter 또는 생성자 주입이 편함

테스트 코드에서 사용하기 위해 하는 경우가 있긴하지만 스프링에서는 이를 warning으로 권장하지 않음

#### **◼ 메서드 주입**

생성자, Setter가 아닌 새로운 메서드를 통해서 의존성 주입을 수행

Init() 메서드를 만들어서 사용하기도 함, @Autowired를 하는 것은 동일하다. 그렇지만 해당 메서드 주입은 굳이 사용하지 않음. Setter 나 생성자 주입을 통해서 충분히 커버가 가능하며 쓸 필요가 없는 것

## 옵션 처리
스프링 빈이 없어도 동작해야하는 상황이 있다
이런 경우에는 required = false로 설정하면 가능하다.
자동 주입할 대상이 없으면 수정자 메서드 자체가 호출이 안된다.
여기서 수정자 라고 하는것은 의존관계를 주입하는 메서드를 의미한다

@Nullable을 호출하면 호출은 되는데 null을 반환한다.
Optional을 사용하는 방법은 없으면  Optional.empty를 반환
```java
public class autowiredTest {
    @Test
    void AutowiredOption(){
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(TestBean.class);

    }

    static class TestBean{
        @Autowired(required = false)
        public void setNoBean1(Member noBean1){
            System.out.println("nobean1 = " + noBean1);
        }

        @Autowired
        public void setNoBean2(@Nullable  Member noBean2){
            System.out.println("nobean2 = " + noBean2);
        }

        @Autowired
        public void setNoBean3(Optional<Member> noBean3){
            System.out.println("nobean3 = " + noBean3);
        }
    }
}

```

## 생성자 주입을 선택하자
이전에는 메서드 주입 이나 수정자 주입을 많이 썼음 
하지만 최근에 DI를 지원하는 프레임워크는 대부분 생성자 주입을 수행
대부분의 의존 관계는 어플리케이션 종료 직전까지 대부분 변경 되면 안되는 상황이 대부분

프레임워크 없이 순수한 자바 코드 단위 테스트를 하는 경우에 좋은 경우
다음과 같이 수정자 의존관계인 경우

```java
@Autowired
public void setMemberRepository(MemberRepository memberRepository){
	this.memberRepository = memberRepository;
}

@Autowired
public void setDiscountPolicy(DiscountPolicy discountPolicy){
	this.discountPolicy = discountPolicy;
}
```
만약 테스트 코드를 만든다고 생각해보자
수정자를 이용해서 DI를 하기 때문에 생성자를 통해 인스턴스를 생성한 상태에서는 현재 의존 주입이 이루어지지 않은 상태이다.
테스트 코드를 작성하는 경우 개발자는 직관적으로 어떤 상태가 주입되어야 하는지 알 수 가 없다.
또한 생성자는 생성되어 테스트 실행은 되지만 실제로 DI가 이루어지지 않았기에 null point exception이 발생

하지만 생성자 주입을 사용하면 인스턴스를 생성하는 과정에서 new에서 자체적으로 컴파일 에러가 발생하기 때문에 바로 무엇이 필요한지 알 수 있다.

final 을 사용해서 생성자에서 실수로 값이 설정되지 않은 경우에도 컴파일 과정에서 에러가 발생하여 초기에 에러를 통해서 수정이 가능하다.

생정자 주입 방식을 통해서 여러 문제를 컴파일 에러를 통해서 인지가 가능하며 final 키워드를 사용하는 것은 오직 생성자에서만 가능한 방법이다.
또한 초기에 설정한 것 메모리리포지토리, 멤버 서비스, 할인 정책 등은 애플리케이션 실행에서 설정되는 것이며 이것이 변경되는 것은 애플리케이션이 종료되거나 업데이트 과정에서 이루어져야하는 것이다.

## 룸복과 최신 트렌드
생성자를 통해서 사용하는 것은 좋지만 인자값을 각 지정하는 과정이 필요하다
이를 조금 더 간결하게 표현할 수 있는 방법이 바로 롬복이다.
롬복 라이브러리를 gradle에 추가하고 새로 빌드 한 이후 annotation processor enable 상태로 변경하고 pulgin을 추가하면 된다.

롬복을 쓰는 이유는 @Setter, @Getter 등 어노테이션을 사용해서 set???, get???와 같은 메서드를 직접 구현하는 과정을 줄여준다.

setter,getter 이외에도 생성자를 만드는 과정도 어노테이션으로 수행이 가능하다.
@RuiredArgsConstructor를 통하면 final 키워드가 붙은 멤버 변수를 생성자를 통해서 생성이 가능하다.

```java
private final MemberRepository memberRepository;
private final DiscountPolicy discountPolicy;

@Autowired
public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
  this.memberRepository = memberRepository;
  this.discountPolicy = discountPolicy;
}
```
이 생성자 코드가 어노테이션 하나로 대체되는 것이다.
그럼 여기서 의존 주입은 어디서 하는지 의문일 수 도 있는데 @Autowired를 생성자가 1개이면 생성가능하다.
즉 이 특징을 살려서 코드를 간결하게 만들고 DI하는 변수가 추가되거나 생성자가 추가될 떄 final 키워드를 통한 선언만 하면 된다.

코드 상에서는 사라진 것으로 보이지만 디컴파일을 확인하면 실제 소스는 기존과 아주 동일하게 있는 상태이다.

-----------------------------------
### 빈 여러개 일때 해결법
같은 역할을 수행하는 두개의 구현체가 있다고 생각해보자
할인 정책에서 고정할인, 등급 할인 정책 두가지가 있을떄 두가지 모두 컴포넌트로 등록하면 DI 과정에서 동일한 역할 즉 동일한 DiscountPolicy라는 인터페이스를 가진 두개의 컴포넌트가 생기고 이는 빈 충돌이 일어나서 에러가 발생한다.

그럼 2개 이상의 조회 대상 빈이 있을때 어떻게 하는지 한번 알아보자

@Autowired 필드 명 매칭
@Qualifier 끼리 -> 빈 이름매칭
@Primary 사용

#### @Autowired 필드 명 매칭
기본적으로 타입 매칭을 수행한다. 이때 빈이 여러개있으면 필드 이름, 파라미터이름으로 빈 이름을 추가 매칭한다.
타입이 똑같으면 필드나 파라미터 이름이 동일한게 있는지 추가로 확인해서 빈 조회

#### @Qualifier
추가 구분자를 통해서 주입하는 방법
rateDiscountPolicy라는 할인 정책에 @Qualifier("mainDiscountPolicy")
fixedDiscountPolicy라는 할인 정책에 @Qualifier("fixedDiscountPolicy") 라고 각각 어노테이션을 추가한다.

```java
@Autowired
    public OrderServiceImpl(MemberRepository memberRepository, @Qualifier("mainDiscountPolicy") DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
```
주문 서비스 생성자에서 @Qualifier를 사용하면 해당 설정된 이름으로된 스프링빈을 추가로 찾는 것이다.
>Qualifier는 Qualifier를 찾는 용도로만 쓰자
Bean 직접 등록과정에서도 사용가능

#### @Primary
이는 우선순위를 지정해서 사용하는 방법이다.
@Primary 어노테이션이 있는 빈이 우선적으로 등록되는 것이다.
컴포넌트 어노테이션에 추가로 하나더 등록하면 된다.

### 장단점
@Qualifier 모든 생성자 코드에서 Qualifier를 써줘야한다. 하지만 @Primary는 비교적 편하다는 장점이 있다.

예를 들어서 자주쓰는 DB의 커넥션을 획득하는 빈, 서브 데이터베이스의 빈이 있을때 메인의 DB는 Primary로 하고 서브는 Qualifier를 지정해서 명시적으로 획득하는 것이 현업에서 많이 쓰는 방식이자 코드가 깔금해진다.
>Qualifier와 Primary가 둘다 있을때 Qualifier가 더 우선순위가 높다.
Spring은 자동으로 하는 것 보다 수동으로 뭔가 설정을 더 해야되는 상황에 항상 우선순위가 높은 편이다.


