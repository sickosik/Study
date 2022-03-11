# Spring Boot 프로젝트에 롬복(lombok) 적용하기

## 1. 롬복(Lombok) 이란?

롬복(lombok)을 이용하면 getter, setter, constructor 를 매번 생성할 필요가 없다.

롬복은 @Getter, @Setter, @NoArgsConstructor 등등 어노테이션을 추가해주는 것으로 접근 제어자, 생성자 등을 자동으로 생성해준다.



## 2. pom.xml에 lombok 추가

`dependacy`를 추가

#### 2.1 maven

```xml
<dependencies>
    ...
    <dependency>
		<groupId>org.projectlombok</groupId>
		<artifactId>lombok</artifactId>
		<optional>true</optional>
	</dependency>
</dependencies>
```

#### 2.2 gradle

```java
repositories {
	mavenCentral()
}

dependencies {
	compileOnly 'org.projectlombok:lombok:1.18.22'
	annotationProcessor 'org.projectlombok:lombok:1.18.22'
	
	testCompileOnly 'org.projectlombok:lombok:1.18.22'
	testAnnotationProcessor 'org.projectlombok:lombok:1.18.22'
}
```



## 3. (IntelliJ인 경우) 플러그인 설치

Intellij에서는 롬복을 인식하지 못해서, 따로 플러그인을 설치해야 한다.

플러그인에서 lombok을 선택하여 설치한다.



## 4. 접근자/설정자 자동 생성

- `@Getter`

- `@Setter`

Lombok에서 가장 많이 사용되는 어노테이션으로, `aaa`라는 필드에 선언하면 자동으로 `getAaa()`(boolean 타입인 경우, `isAaa()`)와 `setAaa()` 메소드를 생성해준다.

- 필드 레벨에 선언

```java
@Getter
@Setter
private String name;
```

위와 같이 특정 필드에 어노테이션을 붙여주면, 아래와 같이 접근자/설정자 메소드가 자동으로 생성된다.

```java
user.setName("홍길동");
String userName = user.getName();
```

- 클래스 레벨에 선언해줄 경우, 모든 필드에 접근자와 설정자가 자동으로 생성된다.



## 5. 생성자 자동 생성

- `@NoArgsConstructor`  : 파라미터가 없는 기본 생성자를 생성해준다.

- `@AllArgsConstructor` : 모든 필드 값을 파라미터로 받는 생성자를 만들어준다.
- `@RequiredArgsConstructor`  : `final`이나 `@NonNull`인 필드 값만 파라미터로 받는 생성자를 만들어준다.

```java
@NoArgsConstructor
@RequiredArgsConstructor
@AllArgsConstructor
public class User {
  private Long id;
  @NonNull
  private String username;
  @NonNull
  private String password;
  private int[] scores;
}
```

```java
User user1 = new User();
User user2 = new User("홍길동", "123");
User user3 = new User(1, "홍길동", "123", null);
```



## 6. ToString 메소드 자동 생성

- `@ToString`

아래와 같이 `exclude` 속성을 사용하면, 특정 필드를 `toString()` 결과에서 제외시킬 수도 있다.

```java
@ToString(exclude = "password")
public class User {
  private Long id;
  private String username;
  private String password;
  private int[] scores;
}
```

위와 같이 클래스에 `@ToString` 어노테이션을 붙이고, 아래와 같이 필드를 세팅 후 출력을 하면,

```java
User user = new User();
user.setId(1);
user.setUsername("홍길동");
user.setUsername("123");
user.setScores(new int[]{80, 70, 100});
System.out.println(user);
```

다음과 같이, `클래스명(필드1명=필드1값,필드2명=필드2값,...)` 식으로 출력된다.

```bash
User(id=1, username=123, scores=[80, 70, 100])
```



## 7. equals, hashCode 자동 생성

자바 빈을 만들 때 `equals`와 `hashCode` 메소드를 자주 오버라이딩 하는데요. `@EqualsAndHashCode` 어노테이션을 사용하면 자동으로 이 메소드를 생성할 수 있습니다.

```java
@EqualsAndHashCode(callSuper = true)
public class User extends Domain {
  private String username;
  private String password;
}
```



`callSuper` 속성을 통해 `equals`와 `hashCode` 메소드 자동 생성 시 부모 클래스의 필드까지 감안할지 안 할지에 대해서 설정할 수 있습니다.

즉, `callSuper = true`로 설정하면 부모 클래스 필드 값들도 동일한지 체크하며, `callSuper = false`로 설정(기본값)하면 자신 클래스의 필드 값들만 고려합니다.

```java
User user1 = new User();
user1.setId(1L);
user1.setUsername("user");
user1.setPassword("pass");

User user2 = new User();
user1.setId(2L); // 부모 클래스의 필드가 다름
user2.setUsername("user");
user2.setPassword("pass");

user1.equals(user2);
// callSuper = true 이면 false, callSuper = false 이면 true
```



## 8. @Data

- `@Data` : `@Getter`, `@Setter`, `@RequiredArgsConstructor`, `@ToString`, `@EqualsAndHashCode`을 전부 설정해준다.

```java
@Data
public class User {
  // ...
}
```

- 모든 필드를 대상으로 접근자와 설정자가 자동으로 생성된다.

- `final` 또는 `@NonNull` 필드 값을 파라미터로 받는 생성자가 만들어진다
- `toStirng`, `equals`, `hashCode` 메소드가 자동으로 만들어진다.





