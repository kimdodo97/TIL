# **MyBatis 튜토리얼**

MyBatis를 처음 배우며 MyBatis가 무엇인지 그 사용법에 대한 가이드 포스팅을 해보려 한다.

MyBatis는 JPA와 함께 DB와의 연결을 편하게 해주는 프레임워크이다.

JPA가 최신기술이며 전세계적으로 많이 사용된다.

하지만 국내에서는 여전히 MyBatis를 사용하며 레거시 프로젝트는 대부분이 MyBatis 일 것이다.

**MyBatis 프레임워크**는 반복적인 JDBC 프로그래밍을 단순화하여, 불필요한 코드를 제거하고, Java 소스코드에서 **SQL 문을 분리하여 별도의 XML 파일**로 저장하고, 이 둘을 서로 연결시켜주는 기능을 제공한다.

그럼 이제 어떻게 사용하는지 한번 단계별로 알아보자

추가로 아래 링크는 MyBatis의 공식 docs 문서이다.

[mybatis.org](https://mybatis.org/mybatis-3/ko/index.html)

<script>(adsbygoogle = window.adsbygoogle || []).push({});</script>

## **1\. 환경 설정**

MyBatis는 기본적으로 환경 설정은 xml 파일을 통해서 수행한다.

#### **◼ Configuration**

MyBatis의 환경 설정 xml 파일이다.

jdbc 커넥션 정보 및 별칭, mapper 등의 대부분의 설정 정보를 담고 있다.

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <!-- jdbc.properties 등록 -->
    <properties resource="main/resource/config/jdbc.properties"/>
    <typeAliases>
        <typeAlias alias="Student" type="main.com.student.domain.Student"/>
    </typeAliases>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <!-- jdbc.properties 등록된 값을 참조: ${key} -->
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.userid}"/>
                <property name="password" value="${jdbc.passwd}"/>
            </dataSource>
        </environment>
    </environments>
    <mappers>
        <!-- DeptMapper.xml 등록 -->
        <mapper resource="main/resource/mapper/StudentMapper.xml"/>
    </mappers>
</configuration>
```

각 태그에 대해서 간단하게 알아보자

**👉 properties 태그** 

jdbc 커넥션 정보가 담긴 properties 파일을 저장하는 경로를 지정하거나 하위 엘리멘트도 지정가능

id나 passwd 등

**👉 typeAliases**

별칭 태그로 이후 mapper에서 사용할 resultType 등에 사용할 객체의 classpath 대신 별칭으로 관리

**👉 environment** 

트랙젝션 매니저, DB 커넥션 인스턴스를 가져오기 위한 DataSoruce를 포함한 태그

Datasource 태그 내부에서는 properties 또는 직접 db 연결 정보를 사용

**👉 mapper**

SQL 매핑 xml 파일을 지정

#### **◼ JDBC Properties**

jdbc 에 대한 설정 정보를 직접 xml 파일에 업로드 해도 되지만 jdbc.properties 설정 파일로 관리한다.

```
jdbc.driver=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/workshop
jdbc.userid=root
jdbc.passwd=1234
```

설정 파일에는 4가지의 정보가 포함되며 각 정보는 key, value의 형태로 저장된다.

> Git 또는 다른 외부 소스 공유시 db 접속 정보를 함께 올리는 것은 문제가 될 수 도 있다.  
> 그리고 스타트업에서 일하면서 느낀 건 설정 파일에서 중요한 정보 또는 변동이 필요한 정보는 별도의 파일로 관리하는 것이 유지 보수 측면에서 더 좋았다.

#### **◼ Mapper xml**

mapper 정보 즉 SQL 쿼리문이 담긴 파일이다.

이 파일은 설정 정보이자 실질적인 쿼리 분리 파일이다.

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="Mapper">

</mapper>
```

기본적으로 위 구조를 따르면 namespace를 지정해서 각 매퍼 xml 파일을 구분해준다.

이는 추후 기능들의 이름이 동일한 경우(findAll, findById..) 에 이름을 통해서 구분이 필요하다.

## **2\. Session 사용**

JDBC 에서는 Connection을 통해서 DB에 쿼리를 실행한 방식처럼 MyBatis에서는 SqlSessionFactory를 통해서 세션 인스턴스를 생성해서 사용한다.

말로 설명보다는 코드를 통해서 한번 보자

```
public class MySqlSessionFactory {
    private static String resource = "main/resource/config/Configuration.xml";
    private static volatile SqlSessionFactory sqlSessionFactory;

    private MySqlSessionFactory() {} // Private constructor to prevent instantiation

    public static SqlSessionFactory getSqlSessionFactory() {
        if (sqlSessionFactory == null) {
            try (InputStream inputStream = Resources.getResourceAsStream(resource)) {
                // Create SqlSessionFactory
                sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
            } catch (Exception e) {
                e.printStackTrace();
                throw new RuntimeException("Failed to initialize SqlSessionFactory: " + e.getMessage(), e);
            }
        }
        return sqlSessionFactory;
    }

    public static SqlSession openSession() {
        return getSqlSessionFactory().openSession();
    }
}
```

이 코드는 SessionFactory를 통해서 세션을 생성하는 클래스로 싱글톤으로 관리되게 만들었다.

앞에서 설정한 Configuraiton 파일의 경로를 통해서 SqlSessionFactory를 빌드하고 여기서 openSession을 통해서 세션 인스턴스를 생성한다.

이 세션은 반드시 사용이 끝나면 close를 통해서 닫아줘야한다.

> MyBatis를 사용하는게 처음이며 공부한 지식으로는 일반적으로 커넥션 풀에서 커넥션을 가져와 사용하고 반납하는 형식이라고 들었다.  
> 그 구조를 나름대로 이해하고 구현한 결과로 세션을 생성하는 세션 팩토리를 굳이 새롭게 할당하는 것보다 싱글톤으로 만들어 필요한 메서드에서 새롭게 세션만 생성하는 것이 더욱 좋은 방식이라고 생각되었다.