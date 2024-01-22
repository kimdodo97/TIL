# 자바 람다식(Java Lambda)
Java 8의 핵심 문법인 Stream을 이해하기 위해서는 람다식을 이해해야 한다.
Java 8에서 주요한 문법중 하나인 람다식에 대해서 알아보자 

## 람다식이란
람다를 파이썬에서도 몇번 써보았는데 사실 습관처럼 코드를 한줄에 구겨넣을려고 사용한 경우가 대부분이다.
람다식은 쉽게 말하자면 **하나의 식**으로 표현하는 것이다.
> 한줄에 코드에 구겨넣는 느낌이긴 하다

```java
int[] arr = new int[5]
Arrays.setAll(arr, (i) -> (int) (Math.random() * 5) + 1);
 
// (i) - (int) (Math.random() * 5) + 1 람다식을 메서드로 표현
int method() {
	return (int) (Math.random() * 5) + 1;
    }
```
이런 식이 있을떄 method가 하는 코드의 역할을 람다식을 통해서 한 줄에 코드에 우겨넣는다

메서드는 랜덤 숫자를 만들어고 5를 곱한뒤 1을 더한 숫자를 반환하는 코드이다.
즉 랜덤 숫자를 생성하는 규칙을 가진 메서드이고 이를 통해서 Array에 랜덤한 코드를 모두 채우기 위해서 반복되는 상황을 람다식으로 함축 표현 한것

👉 람다식은 오직 람다식 자체로 메서드의 역할을 하기도 , 매개변수로 전달되어지기도, 메서드 결과로 반환 되기도 한다.

람다식을 통해서 **메서드를 변수**처럼 다룰 수 있는 것

## 람다식 사용버
람다식은 익명 함수인데, 메서드에서 이름과 반환 타입을 제거하고 매개변수 선언부와 {} 사이에 -> 를 추가해서 사용한다.
![](https://velog.velcdn.com/images/kimdodo/post/53e871a1-2f57-4f46-ad39-5d05a3eb146f/image.png)

두개의 값을 비교해서 대소를 비교하는 특정한 메서드가 존재한다고 가정해보자
원본 메서드는 다음과 같다.

한 단계식 람다식으로 바꿔보자

```java
int max(int x,int y){
	return x>y ? x : y;
}
# --------> 메서드명 날리기

(int x,  int y) -> (return x > y ? x : y;)

# -------> 반환 타입 날리기
(int x, int y) -> x > y ? x : y;
# -------> 매개변수 타입 날리기
(x,y) -> x>y ? x : y;
```
기본적인 람다식 작성법을 정리해보자
>1. 반환타입 매서드 이름 제거
>2. 매개변수의 선언부 괄호와 함수 바디 사이에 -> 추가
※문장이 한줄인 경우는 중괄호도 생략(세미콜론도 제거 + return문이 유일한 경우는 return 삭제)

## 함수형 인터페이스
람다식의 형태는 자바의 메소드를 변수로 선언하는 것처럼 보인다.
하지만 자바는 메소드를 단독으로 쓸 수 없는 대표적인 객체지향 언어이다.
사실은 코드상에서만 그렇게 보이고 실제로는 변수에서 메스드를 호출하는 객체의 개념

**람다식도 객체**인 것이다. 정확히는 **익명 클래스로 구현한 익명 객체**이다.

### 객체 지향 VS 람다식 표현
Java8에서 람다식이 중요한 문법인 이유는 Java7을 보자
```java
interface IAdd {
    int add(int x, int y);
}

class Add implements IAdd {
    public int add(int x, int y) {
        return x + y;
    }
}
        
public class Main {
    public static void main(String[] args) {
        // 생 클래스로 메소드 사용하기
        Add a = new Add();
        
        int result1 = a.add(1, 2);
        System.out.println(result1);
    }
}
```
객체 지향식 표현으로 더하기를 하는 클래스를 만들고 구현하면 위 코드처럼 된다.
근데 이 클래스가 만약 다회성이 아닌 경우라면 이 클래스를 구현하고 인스턴스를 선언하는 것이 비효율적이다.

```java
interface IAdd {
    int add(int x, int y);
}

public class Main {
    public static void main(String[] args) {
        // 익명 클래스로 정의해 사용하기 (일회용)
        Iadd a = new IAdd() {
            public int add(int x, int y) {
                return x + y;
            }
        };
        
        int result2 = a.add(1, 2);
        System.out.println(result2);
    }
}

public class Main {
    public static void main(String[] args) {
        // 람다 표현식으로 함축 하기
        IAdd lambda = (x, y) -> { return x + y; }; // 람다식 끝에 세미콜론을 잊지말자
        
        int result3 = lambda.add(1, 2);
        System.out.println(result3);
    }
}
```
그래서 익명객체를 통해서 일회성 오버라이딩을 수행하고 한번만 쓰고 버리는 것이다.
람다는 이 익명클래스의 코드를 짧게 줄이는 것을 말한다.

> 추상 클래스의 메소드를 람다식으로 줄이거나 하는 행위는 못한다
오로지 인**터페이스로 선언**한 **익명 구현 객체만 람다식**으로 표현 가능하다.
이런 인터페이스를 우리는 **함수형 인터페이스**라고 한다.


### 함수형 인터페이스란
자 그럼 함수형 인터페이스를 상세하게 알아보자
함수형 인터페이스는 딱 하나의 추상메소드가 선언된 인터페이스를 말한다.
위 예제에서 IAdd가 전형적인 함수형 인터페이스의 예제
근데 이떄 final 상수, default,static,private 메서드는 추상메서드가 아니라서 갯수로 인식 X

@FunctionalInterface 어노테이션을 통해서 컴파일러가 체크를 해주는 기능이 있음
함수형 인터페이스를 사용하려면 해당 어노테이션을 통해서 하나의 추상메서드만 선언 하는 것이 좋음

### 람다식 문제점
람다식의 가장 큰 문제점은 바로 반환 타입이다.
리턴타입, 전달 파라미터 타입도 없는 람다식은 어떤 타입인지 모른다. 소스 상에서는 하지만 컴파일을 된다.
어떻게 타입을 추론하는 것인가??

컴파일러는 람다함수식을 통해서 타입을 유추하는 것이다.
람다식을 받는 메소드의 매개변수 타입을 확인하고 함수형 인터페이스 정의를 통해서 추상 메소드를 확인, 이르 기반하여 타입 자동 판별을 수행

> 일반적으로 제네릭을 사용하는 경우가 많아서 제네릭에서 판별하는 경우에 대다수

Collection sort 메서드를 통해서 람다식 활용 법을 한번 알아보자
![](https://velog.velcdn.com/images/kimdodo/post/9fbde9c6-d1e6-427c-a0d1-d0b7a0938096/image.png)

컬렉션의 sort는 다음과 같이 정의
리스트를 인자값으로 전달하고 ,Comparator 즉 어떤 참조타입 변수를 비교하는 Comparator를 전달해야한다.
```java
import java.util.Arrays;
import java.util.Collections;
import java.util.List;

public class Main {
    public static void main(String[] args) {
        List<String> words = Arrays.asList("aaa", "bbb", "ccc", "ddd");

        Collections.sort(words, (s1, s2) -> Integer.compare(s1.length(), s2.length()));
    }
}
```
위 예시처럼 words 라는 리스트를 전달하고 s1,s2를 인자값으로 전달하는 람다식을 매개변수로 활용
해당 람다식은 리턴 값이 Integer 참조 변수를 비교하는 것인데 이 비교는 인자로 전달된 두개의 변수 변수는 String 

> 1. sort 메서드의 첫번째 매개변수로 List<String> 전달
>2. 첫번째 매개변수 타입 지정에 의해서 sort 매서드의 제네릭 타입은 String
> 3. Comparator 인터페이스 제네릭도 String, 추상메서드 매개변수 타입 String
>4. 최종적으로 람다 함수의 타입 구성은 int 형 메소드 반환 타입과 String 매개변수 타입 두개로 추론
  
개발자 가독성을 위해서 람다식에 변수형을 명시하기도 하지만 이는 취향 차이
  
다음에는 람다식을 활용하는 방법을 알아보자