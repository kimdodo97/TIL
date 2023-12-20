백엔드 직무로 직무 전환 준비를 하면서 충족되어야하는 부분은 코딩테스트이다.
나는 Python으로만 코딩테스트를 항상 응시해서 JAVA 문법 자체에 익숙해질 필요가 있다.

파이썬의 리스트와 거의 흡사하지만 문법적 구조가 조금 달라 배열과 리스트에 대해서 정리해보고자 한다.

## Array(배열)
여러 데이터를 하나의 이름에 집합시킨 것
논리적인 저장 순서와 물리적인 저장 순서가 동일하며 연속도니 메모리 공간 사용
인덱스 값으로 접근 가능 하며 선언과 동시에 크기 지정 필요

```java
int[] a = new int[5];
int[] b = {1,2,3,4,5};

//b[2], a[3] 이런식으로 접근 가능
```

인덱스를 통한 검색이 용이하다.
메모리 관리에도 좋은 성능

다만 동적 할당 구조가 아니기 때문에 크기 지정으로 인한 메모리 낭비가 있고 추후 변경이 어려움


## List(리스트)
순서가 있는 데이터의 집합
불연속적인 메모리 공간에 데이터들이 저장, 포인터를 통해서 연결
동적 할당이며 삽입,삭제 등이 자유롭다

### ArrayList
일반 배열과 ArrayList를 인덱스로 접근가능하다
초기용량은 10이며 초기에 지정한 용량을 초과시 1.5배씩 크기가 증가된다.

```java
List<Integer> listA = new ArrayList<>();     // 초기 용량은 10
List<Integer> listB = new ArrayList<>(100);  // 용량이 100인 리스트 생성

/**
리스트 초기화
**/
List<String> cities = Arrays.asList("Amsterdam", "Paris", "London");  // asList 로 초기화
// new 생성과 동시에 초기화
ArrayList<String> cities = new ArrayList<String>() {{
    add("Amsterdam");
    add("Paris");
    add("London");
}};
List<String> strings = List.of("foo", "bar", "baz");  // List.of 로 초기화
// stream으로 초기화
ArrayList<String> places = new ArrayList<>(Stream.of("Buenos Aires", "Córdoba", "La Plata").collect(Collectors.toList()));
```

각 각 방법으로 초기화가 가능하다.

|  | 배열    | ArrayList |
|----------|---------------|------------|
|크기 |  정적 할당    | 동적 할당 |
| 저장데이터 타입 | primitive type & object    | object |
| 제네릭 사용 | X | O |
| 길이 | length 변수 | size() 함수 |
| 데이터 저장 | =   | add() |

capacity가 존재하고 현재 size 가 있는 형태
capacity가 초과하면 현재까지를 복사해서 새 배열에 저장하는 방식

삽입 또는 삭제가 빈번하게 일어나면 LinkedList가 효율적
