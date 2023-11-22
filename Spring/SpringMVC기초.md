# Spring MVC 기초

> 💻 스프링 입문 - 코드로 배우는 스프링 부트, 웹 MVC, DB 접근 기술

[이전강의](https://velog.io/@kimdodo/Spring-Bean) 에서 @Controller를 사용하고 이를 위해서 스프링 Bean 등록을 위한 컴포넌트 스캔, 자바 코드를 통한 DI(의존성 주입)을 해보았다.
그럼 컨트롤러를 사용하는 웹 개발에서 자주 사용하는 Spring MVC를 통한 회원 웹기능 구현을 해보자

## 홈 컨트롤러
먼저 아래 home 컨트롤러를 새로 만들고 웹의 가장 root 경로에서 home 정적 리소스로 넘겨주는 걸 해보자
```java
package hello.hellospring.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class HomeController {

    @GetMapping("/")
    public String home(){
        return "home";
    }
}
```
이렇게 home으로 넘겨주면 home.html을 호출한다.
그러면 여기서 드는 의문 ❓
이전에 root 템플릿으로 저장한 index.html은 왜 안나오는거지?
아래 그림을 통해서 확인 가능하다.
![](https://velog.velcdn.com/images/kimdodo/post/c2ac1e67-a59f-46a9-bc99-61f10ab3b49b/image.png)
이 그림은 이전에 설명에 나온 예시이다.
웹 브라우저에서 리퀘스트가 오면 내장 톰켓을 통해서 컨트롤러로 처음 가게된다.
컨트롤러에서 home을 호출하기 때문에 home이 호출되고 index는 무시되는것
> 💥 컨트롤러가 정작 파일보다 우선순위가 높다

## 회원 가입
이제 회원 서비스를 간단하게 구현해보자
먼저 구현 코드이다.
```java
package hello.hellospring.controller;

import hello.hellospring.domain.Member;
import hello.hellospring.services.MemberService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;

@Controller
public class MemberController {
    private final MemberService memberService;

    @Autowired
    public MemberController(MemberService memberService) {
        this.memberService = memberService;
    }

    @GetMapping("/members/new")
    public String createForm(){
        return "members/createMemberForm";
    }

    @PostMapping("members/new")
    public String create(MemberForm form){
        Member member = new Member();
        member.setName(form.getName());

        memberService.join(member);

        return "redirect:/";
    }

}
```
코드에서 회원가입을 누르면 아래 그림처럼 생긴 웹 페이지가 나온다
![](https://velog.velcdn.com/images/kimdodo/post/a27e8cdc-aca2-408f-9fad-70ac9cac9bd3/image.png)
여기서 이름을 입력하고 등록을 클릭하면 회원이 등록되는 아주 간단한 구조이다.

그럼 해당 웹 페이지가 어떻게 보여지고 회원 등록이 되는지 일련의 과정을 알아보자
>1. 시작 페이지에서 회원가입을 클릭
2. 회원 가입은 /members/new 로 맵핑 되고 해당 멤버 컨트롤러는 members/createMemberForm을 호출(리턴)
3.members/createMemberForm 을 템플릿에서 찾는다.
4. members/createMemberForm를 View Resolver에 선택되고 thymeleaf를 통해 렌더링
5. HTML 을 그래도 웹에 보여준다

그럼 HTML을 살펴보자
form 이라는 태그로 둘러진 곳을 통해서 입력 받는 하나의 폼을 생성한다.
input 태그를 통해서 입력을 받는다. 여기서 name은 추후 key 값으로 되기 때문에 중요
submit 즉 등록 버튼을 클릭하면 다음 과정이 수행된다

> 1. 등록 클릭
2. action 동작 수행
3. /members/new로 post로 전달 
4. PostMapping의 members/new를 찾아서 해당 함수 수행
5. create함수의 멤버 객체를 생성하고 전달받은 form에서 이름을 받아와서 멤버 서비스 join
6. redirect 수행

>📖 값을 추가 할때는 POST, 조회에는 GET을 주로 사용

```HTML
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<body>
<div class="container">
    <form action="/members/new" method="post">
        <div class="form-group">
            <label for="name">이름</label>
            <input type="text" id="name" name="name" placeholder="이름을 입력하세요">
        </div>
        <button type="submit">등록</button>
    </form>
</div> <!-- /container -->
</body>
</html>

```

## 회원 조회
```java
    @GetMapping("/members")
    public String list(Model model){
        List<Member> members = memberService.findMembers();
        model.addAttribute("members",members);
        return "members/memberList";
    }
```

멤버 컨트롤러 클래스에 해당 함수를 등록
멤버 서비스에서 현재 등록된 멤버를 findMembers함수로 반환 받고 이를 전달하고 members/nmemberList 템플릿에 찾는다
내부에서 타임리프를 통해서 렌더링 한다.

HTML 코드
```HTML
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<body>
<div class="container">
    <div>
        <table>
            <thead>
            <tr>
                <th>#</th>
                <th>이름</th>
            </tr>
            </thead>
            <tbody>
            <tr th:each="member : ${members}">
                <td th:text="${member.id}"></td>
                <td th:text="${member.name}"></td>
            </tr>
            </tbody>
        </table>
    </div>
</div> <!-- /container -->
</body>
</html>
```
> thymeleaf를 통해서 model에 attribute 로 추가한 모델의 값을 가져와 보여준다
> member 객체를 저장한 List를 전달
member 객체는 각각 id 와 name으로 이루어져있다
객체 멤버 변수를 each 문법을 통해서 하나씩 출력한다.

여기서 하나 주의점은 id, name은 private 멤버 변수라 직접 접근이 아닌 Java Property 접근 방식으로 접근
getter, setter를 통해서 접근해서 값 반환

>🥾 현재 메모리로 DB를 대신하고 있어 서버가 새로 시작되면 멤버 기록을 삭제된다.
DB를 통한 데이터 저장 및 접근 방식 파악해보자