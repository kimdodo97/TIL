Java 8 문법의 핵심이라고 하면 Stream을 뺄 수 없다.
실제 예제 코드나 타 프로젝트 레퍼런스를 확인해도 Stream을 사용한 경우가 많았다.
그럼 Stream이 무엇이고 어떤 기능이 있나 알아보자


## 스트림(Stream)
기존 루프문 처리는 for, foreach와 같은 루프문을 사용했다.
이를 통해서 요소들을 하나씩 처리하면서 다루었다.
간단한 처리는 문제가 없지만 복잡한 처리를 하기 위해서 컬렉션의 크기가 커지고 루프문 저하 발생
여기서 컬렉션은 List, Set, Queue, Map과 같은 자료 구조 의미

이런 컬렉션 데이터를 선언형으로 쉽게 처리할 수 있는 기능이 **Stream** 이다
스트림은 병렬처리를 별도의 멀티 쓰레드 구현 없이 사용 가능하다.

그럼 예시를 통해서 한번 알아보자

>스트림 사용 無

```java
List<Apple> redApples = forEach(appleList, (Apple apple) -> apple.getColor().equals("RED"));

redApples.sort(Comparator.comparing(Apple::getWeight));

List<Integer> redHeavyAppleUid = new ArrayList<>();
for (Apple apple : redApples)
    redHeavyAppleUid.add(apple.getUidNum());
```

> 스트림 사용 有

```java
List<Integer> redHeavyAppleUid = appleList.stream()
        .filter(apple -> apple.getColor().equals("RED"))        
        .sorted(Comparator.comparing(Apple::getWeight))         
        .map(Apple::getUidNum).collect(Collectors.toList());  
 
 //병렬처리
List<Integer> redHeavyAppleUid = appleList.paralleStream()
        .filter(apple -> apple.getColor().equals("RED"))        
        .sorted(Comparator.comparing(Apple::getWeight))         
        .map(Apple::getUidNum).collect(Collectors.toList());  
```

스트림 사용이 없는 코드는 각각의 과정을 처리를 for, forEach로 수행
스트림을 사용하면 스트림 API를 통해서 처리 가능하다.
paralle을 통한 병렬처리도 가능하다.

>**👍 Stream 사용 이점**
1. 간결하고 가독성 증가
2. 함수 조립을 통한 유연성 증가
3. 병렬처리를 통한 성능 증가

지금까지는 Stream이 무엇이고 어떤 것을 개선하고자 나왔으며 어떻게 사용하는지 간단한 예시를 보았다.

그럼 정확한 정의가 무엇인지 한번 알아보자


## 스트림 정의(기술면접 대비)
스트림은 데이터 처리 연산을 지원하도록 소스에서 추출된 연속된 요소 라고 정의

- 연속된 요소
특정 요소 형식으로 이루어진 연속된 값의 집합 인터페이스 제공
컬렉션은 데이터 (ArrayList, Set ...)
스트림은 계산 (filter, sorted, map)

- 소스
컬렉션, 배열, I/O 등의 데이터 제공 소스로 부터 데이터를 소비
정렬된 컬레션으로 스트림을 생성하면 정렬이 그대로 유지

- 데이터 처리 연산
SQL 질의문과 비슷한 연산 처리 가능
filter, sort, map 등으로 데이털르 조작하고 순차 혹은 병렬 실행도 가능


✔ 해당 강의는 자바의 정석 책에 기반하여 Stream 추가 정리 예정
