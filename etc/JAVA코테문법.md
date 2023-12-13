# JAVA 코딩테스트 주요 문법 정리
Python에서 Java로 주력 개발 언어 변경 중인 시점에서 여러 기업의 최초 필터링은 코딩테스트이다.
코딩테스트를 준비하는 와중에 파이썬과 Java에서 사용하는 문법적 차이점이 존재하여 이를 정리해본다.

> 💢 Python list는 array처럼 접근 되는데 안되는게 제일 화가남

## 라이브러리 추가
대부분 사용하는 라이브러리는 util 내부
```java
import java.util.*;
import java.io.*;
```

## Array
```java
// 오름 차순 정렬
Arrays.sort(arr1);

// 내림차순 정렬
Arrays.sort(arr1, Collections.reverseOrder());

// 일부만 정렬 0-3번까지의 인덱스만 정렬 
Arrays.sort(arr1, 0, 4)

// 오름차순 정렬이면 binary search로 특정 값을 찾을 수 있다.
Arrays.binarySearch(arr1, 2);  // 조회 속도 빠름

// 배열을 리스트 변환❗
List list = Arrays.asList(arr1);

// 배열의 특정 범위 자르기 (파이썬 슬라이스)
int tmp[] = Arrays.copyOfRange(arr1, 0, 3);
```

## 길이 반환
```java
//array length 변수
arr.length

//String length 변수
str.length

//ArrayList는 size 메소드
ArrayList.size()
```

## String 문법
```java
// 특정 문자열 기준으로 String 분리
str.split("?");

//문자열 길이
str.length();

//문자열 일부 추출
str.substring(0,5);

//문자열 문자 접근
str.charAt(0);

//문자열 배열로 변환
String[] Arr = str.split("");

//대소문자 변환
str = str.toUpperCase();
str = str.toLowerCase();	

// 한번 쓴 문자열은 변경 불가. substring 으로 새로 선언
String name="starfucks";
String newname=name.substring(0,4)+'b'+name.substring(5);
```

## HashMap 문법
파이썬의 딕셔러니 또는 Set 기능

```java
//key value 타입 지정 후 선언
HashMap<String, Integer> hm = new HashMap<>();

// key-value 넣기
hm.put("java", 0);

// 키로 값 가져오기
hm.get("java");

// containsKey()로 존재유무 확인
if (!hm.containsKey("java")) hm.put("java", 1);

// 특정 키가 없으면 값 설정, 있으면 기존 값 가져오는 함수
hm.put("java", hm.getOrDefault("java", 3);	

// KeySet() 함수로 맵 순회
for(String key : hm.KeySet()) {				
	hm.get(key);
}
```

## ArrayList 문법
```
// 선언
ArrayList<String> list = new ArrayList<>();

// 삽입
list.add("java");			// {"java"}
list.add(0, "ryu");			// {"ryu", "java"} (0번째 인덱스에 삽입)

// 수정
list.set(1, "c++");			// {"ryu", "c++"}

// 삭제
list.remove(1);				// {"ryu"}

// 값 존재 유무 확인
list.contains("java");		// false
list.indexOf("ryu");		// 0 존재하면 인덱스 리턴

// iterator 사용
Iterator it = list.iterator();

// 인덱스 오름차순 순회
while (it.hasNext()) {
	...
}

// 인덱스 내림차순 순회
while (it.hasPrevious()) {
	...
}

// 중복없이 값 삽입
if (list.indexOf(value) < 0) {	// 없으면 -1을 리턴하기 때문에
	list.put(value);
}

// 리스트 값 하나씩 가져올 때 (int 일 경우)
for(int i = 0; i < list.size(); i++) {
	list.get(i).intValue();
}
```

## Queue
```java
// 선언
Queue<Integer> q = new LinkedList<>();		// linked list로 선언해야함

// 삽입
q.add(10);			// {10}
q.offer(2);			// {10, 2}

//  프론트값 반환
q.peek();			// 10

// 삭제
q.remove();
q.poll();

// 초기화
q.clear();

// 비었는지
q.isEmpty();

// pair 같은 경우는 그냥 구현해서 사용
static class Node{
        int y;
        int x;
        int dist;
        
        Node(int y,int x,int dist){
            this.y=y;
            this.x=x;
            this.dist=dist;
       }
   }

Queue<Node> queue=new LinkedList<>();
queue.add(new Node(1,2,3));
Node node= queue.poll();

```


## 우선순위 큐
우선순위 큐 같은 경우는 자료 구조를 정리하는 과정에서 추가로 알아보자
```java
// 선언
PriorityQueue<Integer> pq = PriorityQueue<Integer>();	// 최소힙
PriorityQeueu<Integer> pq=PriorityQueue<Integer>(Collections.reverseOrder());	// 최대힙

// 삽입
pq.add(3);

// 삭제
pq.remove();

// root 값 추출
pq.peek();

// pair 사용 시 
import java.io.IOException;
import java.util.PriorityQueue;

public class PQ {

    static class Node{
        int y;
        int x;

        Node(int y,int x){
            this.y=y;
            this.x=x;
        }
		
        // 비교 함수 만들어야함!!
        public int compareTo(Node p) {
            if(this.y < p.x) {
                return -1; // 오름차순
            }
            else if(this.y == p.y) {
                if(this.x < p.x) {
                    return -1;
                }
            }
            return 1;
        }
    }

    public static void main(String[] args) throws IOException{

        PriorityQueue<Node> pq1=new PriorityQueue<>(Node::compareTo);
        pq1.add(new Node(1,2));
        pq1.add(new Node(1,1));
        pq1.add(new Node(2,3));
        pq1.add(new Node(2,1));

        while(!pq1.isEmpty()){
            Node node=pq1.peek();
            System.out.println(node.y+" "+node.x);
            pq1.remove();
        }
    }
}
```