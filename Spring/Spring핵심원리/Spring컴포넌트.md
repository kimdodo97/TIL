# **Spring 컴포넌트**

스프링 빈을 등록할때 Bean 어노테이션을 통한 설정 정보 파일에서 직접 스프링빈을 등록하는 방식을 이용했다.

하지만 이렇게 빈이 100개 이상 수백 수천개인 대규모 시스템에서는 단순 작업을 반복 수행하는 것이고 이는 실수 발생, 시간 자원 소모의 문제로 이어질 수 있다.

Spring 에서는 이런 문제를 해결하고자 자동으로 스프링 빈을 등록하는 컴포넌트 스캔이라는 기능을 제공한다.

DI 주입을 자동으로 해주는 @Autowired 기능도 존재한다.

> Autowired 기능은 다음 강의에서 설명 예정이지만 간략하게 말하자  
> 스프링 빈을 자동으로 등록한다고 해서 @Bean 으로 수동 등록하며 DI 주입을 하는 것을 대신하는게 아니기 때문에 @Autowired 로 반드시 DI 가 필요

<script>(adsbygoogle = window.adsbygoogle || []).push({});</script>

```java
@Configuration
@ComponentScan(
        basePackages="hello.core.member",
        excludeFilters = @ComponentScan.Filter(type= FilterType.ANNOTATION, classes=Configuration.class)
)//이거 하는 이유는 기존의 APpConfig 무시하기 위함
public class AutoAppConfig {
}
```

이런식으로 기존의 @Configuration 어노테이션을 등록한 Config 파일에서 Componentscan이라는 어노테이션이 추가되는 것이다.

그리고 @Bean 수동 등록은 이제 필요가 없다.

```java
package hello.core.member;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class MemberServiceImpl implements MemberService{
    private final MemberRepository memberRepository;

    @Autowired
    public MemberServiceImpl(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }

    @Override
    public void join(Member member) {
        memberRepository.save(member);
    }

    @Override
    public Member findMember(Long memberId) {
        return memberRepository.findById(memberId);
    }

    //테스트 용도
    public MemberRepository getMemberRepository(){
        return memberRepository;
    }
}
```

이렇게 컴포넌트 어노테이션을 등록하고 의존성 주입이 필요한 생성자에서 @Autowired 어노테이션 추가

## **1\. 컴포넌트 스캔**

컴포넌트 스캔을 하면 프로젝트에서 어디까지 스캔을 해서 알아내는 것일까 라는 의문이 생긴다

basePackage 인자값을 통해서 패키지의 **시작 위치를 설정 가능**하다.

이렇게 되면 지정한 패키지의 위치에서만 탐색을 하는 것이다.

```
basePackages="hello.core.member",
```

이런식으로 한다면 hello.core.member 패키지의 컴포넌트만 탐색하는 것

그럼 basePackage의 default 값은 무엇일까

설정 정보 클래스가 위치한 그 클래스 패키지로 부터 시작하여 **하위 패키지들을 전부 스캔**하는 방식이다.

즉 설정 정보의 위치가 굉장히 중요한 것이다.

일반적으로 설정 정보는 그 프로젝트를 대표하는 설정 문서가 되는 것이고 처음 프로젝트에 참여하면 설정 파일을 통해서 구조를 파악하기 마련이다. 

그렇기 때문에 관례상으로도 디폴값의 특성으로 설정 파일을 프로젝트의 가장 루트 패키지에 일반적으로 위치하게 하고 basePackage는 디폴트 값을 통해서 사용한다.

> Spring 부트에서도 @SpringBootApplication에서 사실 컴포넌트 스캔이 있고 부트를 통해서 생성한 패키지의 최상단에 위치한다

#### **✅ 컴포넌트 스캔 기본 대상**

컴포넌트 스캔은 @Component 이외의 다른 어노테이션도 스캔하여 빈에 등록한다.

대표적으로는 5개가 있다.

| **어노테이션** | **설명** |
| --- | --- |
| @Component | 컴포넌트 스캔에 사용 |
| @Controller  | 스프링 MVC 컨트롤러에서 사용 |
| @Service | 스프링 비지니스 로직에서 사용 |
| @Repository | 스프링 데이터 접근 계층에서 사용 |
| @Configuration | 스프링 설정 정보에서 사용 |

어노테이션은 상속관계가 실제로 존재하지 않지만 스프링 프레임워크가 이를 지원해서 나머지 서비스도 컴포넌트 스캔을 통해서 빈에 등록이 되는 것이다.

컴포넌트 스캔의 용도 뿐만 아니라 어노테이션을 통해서 부가 기능을 수행하기도 한다. 

@Controller는 MVC 의 컨트롤러 역할을, @Repository는 스프링 데이터 접근 계층으로 인식하고 데이터 계층 예외를 스프링 예외로 변환 하는것, @Service 특별한 역할은 없지만 개발자가 서비스 로직임을 구분하게 해주는 것 등 의 역할이 존재한다.

> ◼ 필터 옵션 5가지  
> ANNOTATION은 기본값, 어노테이션을 인식  
> ASSIGNABLE\_TYPE 지정한 타입과 자식 타입을 인식  
> ASPECTJ AspectJ 패턴 이용  
> REGEX 정규 표현식  
> CUSTOM TypeFilter인터페이스를 구현해서 처리

## **2\. 컴포넌트 중복 등록**

자동 등록 빈 vs 자동 등록 빈

수동 등록 빈 vs 자동 등록 빈

이렇게 두가지의 케이스가 충돌 케이스 문제이다.

**◼ 자동 빈 / 자동 빈 충돌**

컴포넌트 스캔에 의해 자동으로 Spring Bean 이 등록되는데 동일 Bean 이름이 존재하면 예외가 발생한다.

ConfilctBeanDefinitionException이 발생하게 된다.

**◼ 수동 빈 / 자동 빈 충돌**

수동 빈과 자동빈이 동일한 이름으로 충돌하게 된다면 Spring 프레임워크는 어떻게 처리할까

답은 수동 빈 즉 사용자가 **수동으로 등록한 빈**의 **우선순위가 높다**고 판단하고 수동 빈으로 **오버라이딩** 시킨다.

그럼 자동 빈간의 이름 중복만 조심하면 되냐❔

아니다 오히려 수동 빈으로 덮어지는 경우가 더 위험하다.

에러가 발생하지 않고 어디서 문제가 발생하는지 찾아보려면 백로그를 열심히 뒤져야하는 상황이 발생하는 것이다.

Spring Boot 에서는 이런 문제가 크리티컬 하다고 판단하여 수동빈과 자동빈이 동일하면 덮어쓰지 않고 에러가 발생하게끔 변경하였다.

오버라이딩을 허용하려면 오버라이드를 설정에서 true로 바꿔줘야한다.

> 개인적인 여러 강의, 회사 재직 경험에 비롯했을때 실질적으로 스프링 부트를 쓰는 환경보다 아직도 스프링 프레임 워크만 사용하는 경우도 다수 존재할 것이다.  
> 그렇기 때문에 이 문제가 스프링 부트만 쓰면 단순하게 조심하면 되지만 실제 현업에서는 크리티컬한 문제로 이어 질 수 있다는 사실로 반드시 기억해야한다.❗❗

<script>(adsbygoogle = window.adsbygoogle || []).push({});</script>