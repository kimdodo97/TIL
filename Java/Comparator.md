# Comparable / Comparator
Heap을 구현해야하는 상황에서 Comparator 사용하는 예제 코드가 있었다.
해당 Comparator가 어떤 역할을 수행하는지 알아보고 해당 글을 정리하게 되었다.

Comparable / Comparator는 모두 인터페이스이다.
인터페이스를 생각하면 무엇을 해야하나
그렇다 두개의 역할 즉 당신들의 배역이 무엇인지 알아봐야한다.

그럼 과연 두개의 인터페이스는 자바에서 어떤 배역을 가지고 있을까?

### 역할(배역)
객체를 비교하는 역할

공식 문서에 따르면 이렇다.
> A comparison function, which imposes a total ordering on some collection of objects

번역하자면 개체 컬렉션에서 전체 순서를 적용하는 비교 기능이다.
그럼 컬렉션을 비교하는것이 왜 중요할까
int 형, double형 등 실수형 변수들을 부등호를 통한 비교가 쉽게 이루어 진다.

하지만 클래스의 객체가 존재한다고 생각하면 어떠한가

예를 들어서 학생 클래스 인스턴스 들이 존재한다고 해보자
```java
public class Test {
	public static void main(String[] args)  {
 
		Student a = new Student(17, 2);	// 17살 2반
		Student b = new Student(18, 1);	// 18살 1반
	}
}
 
class Student {
 
	int age;			// 나이
	int classNumber;	// 학급
	
	Student(int age, int classNumber) {
		this.age = age;
		this.classNumber = classNumber;
	}
}
```

a,b 두개의 인스턴스를 어떻게 비교할거냐?
객체간에 부등호 처리를 통해서 당장 비교 할 수 있는 방법이 없다.

즉 객체간의 우선순위를 비교하기 위한 방법으로 Comparable/Comparator가 사용되는 것이다.

객체를 비교하기 위한 배역을 가지고 있다는 것이다.

두개가 매우 유사한데 어떤 차이가 있을까?

### 차이점
- Comparable
Comparable을 구현한 객체의 compareTo 는 매개변수가 1개
이는 자기자신과 매개변수를 객체를 비교

- Comparator
구현 객체의 compare 의 매개변수는 2개
이는 들어온 두개의 객체를 비교

즉 정리하자마녀 Comparable은 자기자신과 파라미터로 들어온 객체를 비교하지만,
Comparator는 자기자신의 상태가 어떻던 간데 파라미터로 들어오는 두 객체를 비교하는 것이다.
비교라는 배역은 동일하지만 두 배역의 상세 역할이 또 다른 것이다.


### Comparable 상세 설명
```java
public interface Comparable<T>
```
인터페이스가 이렇게 정의 되어 있고 이를 이용해서 어떤 비교를 수행해보자
예시는 아까 학생의 경우로 한다.

```java
class Student implements Comparable<Student> {
 
	int age;			// 나이
	int classNumber;	// 학급
	
	Student(int age, int classNumber) {
		this.age = age;
		this.classNumber = classNumber;
	}
	
	@Override
	public int compareTo(Student o) {
		/*
		 * 비교 구현
		 */
	}
}
```
인터페이스이기 때문에 인터페이스를 상속한 Student 클래스를 정의한다.
compareTo 즉 이제 새로운 Student 클래스가 인자값으로 전달되어도
내가 원하는 우선 순위 멤버 변수를 기반으로 하여 비교가 가능한 것이다.


### Comparator 상세 설명
비슷하지만 다른 의미의 Comparator을 한번 알아보자
이는 두개의 매개변수 객체를 비교하는 것이다.
```java
public interface Comparator<T>
```
라고 인터페이스 정의는 동일하다.
한번 상속해서 클래스 구현을 통해서 알아보자
```java
import java.util.Comparator;	// import 필요
class Student implements Comparator<Student> {
 
	int age;			// 나이
	int classNumber;	// 학급
	
	Student(int age, int classNumber) {
		this.age = age;
		this.classNumber = classNumber;
	}
	
	@Override
	public int compare(Student o1, Student o2) {
    
		// o1의 학급이 o2의 학급보다 크다면 양수
		if(o1.classNumber > o2.classNumber) {
			return 1;
		}
		// o1의 학급이 o2의 학급과 같다면 0
		else if(o1.classNumber == o2.classNumber) {
			return 0;
		}
		// o1의 학급이 o2의 학급보다 작다면 음수
		else {
			return -1;
		}
	}
}
```
두개의 객체를 비교하는 것으로 인자값을 두개의 객체를 설정하고 구현해보았다.
두 학급에 따라 비교하는 클래스이다.

그럼 두개가 가지는 차이는 말 그대로 자기자신의 유무이다.
Comparator는 자기자신과 별개로 파라미터로 받은 두개의 객체를 비교하는 것이다.

그럼 의문이 생길것이다.
> 어차피 생성한 인스턴스로 비교하면 Comparable만 상속하면 되는데 Comparator는 왜 쓰는거지?

우리가 Comparator를 쓰는건 여러 객체를 비교하는 용도로 쓰고 싶은 것이다.
이는 자바의 익명 객체를 활용하면 된다.

### ❓ 익명 객체
익명객체란 말 그대로 이름이 정의되지 않은 객체를 의미한다.
현재의 고민인 특정 구현 부분만 사용하고 싶은데 기능도 나중에 바꿔야될 수 도 있는 상황일때 익명객체라고 한다.

```java
public class Anonymous {
	public static void main(String[] args) {
	
		Rectangle a = new Rectangle();
		
		// 익명 객체 1 
		Rectangle anonymous1 = new Rectangle() {
		
			@Override
			int get() {
				return width;
			}
		};
		
		System.out.println(a.get());
		System.out.println(anonymous1.get());
		System.out.println(anonymous2.get());
	}
	
	// 익명 객체 2
	static Rectangle anonymous2 = new Rectangle() {
		
		int depth = 30;
		@Override
		int get() {
			return width * height * depth;
		}
	};
}
 
class Rectangle {
	
	int width = 10;
	int height = 20;
	
	int get() {	
		return height;
	}
}
```
우리가 아는 객체 생성은 new Rectangle()
이런식이다. 근데 익명객체는 할당 후 구현부를 추가한다.

즉 객체를 생성했고 이는 Rectangle이기는 했는데 메소드 Override를 했다.
쉽게 이해하자면 Rectangle을 상속했지만 이름을 새로 정하지 않고 기능만 Override 된 새로운 class
>핵심은 이름이 정의되지 않았다는 것이다.

상속의 개념은 담고 있지만 이름을 명시하지 않은 것이 익명 객체다.

그럼 다시 본론으로 돌아가보자

### Comparator 익명 객체
```java
Comparator<Student> comp1 = new Comparator<Student>() {
			@Override
			public int compare(Student o1, Student o2) {
				return o1.classNumber - o2.classNumber;
			}
		};
```
결국 Comparator를 통해서 이루고자 하는 목표 즉 해당 배역의 구현체의 역할은
객체의 비교를 수행하는 것이다.

이를 Student라는 객체간의 비교를 수행하는 하나의 배우를 만들었고 이를 익명 객체를 통해서 구현한 것이다.

> Heap 구현에서 Comparator를 사용하는 이유도 객체 비교
즉 객체로 구성된 완전 이진 트리의 객체 Array를 비교하기 위함

Arrays.sort(객체 어레이, compare 익명객체) 를 전달하는 방식을 통하면 더욱 쉽게 객체 어레이 정렬 가능
