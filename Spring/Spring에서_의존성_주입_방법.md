# Spring에서 의존성 주입 방법

Spring 프레임워크의 핵심 기술 중 하나가 바로 DI(Dependency Injection, 의존성 주입)이다. Spring 프레임워크와 같은 DI 프레임워크를 이용하면 다양한 의존성 주입을 이용하는 방법이 있는데, 각각의 방법에 대해 알아보도록 하자.

 

 

## 1. 다양한 의존성 주입 방법

#### *생성자 주입(Constructor Injection)*

생성자 주입(Constructor Injection)은 생성자를 통해 의존 관계를 주입하는 방법이다.

```java
@Service
public class UserServiceImpl implements UserService {

    private UserRepository userRepository;
    private MemberService memberService;

    @Autowired
    public UserServiceImpl(UserRepository userRepository, MemberService memberService) {
        this.userRepository = userRepository;
        this.memberService = memberService;
    }
    
}
```

 

생성자 주입은 생성자의 호출 시점에 1회 호출 되는 것이 보장된다. 그렇기 때문에 주입받은 객체가 변하지 않거나, 반드시 객체의 주입이 필요한 경우에 강제하기 위해 사용할 수 있다. 또한 Spring 프레임워크에서는 생성자 주입을 적극 지원하고 있기 때문에, 생성자가 1개만 있을 경우에 @Autowired를 생략해도 주입이 가능하도록 편의성을 제공하고 있다. 그렇기 때문에 위의 코드는 아래와 동일한 코드가 된다.

```java
@Service
public class UserServiceImpl implements UserService {

    private UserRepository userRepository;
    private MemberService memberService;

    public UserServiceImpl(UserRepository userRepository, MemberService memberService) {
        this.userRepository = userRepository;
        this.memberService = memberService;
    }

}
```

 

 

#### *수정자 주입(Setter 주입, Setter Injection)*

수정자 주입(Setter 주입, Setter Injection)은 필드 값을 변경하는 Setter를 통해서 의존 관계를 주입하는 방법이다. Setter 주입은 생성자 주입과 다르게 주입받는 객체가 변경될 가능성이 있는 경우에 사용한다. (실제로 변경이 필요한 경우는 극히 드물다.)

```
@Service
public class UserServiceImpl implements UserService {

    private UserRepository userRepository;
    private MemberService memberService;

    @Autowired
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Autowired
    public void setMemberService(MemberService memberService) {
        this.memberService = memberService;
    }
}
```

 

@Autowired로 주입할 대상이 없는 경우에는 오류가 발생한다. 위의 예제에서는 XXX 빈이 존재하지 않을 경우에 오류가 발생하는 것이다. 주입할 대상이 없어도 동작하도록 하려면 @Autowired(required = false)를 통해 설정할 수 있다.

 

 

#### *필드 주입(Field Injection)*

필드 주입(Field Injection)은 필드에 바로 의존 관계를 주입하는 방법이다. IntelliJ에서 필드 인젝션을 사용하면 Field injection is not recommended이라는 경고 문구가 발생한다.

```
@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserRepository userRepository;
    @Autowired
    private MemberService memberService;

}
```

필드 주입을 이용하면 코드가 간결해져서 과거에 상당히 많이 이용되었던 주입 방법이다. 하지만 필드 주입은 외부에서 변경이 불가능하다는 단점이 존재하는데, 점차 테스트 코드의 중요성이 부각됨에 따라 필드의 객체를 수정할 수 없는 필드 주입은 거의 사용되지 않게 되었다. 또한 필드 주입은 반드시 DI 프레임워크가 존재해야 하므로 반드시 사용을 지양해야 한다. 그렇기에 애플리케이션의 실제 코드와 무관한 테스트 코드나 설정을 위해 불가피한 경우에만 이용하도록 하자.

 

 

#### *일반 메소드 주입(Method Injection)*

일반 메소드를 통해 의존 관계를 주입하는 방법이다. 한번에 여러 필드를 주입 받을 수 있지만, 거의 사용할 필요가 없는 주입 방법이다.

 

 

## **2. 생성자 주입을 사용해야 하는 이유**

------

최근에는 Spring을 포함한 DI 프레임워크의 대부분이 생성자 주입을 권장하고 있는데, 자세한 이유를 살펴보도록 하자.

 

#### *생성자 주입을 사용해야 하는 이유*

 

**1. 객체의 불변성 확보**

실제로 개발을 하다 보면 느끼겠지만, 의존 관계 주입의 변경이 필요한 상황은 거의 없다. 하지만 수정자 주입이나 일반 메소드 주입을 이용하면 불필요하게 수정의 가능성을 열어두게 되며, 이는 OOP의 5가지 개발 원칙 중 OCP(Open-Closed Principal, 개방-폐쇄의 법칙)를 위반하게 된다. 그러므로 생성자 주입을 통해 변경의 가능성을 배제하고 불변성을 보장하는 것이 좋다.

 

 

**2. 테스트 코드의 작성**

실제 코드가 필드 주입으로 작성된 경우에는 순수한 자바 코드로 단위 테스트를 작성하는 것이 불가능하다. (물론 ReflectionTestUtilsf를 사용해 주입해줄 수 있기는 하다.) 예를 들어 아래와 같은 실제 코드가 존재한다고 하자.

```
@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserRepository userRepository;
    @Autowired
    private MemberService memberService;

    @Override
    public void register(String name) {
        userRepository.add(name);
    }

}
```

 

위의 코드에 대한 순수 자바 테스트 코드를 작성하면 다음과 같이 작성할 수 있다.

```
public class UserServiceTest {

    @Test
    public void addTest() {
        UserService userService = new UserServiceImpl();
        userService.register("MangKyu");
    }

}
```

 

위와 같이 작성한 테스트 코드는 어떻게 되겠는가? 테스트 코드가 Spring과 같은 DI 프레임워크 위에서 동작하지 않으므로 의존 관계 주입이 되지 않을 것이고, userRepository가 null이 되어 userRepository의 add 호출 시 NPE가 발생할 것이다. 이를 해결하기 위해 Setter를 사용하면 여러 곳에서 사용가능한 싱글톤 패턴 기반의 빈이 변경될 수 있는 치명적인 단점을 갖게 된다. 또한 반대로 테스트 코드에서도 @Autowired를 사용하기 위해 스프링 빈을 올리면 단위 테스트가 아니며 컴포넌트들을 등록하고 초기화하는 시간이 커져 테스트 비용이 증가하게 된다.

반면에 생성자 주입을 사용하면 컴파일 시점에 객체를 주입받아 테스트 코드를 작성할 수 있으며, 주입하는 객체가 누락된 경우 컴파일 시점에 오류를 발견할 수 있다. 심지어 우리가 테스트를 위해 만든 Test객체를 생성자로 넣어 편리함을 얻을 수도 있다.

 

 

**3. final 키워드 작성 및 Lombok과의 결합**

생성자 주입을 사용하면 필드 객체에 final 키워드를 사용할 수 있으며, 컴파일 시점에 누락된 의존성을 확인할 수 있다. 반면에 생성자 주입을 제외한 다른 주입 방법들은 객체의 생성(생성자 호출) 이후에 호출되므로 final 키워드를 사용할 수 없다.

또한 final 키워드를 붙임으로써 Lombok과 결합되어 코드를 간결하게 작성할 수 있다. Lombok에는 final 변수를 위한 생성자를 대신 생성해주는 @RequiredArgsConstructor를 [여기](https://mangkyu.tistory.com/78) 에서 살펴보았다.

Spring과 같은 DI 프레임워크는 Lombok과 환상적인 궁합을 보여주는데, 위에서 작성했던 생성자 주입 코드를 Lombok과 결합시키면 다음과 같이 간편하게 작성할 수 있다.

```
@Service
@RequiredArgsConstructor
public class UserServiceImpl implements UserService {

    private final UserRepository userRepository;
    private final MemberService memberService;

    @Override
    public void register(String name) {
        userRepository.add(name);
    }

}
```

 이러한 코드가 가능한 이유는 앞서 설명하였듯 Spring에서는 생성자가 1개인 경우 @Autowired를 생략할 수 있도록 도와주고 있으며, 해당 생성자를 Lombok으로 구현하였기 때문이다.

 

 

**4. 순환 참조 에러 방지**

애플리케이션 구동 시점(객체의 생성 시점)에 순환 참조 에러를 방지할 수 있다.

예를 들어 UserServiceImpl의 register 함수가 memberService의 add를 호출하고, memberServiceImpl의 add함수가 UserServiceImpl의 register 함수를 호출한다면 어떻게 되겠는가?

```
@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private MemberServiceImpl memberService;
    
    @Override
    public void register(String name) {
        memberService.add(name);
    }

}
```

 

```
@Service
public class MemberServiceImpl extends MemberService {

    @Autowired
    private UserServiceImpl userService;

    public void add(String name){
        userService.register(name);
    }

}
```

 

위의 두 메소드는 서로를 계속 호출할 것이고, 메모리에 함수의 CallStack이 계속 쌓여 StackOverflow 에러가 발생하게 된다.

```
Caused by: java.lang.StackOverflowError: null
	at com.mang.example.user.MemberServiceImpl.add(MemberServiceImpl.java:20) ~[main/:na]
	at com.mang.example.user.UserServiceImpl.register(UserServiceImpl.java:14) ~[main/:na]
	at com.mang.example.user.MemberServiceImpl.add(MemberServiceImpl.java:20) ~[main/:na]
	at com.mang.example.user.UserServiceImpl.register(UserServiceImpl.java:14) ~[main/:na]
```

만약 이러한 문제를 발견하지 못하고 서버가 운영된다면 어떻게 되겠는가? 해당 메소드의 호출 시에 StackOverflow 에러에 의해 서버가 죽게 될 것이다.

하지만 생성자 주입을 이용하면 이러한 순환 참조 문제를 방지할 수 있다.

```
Description:

The dependencies of some of the beans in the application context form a cycle:

┌─────┐
|  memberServiceImpl defined in file [C:\Users\Mang\IdeaProjects\build\classes\java\main\com\mang\example\user\MemberServiceImpl.class]
↑     ↓
|  userServiceImpl defined in file [C:\Users\Mang\IdeaProjects\build\classes\java\main\com\mang\example\user\UserServiceImpl.class]
└─────┘
```

애플리케이션 구동 시점(객체의 생성 시점)에 에러가 발생하기 때문이다. 그러한 이유는 Bean에 등록하기 위해 객체를 생성하는 과정에서 다음과 같이 순환 참조가 발생하기 때문이다.

```
new UserServiceImpl(new MemberServiceImpl(new UserServiceImpl(new MemberServiceImpl()...)))
```

 

 

### ***\*[ 요약 정리 ]\****

- OCP 원칙을 지키며 객체의 불변성을 확보할 수 있다.
- 테스트 코드의 작성이 용이해진다.
- final 키워드를 사용할 수 있고, Lombok과의 결합을 통해 코드를 간결하게 작성할 수 있다.
- 순환 참조 문제를 를 애플리케이션 구동(객체의 생성) 시점에 파악하여 방지할 수 있다.

이러한 이유들로 우리는 DI 프레임워크를 사용하는 경우, 생성자 주입을 사용하는 것이 좋다.



출처: https://mangkyu.tistory.com/125 [MangKyu's Diary]