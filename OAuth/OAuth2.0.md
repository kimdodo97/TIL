## OAuth 2.0

### 용어 정리

User : 사용자  
Mine : 내 서비스 (개발자의 운영 서비스)  
Their : 구글 페이스북 트위터 등 타 서비스(OAuth2.0 제공 서비스)

이렇게 3가지의 일반적인 용어를 OAuth 에서는 사용하지 않는다.  
그럼 각각의 용어가 어떻게 바뀌는지 한번 알아보자

Their ▶ Resource Server  
페이스북 구글 등 우리가 **제어하고자 하는 자원**(소셜 로그인 정보)를 **가지고 있는 곳** 이라는 의미로 Resource Server

User ▶ Resource Owner  
사용자는 제어하고자 하는 정보의 소유자 라는 의미로 Resource Owner 가 된다.

Mine ▶ Client
우리의 서비스가 리소스 서버에서 자원을 가져오는 역할을 가짐으로 클라이언트 라고 부른다.

✅ 공식 문서를 보면 Authorization Server 가 존재한다. Authrization은 즉 권한을 부여하는 서버로 정보를 가진 서버 그리고 권한 부여를 하는 서버를 분리한다  
하지만 일단 이부분을 Resource 서버로 통합해서 생각

> 여기서 많은 사람들이 우리의 서비스가 클라이언트가 되고 갑자기 기존의 클라이언트 즉 우리 서비스의 고객이 Resource Owner 가 되는 용어의 급격한 변경으로 인해서 개념적으로 접근하지 어렵게 느껴질 수 있다.

용어 정리를 통한 접근 사고의 전환을 먼저 항상 가지자

### OAuth 2.0 등록

우리는 OAuth 2.0 을 사용하는 주요 목적은 민감한 사용자의 정보를 타 서비스를 통해서 관리하는데 있다  
그러기 위해서 우리는 먼저 타 서비스 즉 Resource Server에 우리 서비스에 대한 정보를 등록해야 한다.

모든 서비스가 각각 개발자 센터 또는 다른 방법을 통해서 이 등록 절차를 수행한다.
하지만 공통적으로 **3가지의 정보**를 가진다.

1. Client ID : 노출 가능
2. Client Secret : 노출 절대 불가능
3. Authorized Redirect URI

우리는 국내에서 많이 사용하는 네이버를 기준으로 하겠다.

1. 네이버 개발자 센터에서 어플리케이션 등록 및 네이버 로그인을 선택
   ![alt text](image-1.png)

2. 서비스의 URL 와 Authorized Redirect URI 설정
   ![alt text](image.png)

3. 어플리케이션 등록 및 정보 확인
   ![alt text](image-2.png)

이렇게 3단계의 과정만 진행하면 쉽게 OAuth를 위한 3가지의 정보를 취득 가능하다.  
이 과정이 바로 Resource Server에 Client 를 Register 한 것이다.

### Resource Owner 승인 과정

앞선 과정들로 Resource Server는 3가지 정보를  
그리고 Client 는 Client ID, Secret 그리고 Redirect URI에 대한 컨트롤러 처리를 구현했을 것이다.

이때 만약 Resource 서버가 가지는 A,B,C,D 기능 중 B,C 정보만 취득하고 싶다면 어떻게 될까❔

> 어렵게 생각할 수 있지만 B,C 정보라는 건 위 예시의 사용 API 중 어떤 것만 필수로 쓸 것인가 이다
> ![alt text](image-4.png)

이제 여기서 클라이언트는 로그인 버튼을 제공하고 로그인 버튼을 클릭하면 아래 그림의
리소스 서버로 클라이언트 아이디 , scope , redirect_url 를 쿼리 스트링으로 가진 요청을 리소스 서버로 보낸다

![alt text](image-5.png)

그리고 이 상황에서 리소스 서버에서 리소스 오너가 로그인 되어 있는지를 확인하고 로그인이 안되어 있으면 로그인 하는 화면을 제공한다.

이제 로그인을 하고 나면 쿼리 스트링의 클라이언트 ID 와 redirect_url 을 확인해서 등록된 클라이언트인지를 확인한다.

확인 이후 클라이언트에게 scope에 해당 하는 정보 제공에 동의 하는지를 확인한다.

![alt text](image-6.png)

리소스 오너는 어떤 클라이언트가 너의 이런 정보 사용 동의를 요청 한다 라는 사실을 명시하고 동의 하면 리소스 오너의 식별자와 허용 스코프를 리소스 서버에 저장한다.

이제 우리는 리소스 오너(서비스 사용자)의 동의를 구하는 과정 까지 처리하였다.

### Resource Server 승인

이전에는 리소스 오너의 승인을 획득하였다
이제는 리소스 서버가 이 접근에 대한 승인을 해줘야한다.

리소스 서버는 바로 엑세스토큰을 지급하는 것이 아니다
먼저 Authorization Code를 쿼리 스트링 형식으로 헤더의 Location 에 URI로 담아서
리소스 오너에게 응답으로 전달한다.

![alt text](image-7.png)

로케이션의 URI로 클라이언트에게 요청을 하게 되고  
클라이언트는 Authorization Code를 가지게 된다.
클라이언트는 이렇게 받은 Authorization Code를 포함한 토큰 발급 요청을 리소스 서버로 보낸다.  
이때 중요한 점은 Client Secrect을 반드시 전달한다.

아래 그림과 같은 URI로 전달
![alt text](image-8.png)

리소스 서버는 쿼리 스트링에서 전달받은 스크릿 , code, 클라이언트 ID, redirect url 정보를 확인하고 모두 일치하면 다음 단계인 엑세스 토큰을 지급한다.
