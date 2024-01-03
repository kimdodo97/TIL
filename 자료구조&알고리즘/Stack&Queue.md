# 로드밸런싱
작성일 : 2024-01-03

## 개요 
코딩테스트를 위한 알고리즘 공부를 위해서 자주 사용되는 자료구조 및 알고리즘 공부를 위한 내용 정리를 수행한다.
전체 자료구조와 알고리즘은 Java를 기반해서 구현되고 설명된다.
시리즈의 첫번째 스택&큐이다.

## Stack
Stack 말 그대로 직역하면 쌓다 이다.
하나의 네모 박스 안에 책을 하나씩 쌓는 형태를 생각하면 이해가 쉽다.
![](https://velog.velcdn.com/images/kimdodo/post/7451f019-2f37-4e8e-92d8-c8b8868fe87f/image.png)
위 그림 처럼 0번부터 2번까지 순차적으로 스택이라는 자료 구조에 들어왔다.
이떄 가장 늦게 들어온 데이터가 가장 먼저 나가게 되는 **LIFO(Last In First Out)** 구조를 가진다.

흔하게 사용되는 경우는 수식계산, undo/redo 뒤로가기 와 같은 이전 작업을 가장 먼저 호출하는 상황에 유용하게 쓰인다.

자바에서 해당 자료구조를 자체적으로 제공하고 있다
![](https://velog.velcdn.com/images/kimdodo/post/71c61a78-229d-45e4-9d4f-f902dc0fdf29/image.png)

```java
import java.util.*;

public class Example{
	public static void main(String[] args){
    	Stack st = new Stack();
        st.push("0");
        st.push("1");
        st.push("2"):
        
        st.pop();
        st.pop();
        st.pop();
    }
}
```
이렇게 샘플 코드를 짜면 2,1,0 순서로 출력이 이루어짐

## Queue
Queue는 스택과 반대의 역할을 수행하는 경우이다.
흔히 선입 선출이라는 개념을 보여주는 자료구조로 먼저 들어온 데이터가 먼저 나가는 형태의 FIFO 구조로 이루어져있다.
Stack 과 다르게 인터페이스만 지원하고 클래스가 없다.
인터페이스를 구현한 클래스를 사용해야하는데 해당 예제에서는 링크드리스트를 사용
아래는 Queue 인터페이스이다.
![](https://velog.velcdn.com/images/kimdodo/post/64b44b76-5b80-4376-9412-3d514ad5bc0e/image.png)
Stack은 push , pop이 삽입 삭제 연산이었다면 Queue 는 offer,poll 이라는 경우는 사용한다.
이제 각 메소드의 차이점은 null이나 false를 반환하거나 예외처리를 통한 예외 메세지 반환의 경우이다. 이것은 각 사용 상황에 따라 이용
> 코딩테스트에서는 예외처리보다 null 이나 false 반환이 용이할것으로 판단

```java
import java.util.*;

public class Example{
	public static void main(String[] args){
    	Queue q = new LinkedList();
        st.offer("0");
        st.offer("1");
        st.offer("2"):
        
        st.poll();
        st.poll();
        st.poll();
    }
}
```

## 마무리하며
Stack 과 Queue 는 아주 대표적인 기본 자료구죠이다.
Queue는 왜 클래스를 지원하지 않고 인터페이스만 지원하는지, 그리고 우선순위 큐를 더 많이 사용하는데 그럼 우선순위 큐는 무엇인지 앞으로 알아볼 예정이다.

