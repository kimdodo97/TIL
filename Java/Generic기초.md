# Generic 기초
제네릭 이름을 통해서 직독직해를 해보면 일반화, 일반적인 이런 의미로 보인다.

자 그럼 어떤게 일반적인가? 

데이터 형식에 의존하지 않고 하나의 값이 다른 여러 데이터 타입을 가게 하는 것이다.여러 가지 데이터 타입이란 정의된 인터페이스가 아니라 Integer, String, float 와 같은 기본 데이터 타입을 의미

가장 쉬운 예시로 ArrayList, LinkedList 와 같은 컬렉션 인터페이스 상속 클래스를 사용할 때를 생각해보면 쉽다.

```
ArrayList<Integer> list1 = new ArraytList<Integer>();
LinkedList<Double> list3 = new LinkedList<Double>();
```

자 이 예시를 통해서 보자면 <> 내부에 타입을 설정한다. 이때 이 선언된 데이터 타입으로 구성된 리스트들을 생성하게 되는 것이다.

👉 제네릭은 **클래스 외부에서** 사용자에 의해 **지정되는 것**을 의미한다.

### 1\. 기본 제네릭

제네릭을 사용할때 일반적으로 쓰는 암묵적 규칙이 있다 아래 표를 참고하자

| 타입 | 설명 |
| --- | --- |
| <T> | Type |
| <E> | Element |
| <K> | Key |
| <V> | Value |
| <N> | Number |

#### 1.1 클래스 및 인터페이스 

```
public class ClassName <T> { ... }
public interface InterfaceName <T> { ... }
```

제네릭 타입의 클래스나 인터페이스는 다음과 같이 선언한다.

T 타입은 블럭 안의 스코프에서만 유효하다.

단일 데이터 타입이 아니라 두개의 데이터 타입도 줄 수 있다.

HashMap 을 생각하면 편하다. Key - Value를 쌍으로 사용하는 경우

사용 예시를 간략하게 보자

```
public class ClassName <T> { ... }
 
public class Main {
	public static void main(String[] args) {
		ClassName<Integer> a = new ClassName<Integer>();
	}
}
```

외부에서 ClassName 이라는 선언된 클래스의 데이터 타입을 Integer로 사용 헀다. 

> 이떄 타입 파라미터는 참조 타입 즉 int, double, char 같은 primitive type 은 불가능

참조 타입만 사용 가능 하다는 것은 결국 클래스 타입을 사용도 가능하다는 것이다.

아래 예시처럼 Object라는 클래스로 구성된 ArrayList도 구성가능 한 것

```
ArrayList<Object> objectList = new ArrayList<Object>():​
```

#### 1.2 제네릭 클래스

클래스/ 인터페이스를 제네릭으로 받는 방법을 활용 해보자

아래와 같은 예시를 보자

```
class ClassName<E> {
	
	private E element;
	
	void set(E element) {
		this.element = element;
	}
	
	E get() {
		return element;
	}
	
}
```

이렇게 정의된 제네릭 클래스가 있다. 이 클래스를 사용하려면 어떻게 해야하나

```
ClassName<String> a = new ClassName<String>();

a. set("10");
System.out.println("a data : " + a.get());
System.out.println("a E Type : " + a.get().getClass().getName());
```

이렇게 정의하고 테스트를 해보면 즉 element는 String으로 타입 파라미터를 지정했기 때문에

결국 String 타입으로 정의되고 출력을 해도 String 타입임을 보여준다.

두개를 쓰는 것도 똑같다.

즉 제네릭 클래스를 활용하면 외부에서 타입을 지정해서 쓸 수 있는 클래스를 만들 수 있는 것이다.

#### 1.3 제네릭 메소드

제네릭 메소는 인자값, 반환 타입을 지정하는 것이라고 볼 수 있다.

특정 로직을 참조 타입에 상관없이 수행되는 그런 경우에 사용하기 좋다.

```
public <T> T genericMethod(T o) {	// 제네릭 메소드
		...
}
```

기존과 조금 다른것이 제네릭 타입을 메소드 반환 타입 이전에 선언하는 것이다.

즉 이 메소드는 제네릭 타입을 설정했기 떄문에 클래싀의 어떤 멤버 변수와는 무관하게 반환 타입, 인자 값 타입을 가지게 된다.

#### 1.4. 주의 사항

Static 메소드로 사용할 때 주의를 해야한다.제네릭을 통해서 외부에서 타입을 지정해주는데, 이는 객체가 인스턴스화 됬을때 타입을 넘겨주는 과정을 가지게 된다.근데 Static 은 어떤가, 프로그램 실행 과 동시에 이미 메모리에 올라가있다.

> 타입을 지정해준적이 없는데 메모리에 올라가야한다 

정상적인 선언 방식에서는 타입을 받아올 방법은 없다 ---> 에러 발생

제네릭이 사용되는 메소드를 정적메소드로 두고 싶은 경우에는 제네릭 클래스와 별도로 독립적인 제네릭 사용이 되어야 한다.

그래서 <>를 반환 타입 전에 선언하고 파라미터로 제네릭 타입을 지정하는 것이다.

객체 지향 언어인 Java에서 클래스와 별개의 Static 메소드로 관리하기 위해서 이런 번거로워 보이는 선언 방식을 사용한다.