# **Spring 빈 생명주기**

데이터베이스 커넥션 풀이나 네트워크 소켓처럼 애플리케이션 시작 시점에 필요한 연결을 미리 해두고 애플리케이션 종료 시점에 모두 종료하는 작업을 진행하려면 객체의 초기화 및 종료 작업 필요하다

간단하게 외부 네트워크에 미리 연결하는 객체를 하나 생성한다고 가정해보자  
NetworkClient는 애플리케이션 시작 시점에 connect()를 호출해서 연결을 가지고 애플리케이션 종료 시점에 disconnect를 호출해서 연결 종료

◼ 샘플 네트워크 클라이언크 

```
public class NetworkClient {
 private String url;
 public NetworkClient() {
 System.out.println("생성자 호출, url = " + url);
 connect();
 call("초기화 연결 메시지");
    }
를 호출해서 연
public void setUrl(String url) {
 this.url = url;
    }
 //서비스 시작시 호출
public void connect() {
 System.out.println("connect: " + url);
    }
 public void call(String message) {
 System.out.println("call: " + url + " message = " + message);
    }
 //서비스 종료시 호출
public void disconnect() {
 System.out.println("close: " + url);
    }
 }
```

◼ 테스트 코드

```
public class BeanLifeCycleTest {
    @Test
 public void lifeCycleTest() {
 ConfigurableApplicationContext ac = new 
AnnotationConfigApplicationContext(LifeCycleConfig.class);
 NetworkClient client = ac.getBean(NetworkClient.class);
        ac.close(); //스프링 컨테이너를 종료, ConfigurableApplicationContext 필요
    }
    @Configuration
 static class LifeCycleConfig {
        @Bean
 public NetworkClient networkClient() {
 NetworkClient networkClient = new NetworkClient();
            networkClient.setUrl("http://hello-spring.dev");
 return networkClient;
        }
    }
 }
```

테스트를 실행하면 어떻게 될까

당연하게도 url에 관련 정보는 모두 null이 나온다

이유는 생성자에서 connect,call 메서드가 모두 호출되는데 의존주입은 생성자가 아닌 수정자 주입을 쓰기 때문이다.

## **1\. 빈 라이프사이클**

생명주기 관리를 하기 이전에 Bean의 라이프사이클을 알아보자

스프링 빈은 객체 생성 이후 의존 주입을 수행한다.

기본적으로 객체 생성, 의존 주입 과정이 모두 수행되야 객체를 사용해서 어떤 로직을 수행하거나 원하는 명령을 수행할 수 있는 상태가 되는 것이다.

테스트 코드에서 발생한 문제는 바로 이것이다. 언제 의존 주입이 완료되는지 개발자는 알 수 없다. 

스프링은 이런 문제를 해결하고자 의존관계 주입이 완료되면 스프링 빈에게 콜백 메서드를 통해서 초기화 시점을 알려주는 다양한 기능을 제공

스프링 빈이 종료되기 직전에 소멸 콜백또한 전달하여 안전한 종료를 수행한다.

**◼ 스프링 빈 라이프 사이클**

컨테이너 생성 ⇒ 스프링빈 생성 ⇒ 의존관계 주입 ⇒ 초기화 콜백 ⇒ 사용 ⇒ 소멸전 콜백 ⇒ 스프링 종료

**초기화 콜백** : 빈이 생성, 의존관계 주입이 완료된 후 호출

**소멸전 콜백** : 빈이 소멸되기 직전에 호출

> 생성자 주입을 쓸때는 콜백을 어떻게 쓰는지 궁금했는데 생각해보니 초기화 콜백을 따로 만들필요가 없다.  
> 어차피 생성자 주입을 통해서 의존 주입이 추가로 이루어지지 않는데 추가로 메소드를 만들어 초기화 콜백 이라고 지정할 필요가 없는 것

> **✅ 객체 생성, 초기화 분리**  
> 생성자는 필수 정보만 받고 메모리를 할당받아 객체를 생성하는 것이 목적  
> 반면 초기화는 이렇게 생성된 값을 활용해서 외부 커넥션 등 무거운 동작을 수행한다.  
> 따라서 생성자 안에서 무거운 초기화 작업을 하는 것 보다는 **객체를 생성하는 부분**, **초기화 부분**을 **나누는 것**이 유지보수 측면에서 용이하다.  
> 초기화 작업이 매우 간단하거나 값 변경 정도면 굳이 분리할 필요는 없음

## **2\. 생명주기 콜백 지원 방법**

3가지의 빈 생명주기 콜백 지원이 가능하다.

1\. 인터페이스 활용

2\. @Bean 어노테이션 활용

3\. @PostConstruct, @PreDestory 어노테이션 활용

#### **◼ 인터페이스 활용**

```
 public class NetworkClient implements InitializingBean, DisposableBean
```

클라이언트 클래스에 **두가지의 인터페이스를 상속**시키는 방법이다.

이렇게 상속되면 함수 오버라이드를 통해서 2개의 메서드를 사용하게 된다.

```
 @Override
 public void afterPropertiesSet() throws Exception {
 connect();
 call("초기화 연결 메시지");
    }
    @Override
 public void destroy() throws Exception {
 disConnect();
    }
```

이렇게 되면 스프링에서 afterPropertiesSet를 초기화 콜백, destroy 소멸 콜백으로 인식한다.

Spring 에서 지원하는 방법이지만 **메서드 이름 변경 불가, 외부 라이브러리 적용 불가** 등 문제가 있음

그리고 스프링에서 지원하기 때문에 테스트에서 **스프링이 강제**된다.

#### **◼  @Bean 어노테이션 활용**

```
@Bean(initMethod = "???", destoryMethod = "???")
```

어노테이션을 통해서 지원된다.

initMethod 는 초기화 메서드 이름, destoryMethod  종료(소멸) 메서드 이름을 입력하면 Bean 등록시 해당 메소드를 각각 초기화, 종료라고 인식한다.

기존 코드 수정이 적으며 메서드 이름을 설정 가능

**스프링 빈이 스프링 코드에 의존하지 않으며**, 설정 정보를 사용하기에 **외부 라이브러리에서도 적용 가능**한 방법이다.

**👉 종료 메서드 추론**

종료 메서드 destoryMethod에는 특별한 기능이 있는데 바로 추론 기능이다.

이 속성의 구현코드를 따라가면 (inferred) 기능이 있다. 이는 대부분 close, shutdown 이라는 이름으로 종료 메서드를 만들고 destoryMethod는 이런 특징에 기반해서 종료 메서드 이름이 2개중 하나라면 굳이 파라미터로 주지 않아도 알아서 소멸 콜백을 추론해서 호출한다.

공백("")으로 등록하면 추론 기능을 사용하지 않는다.

#### **◼  @PostConstruct, @PreDestory 어노테이션 활용**

```
 @PostConstruct
 public void init() {
     System.out.println("NetworkClient.init");
     connect();
     call("초기화 연결 메시지");
}
@PreDestroy
 public void close() {
     System.out.println("NetworkClient.close");
     disConnect();
}
```

기존 초기화, 종료 메서드에 2개의 어노테이션을 등록하여 사용 가능하다.

이름 그대로 생성자 이후, 종료 이전이라는 이름을 가지고 이 어노테이션을 통해서 각각 초기화와 종료를 실행 가능하다.

**👉 특징**

◾ 스프링에서 권장하는 방식

◾ 코드 자체가 간단한 방법

◾ 패키지가 javax 패키지에 기반한다. 즉 스프링 종속적 기술이 아니라 자바 문법이라는 점

◾ 컴포넌트 스캔과 잘 융화되는 방법

단점으로는 외부 라이브러리에서는 쓰기 힘들다 그렇기 때문에 외부 라이브버리에서는 @Bean을 활용한 방법을 사용하면 된다.

<script>(adsbygoogle = window.adsbygoogle || []).push({});</script>