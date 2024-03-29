HELLO SHOP
각 회원, 상품, 주문 3가지 서비스로 이루어지고
각 서비스는 2개의 기능을 가지고 있다.

회원 : 회원가입, 회원 목록
상품 : 상품 등록, 삼풍 수정, 삼품 조회
주문 : 상품 주문, 주문 조회, 주문 취소

상품은 재고 관리 필요
상품 종류는 도서, 음반, 영화 존재
상품을 카테고리로 구분
상품 주문시 배송 정보 입력

## 도메인 모델과 테이블 설계
![](https://velog.velcdn.com/images/kimdodo/post/71d9bc45-071b-43d1-94fb-96ca747fac62/image.png)

회원 여러개의 상품 주문 가능
한번 주문시 여러 상품을 선택가능 함으로 주문과 상품은 다대다
이런 다대다 관계를 관계형 DB또는 엔티티에서도 거의 사용 X
그래서 주문상품이라는 엔티티를 추가해서 다대다를 일대다, 다대일로 변경

상품 분류: 상품은 도서, 음반, 영화로 구분 되는데 상품이라는 공통 속성을 사용하기 때문에 상속 구조로 표현

![](https://velog.velcdn.com/images/kimdodo/post/9b06d80a-f283-4e14-9d4b-b72c25afe4e7/image.png)

회원
이름과 임베디드 타입인 주소(Address), 그리고 주문 리스트를 가진다.

주문 : 한번 주문시 여러 상품을 주문 가능함으로, 주문상품은 일대다 관례, 주문은 상품을 주문한 회원과 배송정보, 주문날짜, 주문 상태를 가지고 있다.
주문 상태는 열거형으로 주문, 취소 두가지 상태를 가지낟.

주문상품 : 주문한 상품 정보, 주문 금액, 주문 수량 정보를 가진다.
(일반적으로 OrderLine, LineItem)

상품 : 이름, 가격, 재고수량을 가지고 있다. 상품을 주문하면 재고 수량이 줄어든다. 상품의 종류로는 도서, 음반, 영화가 있는데 각각 사용하는 속성이 조금씩 다르다.

배송: 주문시 하나의 배송정보를 생성한다. 주문과 배송은 일대일 관계

카테고리 : 상품과 다대다 관계를 맺는다. parent, child로 부모, 자식 카테고리 연결

주소 : 값타입(임베디드 타입)/회원 과 배송에 사용

> 회원이 주문을 하기 때문에, 회원이 주문 리스틀르 가지는 것은 얼핏 보면 잘 설계한거 같지만 실무에서는 적합하지 않다.
회원이 주문을 참조하지 않고, 주문은 회원을 참조하는 것으로도 충분하다. 

![](https://velog.velcdn.com/images/kimdodo/post/74fce373-d9b1-430f-81d6-387c4340fa4a/image.png)


