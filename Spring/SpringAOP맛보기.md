# Spring AOP 맛보기

> 💻 스프링 입문 - 코드로 배우는 스프링 부트, 웹 MVC, DB 접근 기술
[실제 강의 링크](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%9E%85%EB%AC%B8-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8)


## Spring AOP

**AOP** 는 **Aspect Oriented Programming**의 약자로 관점 지향 프로그래밍
쉽게 설명하면 **핵심 로직, 공통 로직**을 **분리**해서 **모듈화** 하는 것이다.

예시를 들어보자면 우리가 계속 만든 회원 가입, 조회 로직에 대한 호출 시간을 측정하고 싶다
이럴때 핵심 사항은 회원 가입 서비스, 공통 관심 사항이 호출 시간 측정이다.

![](https://velog.velcdn.com/images/kimdodo/post/7f932fe6-24e7-4a45-b663-8781d22b65ea/image.png)

핵심 사항과 공통 사항이 분리되지 않은 채로 시간 측정하는 하는 방법을 알아보자


```java
public Long join(Member member) {
 long start = System.currentTimeMillis();
   try {
     validateDuplicateMember(member); //중복 회원 검증
     memberRepository.save(member);
     return member.getId();
   } finally {
     long finish = System.currentTimeMillis();
     long timeMs = finish - start;
     System.out.println("join " + timeMs + "ms");
   }
 }
```
위 코드처럼 try 를 통해서 내가 사용한 모든 메소드에 시작과 끝의 시간을 측정해야한다.

여기서 발생하는 문제는 다음과 같다.
> - 시간 측정은 핵심 관심 사항이 아님
- 시간 측정 로직은 공통 관심 사항임
- 시간을 측정하는 로직과 핵심 비지니스 로직의 혼합으로 유지 보수 어려움
- 시간 측정 로직을 별도의 공동 로직으로 만들기 어려움
- 시간 측정 로직 변경시 모든 로직을 찾아가면 변경 해야함

예를 들어서 100개의 메소드의 시간 측정을 위해서 100개의 메소드에 시간 측정 로직을 추가해야하고 ms 을 s 로 변경시 다시 100개를 모두 변경해야함

## AOP 적용
공통 관심 사항과 핵심 관심 사항을 분리해서 개발한다.

아래 그림 처럼 **기존 서비스 로직을 그대로** 두고 새롭게 공통 사항을 개발해서 연결하는게 중요'
![](https://velog.velcdn.com/images/kimdodo/post/538619af-e29c-4cb7-b759-f1b1ed1b6bbb/image.png)

시간 측정을 하는 AOP 소스 코드이다.
@Around("execution(* hello.hellospring..*(..))") 를 통해서 이 AOP 가 적용된 클래스 범위를 정한다.

그 이외의 로직은 동일한데 joinPoint.proceed()를 통해서 실행을 시킨다고 한다.

- ### 소스 코드
```java
package hello.hellospring.aop;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class TimeTraceAop {

    @Around("execution(* hello.hellospring..*(..))")
    public Object excute(ProceedingJoinPoint joinPoint) throws Throwable{
        long start = System.currentTimeMillis();
        System.out.println("START : " + joinPoint.toString());
        try{
            return joinPoint.proceed();
        } finally {
            long finish = System.currentTimeMillis();
            long timeMs = finish - start;
            System.out.println("END : " + joinPoint.toString() + " " + timeMs + "ms");
        }

    }
}

```
- ### 실행화면
![](https://velog.velcdn.com/images/kimdodo/post/4dac8c90-f9d4-48da-960c-118b6cba8bf0/image.png)

이렇게 AOP를 통해서 분리시 얻는 이점은 무엇인가??
> - 관심 사항 분리 가능
- 시간 측정 로직을 별도의 공통 로직으로 만듬
- 핵심 관리 사항 가독성, 간결성 유지
- 변경 필요하면 시간 측정 로직만 분리
- 원하는 적용 대상 설정 가능

## AOP 동작 방식
기존의 스프링 컨테이너는 아래 그림과 같은데 여기서 프록시로 가짜 멤버 서비스가 생성된다,
가짜 멤버 서비스가 컨트롤러의 요청을 먼저 받고 이후 실제 멤버 서비스를 실행한다.

![](https://velog.velcdn.com/images/kimdodo/post/c60b61ca-9145-4b8b-92db-43f6ea072c17/image.png)

![](https://velog.velcdn.com/images/kimdodo/post/24a0f432-2900-495f-a236-943655db839f/image.png)

AOP 적용된 전체 스프링 컨테이너의 모습이다.
왜 이렇게 되는지 원리, 추가 예제 등 다른 문제는 스프링 핵심 원리 강의에서 추가 설명 예정이다.
![](https://velog.velcdn.com/images/kimdodo/post/a1169572-c8aa-4bda-8075-3465629c1223/image.png)


