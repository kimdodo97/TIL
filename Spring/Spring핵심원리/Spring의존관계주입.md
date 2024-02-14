# **Spring 의존관계 자동 주입**

이전 포스팅에서 Spring 컴포넌트 스캔을 사용해서 @Bean을 통한 수동등록이 아닌 자동으로 빈을 등록하는 것을 보았다.

실제 현업에서는 수동/자동 빈 등록 모두 사용하는 경우도 있으며 의존 관계 주입도 자동 주입, 수동 주입 두가지의 경우가 각각 혼합되어 사용된다. 
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

```java
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

```java
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

#### **✅ 옵션 처리**

@Autowired에서 해당 의존관계가 없는 경우에도 애플리케이션을 실행되야 하는 상황도 존재한다.

이럴때는 옵션 처리를 통해서 해결한다.

1\. required = false

2\. @Nullable

3\. Optional

3가지 방법이 존재한다.

**◾ required \= false**

해당 설정을 @Autowired에 추가하면 의존관계 구현체가 없어도 그냥 애플리케이션은 실행된다.

하지만 의존관계가 없기 때문에 그 @Autowired의 코드는 실행 안됨 (테스트 코드 참조)

**◾ @Nullable**

Nullable을 통해서 사용하면 호출은 되는데 null을 반환한다.

개인적인 생각으로 Nullable은 별로라고 생각한다. null 반환 자체를 지양하는것이 좋으면 NPE 이 생기기 딱 좋은 상태

**◾ Optional**

Nullable과 흡사한데 Optional.empty를 반환하는 방식이다.

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

## **2\. 생정자 주입 선택**

설명한 것 처럼 총 4가지 방법의 의존 주입 방법이 존재한다. 하지만 **생성자 주입**을 사용하는 것을 **권장**한다.

특히 수정자 주입은 이전에는 많이 사용한 의존 주입 방법이다. 하지만 최근에는 DI 지원 프레임워크는 생성자 주입을 권장

왜 생성자 주입이 좋은지는 테스트 과정에서 처음 드러난다.

만약 수정자 DI를 하는 코드일 때 생각해보자

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

테스트는 순수자바 코드를 통해서 수행한다.

> 영한님이 항상 언급하는 부분중 하나가 순수 자바를 통한 유닛테스트가 많은 그리고 이게 잘 수행되는 코드가 좋은 경우가 많다

인스턴스를 생성한 상태에서는 현재 **DI가 이루어지지 않는 상태**이다. 즉 객체 생성 상태에서는 MemberRepository, DiscountPolicy가 어떤 구현체를 쓰는지 모른다는 것

테스트 코드를 작성하는 A 개발자는 이 코드의 어떤 구현 객체를 쓰는지 모른다. 또한 어떤 구현객체의 인터페이스를 사용하는지 모른다. 즉 **DI하는 빈의 타입을 전혀 모른다는 것**이다.

이런 경우 직접 코드 단으로 넘어가서 다시 확인해야 하며 단순하게 인스턴스만 생성해서 쓰면 Null Point Exception이 발생

이제 생성자를 통한 DI를 수행한다고 생각해보자

인스턴스를 생성하는 과정에서 new를 통한 선언에서 이미 자체적으로 컴파일 에러가 발생한다.

**final을 사용**해서 생성자에서 실수로 값을 설정하지 않고 구현된 경우에도 컴파일 에러를 통해서 **초기에 에러가 발견**된다.

즉 생성자를 통해서 기존에 테스트 코드를 작성할때 문제, 구현체가 없는 채로 생성된 인스턴스로 인한 애플리케이션 실행 후 에러 등 문제를 해결 가능

> 대부분의 DI 주입을 통해서 불변하는 성질을 가진 메인 DB 커넥션, 서비스 로직 등은 애플리케이션 시작과 종료까지 변경되면 안되는 상황이다. 그렇기 때문에 생성자 주입을 권장|  

## **3\. LomBok**

생성자를 통해서 의존 주입을 하는 것도 좋다. 그리고 자바에서는 getter, setter를 쓰는 것이 일반적인데 이때 마다 구현 코드를 일일히 쓰는 것은 굉장히 비효율적

이걸 LomBok라이브러리를 통해서 해결 할 수 있다.

롬복의 여러 어노테이션을 통해서 이런 반복적인 작업을 줄일 수 있다.

대표적으로 @Getter, @Setter가 존재한다.

```java
@Getter@Setter
@ToString
public class HelloLombok {
    private String name;
    private int age;

    public static void main(String[] args) {
        HelloLombok helloLombok = new HelloLombok();
        helloLombok.setName("asdf");
        helloLombok.setAge(1);
        String name = helloLombok.getName();
        System.out.println(name);
        System.out.println(helloLombok.toString());
    }
}
```

이런식으로 어노테이션을 사용하면

getter, setter , toString을 구현하지 않아도 롬복에서 이 구현을 대신 해준다.

👉실행 결과

[##_Image|kage@b40xzU/btsEHHVuNQy/x6p274u9na6MJTPEXJvu81/img.png|CDM|1.3|{"originWidth":250,"originHeight":70,"style":"alignCenter"}_##]

생성자를 생성하는 경우에는 @RequiredArgsConstructor를 사용하면 **final 키워드**를 사용한 멤버 변수를 인자값으로 받는 **생성자를 자동**으로 만들어 준다. (사용자 소스에서는 보이지 않지만 디컴파일에서 확인이 가능하다.)

그럼 **의존주입**은 어떻게 하는걸까

@Autowired는 **생성자가 하나이면 자동으로 @Autowired**를 붙여주는 것이 디폴트이기 때문에 생성자가 하나인 경우에서는 이를 활용하면 코드가 획기적으로 줄어든다.

## **4\. Bean 중복 해결**

같은 역할을 수행하는 빈 구현체가 여러개일때 이게 모두 Component가 등록되어 있으면 어떻게 해야 할까

기본적으로 실행을 하면 구현체가 여러개라고 충돌이 발생하며 컴파일에러가 발생한다.

예시를 통해서 한번 알아보자

고객 할인 정책이 고정 할인, 등급 할인 정책이 2가지가 있을때 두가지가 모두 컴포넌트로 등록되어 있다.

3가지의 방법으로 해결이 가능하다.

-   @Autowired 필드 명 매칭
-   @Qualifier 사용
-   @Pirmary 사용

#### **◼ @Autowired 필드 명 매칭**

기본적으로 Autowired는 **타입 매칭을 수행**한다. 이때 여러개 동일 빈 타입이 있으면 **필드 이름, 파라미터 이름**으로 빈 이름을 추가 매칭한다.

즉 DiscountPolicy rateDiscountPolicy로 생성자 DI 주입을 할때 파라미터로 전달하면 DI할때 스프링빈에서 타입을 보고 rateDiscountPolicy가 있는지 확인하고 DI를 수행하는 것

> 개인적으로는 이렇게 이름을 수행하면 구현체에 의존하지는 않지만 이름으로 의존하는 경향으로 보여 DIP 위배 처럼 생각된다.

#### **◼ @Qualifier** 

Qualifier는 **추가 구분자**를 통해서 주입하는 방법이다.

각각 고정, 등급 할인 정책에서 @Qualifier 어노테이션에 이름을 등록하고 아래 예시 코드 처럼 파라미터에도 Qualifier 를 사용하면 그 이름의 빈을 추가로 검색해서 찾아서 DI를 수행

```java
@Autowired
public OrderServiceImpl(MemberRepository memberRepository, @Qualifier("mainDiscountPolicy") DiscountPolicy discountPolicy) {
    this.memberRepository = memberRepository;
    this.discountPolicy = discountPolicy;
}
```

#### **◼ @Pirmary**

우선순위를 지정해서 사용하는 방법이다.

@Pirmary 어노테이션이 있는 빈이 우선적으로 등록된다.

컴포넌트 어노테이션에 추가로 하나를 더 등록하면 된다.

#### **✅ 장단점**

@Qualifier은 모든 생성자 코드에서 @Qualifier를 써줘야한다. 이는 어떻게 보면 추가적인 작업

또한 롬복을 쓰기 힘들어 생성자 주입 코드를 따로 만들어야함

@Primary는 비교적 간단하게 사용이 가능하다.

일반적으로는 primary를 메인 컴포넌트에 사용하고 Qualifier는 필요한 경우 서브 컴포넌트에서 필요한 경우에만 명시적으로 사용하는 방식을 주로 사용한다.

> Qualifier와 Primary가 둘다 있을때 Qualifier가 더 우선순위가 높다. Spring은 자동으로 하는 것 보다 수동으로 뭔가 설정을 더 해야되는 상황에 항상 우선순위가 높은 편이다.

## **5\. 여러 빈 조회 상황**

지금까지는 동일 타입에서 대부분 1개의 빈만 등록되게 Primary를 통해서 우선순위가 높은 빈을 사용했다.

동일 타입의 빈을 유동적으로 사용해야 상황에서 그럼 어떤 식으로 개발을 해야할까

> 할인 서비스를 고객이 원하는 대로 선택해서 사용하는 서비스가 존재한다고 생각해보자  
> 그럼 고정 금액 할인 또는 퍼센트 할인을 사용해야한다.

**List나 Map 과 같은 자료구조에 여러개의 빈을 저장**해서 사용하는 방식을 쓴다

아래 코드를 참조해서 확인하자

```java
public class AllBeanTest {
    @Test
    void findAllBean(){
        ApplicationContext ac = new AnnotationConfigApplicationContext(AutoAppConfig.class, DiscountService.class);

        DiscountService discountService = ac.getBean(DiscountService.class);
        Member member = new Member(1L, "userA", Grade.VIP);
        int discountPrice = discountService.discount(member,10000,"fixedDiscountPolicy");
        Assertions.assertThat(discountService).isInstanceOf(DiscountService.class);
        Assertions.assertThat(discountPrice).isEqualTo(1000);

        int discountPrice2 = discountService.discount(member,20000,"rateDiscountPolicy");
        Assertions.assertThat(discountPrice2).isEqualTo(2000);
    }

    static class DiscountService{
        private final Map<String, DiscountPolicy> policyMap;
        private final List<DiscountPolicy> policyList;
        public DiscountService(Map<String, DiscountPolicy> policyMap, List<DiscountPolicy> policyList) {
            this.policyMap = policyMap;
            this.policyList = policyList;
            System.out.println("policyMap = " + policyMap);
            System.out.println("policyList = " + policyList);
        }

        public int discount(Member member, int price, String discountCode) {
            DiscountPolicy discountPolicy = policyMap.get(discountCode);
            return discountPolicy.discount(member,price);
        }
    }
}
```

위 소스 처럼 할인 서비스라는 어떤 비지니스 로직이 존재하고 이는 자신이 원하는 할인 정책을 선택해서 자신의 할인 금액을 확인할 수 있다.

스프링 컨테이너를 생성하고 등록된 빈을 조회한다. 이때 생성자 주입을 바로 일어나고 수정자 주입은 빈 조회 이후에 일어난다.

자 policyMap은 생성자에서 존재하고 이때 바로 의존주입이 일어난다. policyMap은  String과 DiscountPolicy 타입으로 이주어진 **Map 자료 구**조이다

이때 String은 빈 이름, DiscountPolicy는 해당 이름의 빈 객체를 의미한다.

> 어떻게 자료구조 형태인데도 의존주입이 일어나는지 확인해보니  **Spring**에서는 Map 자료구조이면 String은 빈 이름으로 **빈 타입에 맞게 알아서 의존주입을 수행**하는 형태였다.  
> 만약 @Component("bean1") 라고 되어 있으면 bean1 이라는 이름으로 키가 설정되었을 것이다.

테스트 코드에서는 멤버를 생성하고 discount 에 멤버, 구입 금액, 할인 정책 이름을 전달하면 서비스에서 Map에 할인 정책이름을 **key로 빈을 찾아서** 할인 금액을 계산하여 반환한다.

이런 방식을 통해서 여러개의 동일한 타입의 빈을 유동적으로 사용할 수 있는 것이다.

## **6.현업 사용 방식**

백엔드 실무를 경험하지 않은 상태에서 스프링에 대한 강의 또는 학습을 수행할때 자동 빈 등록, 수동 빈 등록이 어떤 상황에서 쓰이는지 어떤게 더 주류로 사용하는지를 모른다.

결론부터 정리하면 점점 자동을 선호하는 추세이다.

컴포넌트, 서비스, 컨트롤러, 리포지토리 등 여러 어노테이션에 자동으로 컴포넌트 스캔을 통한 빈 등록을 자동으로 수행한다. 기존의 수동 빈 등록은 설정 파일을 만들고 관리하며 빈을 하나하나 등록하는 과정 자체가 반복적인 작업을 수행하는 것이 이는 충분히 자동화 하여 사용가능한 것이다.

> 자동 빈 등록을 사용해도 OCP,DIP 위배 X

그럼에도 수동 빈을 등록해야하는 경우는 존재한다.

이를 위해서 애플리케이션을 알아보자.

애플리케이션은 **업무 로직, 기술 지원 로직**으로 나누어진다.

**◾ 업무 로직** : 컨트롤러, 핵심 비지니스 로직 서비스, 데이터 계층의 로직을 처리하는 리포지토리 등 비지니스 요구사항을 개발시 변경 또는 추가 

****◾**  기술 지원** : 공통관심사 AOP를 처리하거나 기술적인 문제에 사용, DB 연결, 공통 로그 처리 등 업무 지원을 위한 기술 

업무 로직은 갯수도 많고, 한번 개발시 컨트롤러, 서비스 등 유사한 패턴이 있으면 이런 경우에는 자동 등록이 좋다. 일반적으로 에러 발생시 원인을 파악하기가 쉬움

반면에 기술 지원 로직은 숫자가 적으면 애플리케이션의 전반적으로 영향을 준다. 기술 지원 로직은 적용 확인 등 파악이 어려운 문제가 있음

> 애플리케이션의 광범위하게 영향을 미치는 기술 지원 객체는 수동 빈으로 등록해서 설정 정보에서 바로 나타나게 하는 것이 유지보수에 용이

비지니스 로직에서 다형성을 활용하는 경우

5번에서 언급환 동일 타입 빈을 여러개 쓰는 경우를 한번 보자 

할인 정책이 소스를 타고 들어가지 않으면 직관적으로 무엇인지 알수가 없다.

하지만 설정 정보를 통해서 빈 등록을 수동으로 하고 관리한다면 한번에 알수 있다.

즉 여러개의 빈을 유동적으로 사용할때는 그 타입빈을 수동으로 등록하는 타입 config로 관리하는 것이 유리하다.

이렇게 하면 DiscountPolicy를 보고 설정 관리가 되어 있는 파일을 통해서 확인이 가능하다.

이는 권장 사항일뿐 자동 등록으로 할거면 최소한 한 패키지에는 묶여있는 것이 좋다.

> 스프링과 스프링 부타가 자동으로 등록하는 빈들은 예외  
> DataSource라는 DB 연결에 사용하는 기술 지원 로직은 자동으로 등록하는데 이걸 강제로 수동 빈으로 쓸 필요는 없다.  
> 다만 개인적으로 구현한