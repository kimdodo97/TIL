# 스프링 컨테이너 생성 및 빈
```java
ApplicationContext applicationContext = 
	new AnnotationConfigApplicationContext(AppConfig.class);
```
ApplicationContext 는 스프링 컨테이너다
ApplicationContext는 인터페이스이다. 즉 AnnotationConfigApplicationContext 가 인터페이스 구현체 라고 생각하면 될듯 (OCP)

> 스프링 컨테이너를 부를떄 Bean팩토리, ApplicationCOntext를 구분해서 이야기하긴 한다.

## 스프링 컨테이너 생성 과정
### 1. 스프링 컨테이너 생성
![](https://velog.velcdn.com/images/kimdodo/post/007bfa0b-d758-4b57-84a3-37067d487986/image.png)

AnnotationConfigContext를 통해서 생성
스프링 컨테이너 생성시 구성정보를 지정해야하는데 그 역할을 AppConfig.class를 인자로 전달하는 것

### 2. 스프링 빈 등록
![](https://velog.velcdn.com/images/kimdodo/post/40a94db2-381a-441c-8ad1-01d6b6736e6e/image.png)
AppConfig 내부의 빈으로 등록된 각 객체들을 보면서 스프링 빈 저장소에 빈 이름과 객체 정보를 등록

> 빈 이름 default : 메소드 이름 / 특수 상황 : name을 통한 이름 지정

🛑빈 이름은 항상 다른 이름을 부여해야함 / 같은 이름이면 에러나 충돌 발생

### 3. 스프링 빈 의존관계 설정 - 준비
![](https://velog.velcdn.com/images/kimdodo/post/30bd838e-cde3-44c2-9534-3973cfb92fff/image.png)![](https://velog.velcdn.com/images/kimdodo/post/205e7248-d417-4957-aa4d-969159bde531/image.png)

스프링 컨테이너는 설정 정보를 참고해서 의존관계를 주입한다
단순히 자바코드를 호출하는 것과는 차이가 있다.
이는 싱글톤 컨테이너에서 추가 설명한다.

> 스프링은 빈을 생성하는 단계와 DI 단계 구분
근데 자바 코드에 따르면 스프링 빈 등록과 동시에 생성자 호출로 DI도 한번에 처리된다.
다만 우리는 개념적으로 이런 과정을 거친다 라는 것을 알고있자

## Bean 조회
```java
AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

    @Test
    @DisplayName("모든 빈 출력")
    void findAllBean(){
        String[] beanDefinitionNames = ac.getBeanDefinitionNames();
        for (String beanDefinitionName : beanDefinitionNames) {
            Object bean = ac.getBean(beanDefinitionName);
            System.out.println("name = " + beanDefinitionName + " Object = " + bean);
        }
    }@Test
    @DisplayName("애플리케이션 빈 출력")
    void findApplicationBean(){
        String[] beanDefinitionNames = ac.getBeanDefinitionNames();
        for (String beanDefinitionName : beanDefinitionNames) {
            BeanDefinition beanDefinition = ac.getBeanDefinition(beanDefinitionName);
            if(beanDefinition.getRole() == BeanDefinition.ROLE_APPLICATION){
                Object bean = ac.getBean(beanDefinitionName);
                System.out.println("name = " + beanDefinitionName + " Object = " + bean);
            }
        }
    }
```

>getBeanDefinitionNames() : 모든 스프링빈 이름 조회
getBean() : 빈 이름으로 빈 객체 조회
getRole() : 스프링 빈의 상태 조회
ROLE_APPLICATION : 사용자가 등록한 빈
ROLE_INFRASTRUCTURE : 스프링이 내부에서 사용하는 빈