# **Spring 의존관계 자동 주입**

이전 포스팅에서 Spring 컴포넌트 스캔을 사용해서 @Bean을 통한 수동등록이 아닌 자동으로 빈을 등록하는 것을 보았다.

실제 현업에서는 수동/자동 빈 등록 모두 사용하는 경우도 있으며 의존 관계 주입도 자동 주입, 수동 주입 두가지의 경우가 각각 혼합되어 사용된다. \[[2024.02.09 - \[Spring\] - Spring 컴포넌트 스캔\]](https://developer-dodo.tistory.com/entry/Spring-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8-%EC%8A%A4%EC%BA%94 "Spring 컴포넌트 스캔")


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
