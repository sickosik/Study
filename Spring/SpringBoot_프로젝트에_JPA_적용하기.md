# Spring Boot 프로젝트에 JPA 적용하기

## 1. pom.xml에 jpa 추가

`dependacy`를 추가

#### 1.1 maven

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
	<scope>runtime</scope>
</dependency>

<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
```

#### 1.2 gradle

```java
dependencies {
	compile 'org.springframework.boot:spring-boot-starter-data-jpa'
	compile 'mysql:mysql-connector-java'
}
```



## 2. properties 설정 (application.yml)

```properties
spring:
  application:
    name: {name 입력}
  datasource:
    driver-class-name: {jdbc driver 입력}
    url: {db endpoint 입력}
    username: {db 유저이름}
    password: {db 비밀번호}
    
  jpa:
    hibernate:
      ddl-auto: none #create update none
#      # naming 해주면 대소문자를 구분하여 테이블, 칼럼을 생성
#      naming:
#        physical-strategy: org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
    show-sql: false
```



## 3. Entity Class 생성

jpa 를 사용하려면 `@Entity` 클래스를 먼저 생성해야 한다.

```java
@Entity
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Table(name = "user")
public class User {

    @Id
    @Column(name = "user_id")
    private Integer userId;
    @Column(name = "korean_name")
    private String koreanName;
    @Column(name = "english_name")
    private String englishName;

    @Builder
    public User(Integer userId, String koreanName, String englishName) {
        this.userId = userId;
        this.koreanName = koreanName;
        this.englishName = englishName;
    }
}
```

데이터베이스에 저장하기 위해 유저가 정의한 클래스가 필요한데 그런 클래스를 `Entity`라고 한다.



#### **@Id**

primary key를 가지는 변수를 선언하는 것을 뜻한다.

`@GeneratedValue` 어노테이션은 해당 Id 값을 어떻게 자동으로 생성할지 전략을 선택할 수 있다.
`@GeneratedValue(strategy = GenerationType.AUTO)`를 사용하면 DB에 맞게 자동으로 생성해준다.



#### **@Table**

별도의 이름을 가진 데이터베이스 테이블과 매핑한다.

기본적으로 `@Entity`로 선언된 클래스의 이름은 실제 데이터베이스의 테이블 명과 일치하는 것을 매핑한다.

`@Entity`의 클래스명과 데이터베이스의 테이블명이 다를 경우에 `@Table(name=" ")`과 같은 형식을 사용해서 매핑이 가능하다.



#### **@Column**

`@Column` 선언이 꼭 필요한 것은 아니다.

기본적으로 멤버 변수명과 일치하는 데이터베이스 컬럼을 매핑한다.

`@Column`에서 지정한 변수명과 데이터베이스의 컬럼명을 서로 다르게 주고 싶다면 `@Column(name=" ")` 같은 형식으로 작성하면 된다.

컬럼에 언더바가 있을 경우에는 꼭 `@column`을 사용해야 한다.



## 4. JpaRepository 상속

```java
public interface MarketCodeRepository extends JpaRepository<User, Integer> {
	...
}
```

Spring Data JPA에서 제공하는 **JpaRepository 인터페이스를 상속하기만 해도 되며**,
인터페이스에 따로 `@Repository`등의 어노테이션을 추가할 필요가 없다.



JpaRepository를  단순하게 상속하는 것만으로 위의 인터페이스는 Entity 하나에 대해서 아래와 같은 기능을 제공하게 된다.

| **method** | **기능**                                                |
| ---------- | ------------------------------------------------------- |
| save()     | 레코드 저장 (insert, update)                            |
| findOne()  | primary key로 레코드 한건 찾기                          |
| findAll()  | 전체 레코드 불러오기. 정렬(sort), 페이징(pageable) 가능 |
| count()    | 레코드 갯수                                             |
| delete()   | 레코드 삭제                                             |

 

```java
public interface MarketCodeRepository extends JpaRepository<User, String> {
	List<User> findBykoreanName(String koreanName);
}
```

**※ 변수명을 korean_name 처럼 snake케이스로 사용할 시에 findBy가 적용이 안된다.
	변수명을 camel케이스로 맞추자.**



Query 메소드에 **포함할 수 있는 키워드**는 다음과 같다.

| **메서드 이름 키워드** | **샘플**                                           | **설명**                           |
| ---------------------- | -------------------------------------------------- | ---------------------------------- |
| And                    | findByEmailAndUserId(String email, String userId)  | 여러필드를 and 로 검색             |
| Or                     | findByEmailOrUserId(String email, String userId)   | 여러필드를 or 로 검색              |
| Between                | findByCreatedAtBetween(Date fromDate, Date toDate) | 필드의 두 값 사이에 있는 항목 검색 |
| LessThan               | findByAgeGraterThanEqual(int age)                  | 작은 항목 검색                     |
| GreaterThanEqual       | findByAgeGraterThanEqual(int age)                  | 크거나 같은 항목 검색              |
| Like                   | findByNameLike(String name)                        | like 검색                          |
| IsNull                 | findByJobIsNull()                                  | null 인 항목 검색                  |
| In                     | findByJob(String … jobs)                           | 여러 값중에 하나인 항목 검색       |
| OrderBy                | findByEmailOrderByNameAsc(String email)            | 검색 결과를 정렬하여 전달          |

