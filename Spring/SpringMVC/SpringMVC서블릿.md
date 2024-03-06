# **Spring MVC - Servlet**

Spring MVC를 통한 Model View Controller 구조가 일반적으로 사용하는 웹 어플리케이션의 구조이다.

그럼 MVC 패턴을 통한 구조를 만들어서 개발하는 것이 어떤 장점이 있는지 한번 알아보자

해당 포스팅에서 사용하는 모든 예시는 Username 과 Age 를 입력하여 멤버 정보를 저장하는 심플한 예제를 기반으로 한다.

또한 입력값은 모든 동일한 form을 통해서 입력하는 방식이다.

<script>(adsbygoogle = window.adsbygoogle || []).push({});</script>

## **Servlet & JSP**

#### **◼ Servlet 방식**

Spring MVC 가 등장하기 이전에는 Servlet을 통해서 해당 리퀘스트 리스폰스 처리를 수행했다.

아래와 같은 샘플 코드를 먼저 확인해보자

```java
@WebServlet(name="memberListServlet",urlPatterns = "/servlet/members")
public class MemberListServlet extends HttpServlet {
    MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    protected void service(HttpServletRequest reqeust, HttpServletResponse response) throws ServletException, IOException {
        List<Member> members = memberRepository.findAll();
        response.setContentType("text/html");
        response.setCharacterEncoding("utf-8");

        PrintWriter w = response.getWriter();
        w.write("<html>");
        w.write("<head>");
        w.write("    <meta charset=\"UTF-8\">");
        w.write("    <title>Title</title>");
        w.write("</head>");
        w.write("<body>");
        w.write("<a href=\"/index.html\">메인</a>");
        w.write("<table>");
        w.write("    <thead>");
        w.write("    <th>id</th>");
        w.write("    <th>username</th>");
        w.write("    <th>age</th>");
        w.write("    </thead>");
        w.write("    <tbody>");

        for (Member member : members) {
            w.write("    <tr>");
            w.write("        <td>" + member.getId() + "</td>");
            w.write("        <td>" + member.getUsername() + "</td>");
            w.write("        <td>" + member.getAge() + "</td>");
            w.write("    </tr>");
        }
        w.write("    </tbody>");
        w.write("</table>");
        w.write("</body>");
        w.write("</html>");
    }
}
```

해당 샘플인 요청이 들어오면 리포지토리에 존재하는 모든 멤버를 출력해주는 소스코드이다.

URL로 입력이 들어오면 응답은 HTML 코드를 response의 getWriter를 직접 적어서 보내주는 방식으로 구성되어 있다.

각 비지니스 로직을 수행한 결과를 HTML로 만들고 이는 동적으로 만들어서 응답한다.

이 방식을 보면 무슨생각이 들까❓

저걸 언제 저렇게 String 타입으로 하나하나 다 치고 있나 이런 생각이 들것이다.

또 변경해야하는 부분이 있으면 write를 하나하나 수정해야 하면 비효율적인 반복 작업을 해야한다.

즉 이런 문제를 해결하는 것이 바로 템플릿 엔진이다.

HTML 문서에 동적으로 변경해야하는 부분만 자바 코드로 수정하는 방법을 가능하게 해주는 것이 템플릿 엔진이다.

#### **◼ JSP**

JSP는 Java Server Page의 약자로 HTML 코드에 Java 코드를 넣어 동적인 웹 페이지를 생성하는 도구이다.

즉 HTML 내부에 Java 코드를 사용해서 원하는 결과 값을 쉽게 HTML 코드로 렌더링 할 수 있는 것이다.

예시를 한번 확인해보자

```jsp
 <%@ page import="java.util.List" %>
 <%@ page import="hello.servlet.domain.member.MemberRepository" %>
 <%@ page import="hello.servlet.domain.member.Member" %>
 <%@ page contentType="text/html;charset=UTF-8" language="java" %>
 <%
    MemberRepository memberRepository = MemberRepository.getInstance();
    List<Member> members = memberRepository.findAll();
 %>
 <html>
 <head>
    <meta charset="UTF-8">
    <title>Title</title>
 </head>
 <body>
 <a href="/index.html">메인</a>
 <table>
    <thead>
    <th>id</th>
    <th>username</th>
    <th>age</th>
    </thead>
    <tbody>
    <%
    for (Member member : members) {
        out.write("    <tr>");
        out.write("        <td>" + member.getId() + "</td>");
        out.write("        <td>" + member.getUsername() + "</td>");
        out.write("        <td>" + member.getAge() + "</td>");
        out.write("    </tr>");
    }
    %>
    </tbody>
 </table>
 </body>
 </html>
```

멤버 리포지토리와 멤버라는 객체를 import 하고 멤버 리포지토리에서 전체 멤버를 조회하여 보여주는 HTML 이다.

<%%> 내부에 소스 코드를 사용해서 원하는 정보를 출력하거나 연산을 통해서 추출하는 것이 가능하다.

#### **😢한계점**

JSP와 서블릿은 소스만 확인해도 명확한 한계점이 보인다.

먼저 사용자에게 보여지는 화면을 위한 HTML을 만드는 작업이 서블릿에서는 자바 코드 안에서 해야하는 문제점이 있었다

이 문제 해결을 위해서 JSP를 사용했지만 사용해보니 일부 문제가 있다.

먼저 JSP 파일 내부에 Java 와 HTML 코드를 모두 입력해서 관리해야하는 것이다. 하나의 파일이 **여러개의 역할을 동시**적으로 수행하고 이는 **유지보수에 아주 크리티컬**하게 작용할 수 있다.

그래서 이 문제를 해결하고 M**VC 패턴이 등장**하였다.

JSP는 보여주는 기능 즉 HTML 로 화면을 그려주는 역할에만 집중하도록 하는 것이다. 어떻게 하는지 한번 알아보자

## **MVC 패턴**

**MVC패턴**이 등장한 이유는 앞서 말한 것처럼 각 **역할을 명확히 분리하여 유지 보수성을 높이기 위한 것**이다.

유지보수만 잘하기 위함도 있지만 결정적으로 개발에서 중요하게 생각해야하는 변경 주기의 차이 때문이다.

변경 주기 즉 프론트 작업 (UI 변경) 과 백엔드 작업(비지니스 로직) 은 변경 주기가 같기도 하지만 다른 경우도 많다. 즉 버튼을 수정하기 위해서 디자인 소스 이외의 다른 비지니스 소스를 수정하지 않는다.

즉 **변경주기가 다른 것을 하나로 묶어서 관리하는 것은 유지보수에 어려움**을 줄 수 있다.

MVC 패턴 즉 Model / Controller / View 세가지는 각각 어떤 것을 의미하는지 보고 실제 웹 에서 동작 방식을 알아보자

**◾ Controller**

HTTP 요청을 받아서 파라미터를 검증하고, 비지니스 로직을 실행한다. 또한 뷰에 전달할 결과 데이터를 조회해서 모델에 담아준다.

**◾ Model**

뷰에 출력할 데이터를 담아둔다. 뷰가 필요한 데이터를 모델에 담아서 전달해서 뷰는 비지니스 로직이나 데이터 접근과는 무관하게 화면에 렌더링에만 집중

**◾ View**

모델에 담겨있는 데이터를 사용해서 화면을그리는 일에 집중한다. 여기서 HTML 을 생성하는 부분

> 실제 서비스에서는 서비스라는 계층을 만들어 비지니스 로직을 수행  
> 컨트롤러는 비지니스 로직에 있는 서비스를 호출하는 역할 담당  
> 비지니스 로직을 변경 시 컨트롤러의 코드도 변경 가능

![alt text](image.png)

위 그림처럼 MVC 로 각 역할만 수행하게 하며 서비스와 데이터 접근의 경우에는 계층을 분리해서 컨트롤러가 여러 로직을 수행하지 않게끔 한다.

JSP와 서블릿을 쓰는 단계에서 각 MVC로 구조를 분리하면서 가장 중요한 역할은 FrontController이다.

컨트롤러이지만 각 로직을 수행하는 컨트롤러를 호출해주는 공통 컨트롤러이다.

해당 컨트롤러가 필요한 이유는 공통 로직 수행때문에 필요하다.

예를 들어 로깅하는 과정이 반드시 필요한 상황에서 모든 컨트롤러에 로깅하는 객체 서비스를 실행하는 코드를 추가하기에는 반복적인 작업이 너무 많이 수행된다.

이런 문제를 해결하고자 공통 컨트롤러인 프론트 컨트롤러를 도입해서 공통 로직을 한번에 처리하는 것이다.

![alt text](image-1.png)


프론트 컨트롤러가 요청에 맞는 컨트롤러를 다시 호출하고 이를 통해서 나머지 컨트롤러를 서블릿을 쓰지 않아도 된다.

이 프론트 컨트롤러가 Spring 웹 MVC의 핵심이기도 하다.