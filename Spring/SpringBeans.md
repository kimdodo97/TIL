김영한님의 강의 들으면 앞서 서비스 테스트를 작성하는 방법을 배웠다
이후 강의에 핵심 내용의 스프링빈과 의존관계에 대해서 배웠다

그렇다면 여기서 스프링 빈은 과연 무엇인가

## Spring Bean 이란?
정의로는 Spring 컨테이너가 관리하는 자바 객체를 **Bean**이라고 한다.
그럼 여기서 드는 의문은 스프링 컨테이너가 무엇인가 라는 의문이 생긴다

## Spring 컨테이너란?
**스프링 컨테이너**는 스프링 Bean의 생명 주기를 관리하며, 생성된 스프링 Bean들에게 추가적인 기능을 제공하는 역할을 한다.
IoC와 DI의 원리가 스프링 컨테이너에 적용

## DI
DI라는 것은 또 무엇인가
이것은 영한님의 강의 예제 코드를 통해서 알 수 있다.
앞서 멤버 서비스를 개발하였고 이를 회원 컨트롤러에 추가하는 과정에서 의존관계를 주입한다.

```java
package hello.hellospring.controller;
import hello.hellospring.service.MemberService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
@Controller
public class MemberController {
 	private final MemberService memberService;
 	
    @Autowired
    public MemberController(MemberService memberService) {
    	this.memberService = memberService;
    }
}

```

샘플 코드처럼 컨트롤러 생성자 @Autowired를 통해서 스프링이 자체적으로 연관된 객체를 스프링 컨테이너에서 찾아서 넣어준다.
객체 의존관계를 외부에서 주입하는 것은 **DI(Dependency Injection)** 의존성 주입이라고 한다.
이전 테스트에서는 개발자가 직접 주입하였고 현재는 Autowired에 의해서 스프링이 주입한다.
이전 코드는 다음과 같다.

```java
public class MemberService {
 private final MemberRepository memberRepository;
 public MemberService(MemberRepository memberRepository) {
 this.memberRepository = memberRepository;
 }
 ...
}
```
이런식으로 회원 서비스 코드를 DI 가능하게 변경했었다.

✔이런 DI라는 개념을 처음 접하고 사실 아직 어떤 필요성이 있는지 정확하게 몰라 추후 개념 재정립이 필요

여기서 빌드를 하면 에러가 발생한다.
이는 스프링 빈을 등록하지 않아서 발생하는 문제이다

스프링 빈을 등록하는 건 2가지 방법이 있다.

먼저 **컴포넌트 스캔**과 자동 의존관계 설정이다.
@Component 어노테이션을 사용하면 스프링 빈이 자동으로 등록
@Controller가 스프링 빈으로 자동 등록된 이유 또한 컴포넌트 스캔 덕분

@Component를 포함하는 다음 어노테이션도 스프링 빈으로 자동 등록
@Controller / @Service @Repository

즉 멤버 서비스 객체에 @Service를 사용해서 컴포넌트 어노테이션을 스프링빈에 자동 등록된다.


그럼 여기서 질문
>같은 단계 하위가 아닌 패키지를 생성하고 컴포넌트를 통해서 스프링 빈 등록이 가능하냐?
>
>답은 불가능하다❌. 기본적으로 시작하는 @SpringBootApplication 의 하위 패키지를 쭉 스캔하며 등록하기 떄문이다.

아래는 예제 코드

```java
@Service
public class MemberService {
   private final MemberRepository memberRepository;
   
   @Autowired
   public MemberService(MemberRepository memberRepository) {
   		this.memberRepository = memberRepository;
   }
}
```
>생성자가 1개이면 @Autowired 어노테이션 생략 가능

리포지토리 클래스에도 어노테이션 추가

스프링 컨테이너는 memberController에서 부터 의존성을 찾아서 스프링 빈을 찾는다.
![](https://velog.velcdn.com/images/kimdodo/post/2bfad89f-fd59-421f-8d93-c9b34eac98ac/image.png)

> 여기서 스프링은 스프링 컨테이너에 스프링 빈 등록시 싱글톤 디자인으로 객체 등록

자바 코드를 통해서 스프링 빈 등록하는 코드는 다음과 같다
먼저 SpringConfig 패키지를 생성하고 필요한 의존성인 멤버 서비스 그리고 멤버 서비스가 필요한 멤버 리포지토리를 각각 빈에 등록한다.

```java
package hello.hellospring;

import hello.hellospring.repository.MemberRepository;
import hello.hellospring.repository.MemoryMemberRepository;
import hello.hellospring.services.MemberService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class SpringConfig {
    @Bean
    public MemberService memberService(){
        return new MemberService(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository(){
        return new MemoryMemberRepository();
    }


}

```
여기서 주의 사항 있다.

우리는 지금 멤버 서비스가 생성자를 통해서 멤버 컨트롤러에 주입되고
멤버 리포지토리가 생성자로 멤버 서비스게 주입된다

```java
#멤버 서비스 생성자
 public MemberService(MemberRepository memberRepository){
        this.memberRepository = memberRepository;
    }
```
> DI는 필드 주입,setter주입, 생성자 주입 크게 3가지 방식이 있다.
의존관계가 실행 중에 동적으로 변화하는 경우는 크게 없어 생성자 주입 권장

필드 주입은 추후에 변경이 어렵기 때문에 권장 ❌
생성자 주입은 일반적인 경우
setter 주입은 setter를 통해서 주입되지만 실질적으로 한번 생정된 인스턴스가 바뀌는 일이 거의 없음으로 public 하게 함수를 열어두는게 좋지 않음

처음 로딩되고 조립되는 시점에 의존성 주입을 완료하는 생성자 주입 선호
>실무에서는 정형화된 컨트롤러, 서비스, 리포지토리 같은 코든느 컴포넌트 스캔 사용
상황에 따라 클래스 변경이 필요하면 스프링 빈 등록

여기서 중요한건 상황에 따라 구현 클래스를 변경해야한다라는 상황이 중요한 요소
정형화된 상황이라면 괜찮지만 해당 강의에서는 데이터 저장소가 선정되지 않아 메모리 리포지토리를 사용

실제로 메모리리포지토리로 구현된 상황에서 DB나 저장소가 선정된 이후 기존의 운영 코드를 수정하지 않고 바꿔치기 하는 방법을 위해서 스프링 빈 등록을 사용한다.

💥직접 스프링 빈 등록을 하면 DB를 바꿨을 때 리포지토리 빈 등록의 반환되는 부분만 변경하면 된다