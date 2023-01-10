---
title : "[SPRINGBOOT] Spring Container"
tags :
 - Springboot
---



### 스프링 컨테이너

스프링 컨테이너는 스프링에서 자바 객체를 관리하는 공간, 스프링 컨테이너에서는 이 빈의 생성부터 소멸까지를 개발자 대신 관리해주는 곳

![ApplicationContext](https://user-images.githubusercontent.com/46040824/211563724-c8731095-e303-4ce1-afcc-db25213aee28.png)

##### ApplicationContext

애플리케이션 컨텍스트는 빈들의 생성과 의존성 주입 등의 역할을 하는 일종의 DI 컨테이너이다

- **AnnotationConfigApplicationContext** : 자바 어노테이션을 이용한 클래스로부터 객체 설정 정보를 가져온다.
- **GenericXmlApplicationContext** : XML로 부터 객체 설정 정보를 가져온다.
- **GenericGroovyApplicationContext** : 그루비 코드를 이용해 설정 정보를 가져온다.



### 스프링 빈

스프링 빈은 스프링 컨테이너에 의해 관리되는 객체를 의미한다.



#### @ComponentScan

`@SpringBootApplication`이 정의된 곳이 base package가 된다.

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
}
```



###### @Component

`ElementType.TYPE` 설정이 있으므로 Class 혹은 Interface에만 붙일 수 있다.

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Indexed
public @interface Component {
}
```

<br/>

---

<br/>

### DI와 IoC

`DI(Dependency Injection)`란 스프링이 다른 프레임워크와 차별화되어 제공하는 의존 관계 주입 기능으로, 객체를 직접 생성하는 게 아니라 외부에서 생성한 후 주입 시켜주는 방식이다.

![di2](https://user-images.githubusercontent.com/46040824/211570720-8229a534-4f35-4911-b156-bbe64d04c278.png)

첫번째 방법은 A객체가 B와 C객체를 New 생성자를 통해서 직접 생성하는 방법이고,

두번째 방법은 외부에서 생성 된 객체를 setter()를 통해 사용하는 방법이다.

이러한 두번째 방식이 의존성 주입의 예시인데, `A 객체`에서 `B, C객체`를 사용(의존)할 때 `A 객체`에서 직접 생성 하는 것이 아니라 `외부(IOC컨테이너)`에서 생성된 `B, C객체`를 조립(주입)시켜 `setter` 혹은 `생성자`를 통해 사용하는 방식이다.

![di](https://user-images.githubusercontent.com/46040824/211570408-44536ec1-48f3-4816-bac2-f248bea9eb42.png)



`IoC(Inversion of Control)`란 "제어의 역전" 이라는 의미로, 말 그대로 메소드나 객체의 호출작업을 개발자가 결정하는 것이 아니라, 외부에서 결정되는 것을 의미한다.

`IoC`는 제어의 역전이라고 말하며, 간단히 말해 "제어의 흐름을 바꾼다"라고 한다.

간단하게 설명하면 Ioc 컨테이너에 존재하는 Bean 객체에서 메소드나 객체 호출을 하는것이다.

출처

* *https://velog.io/@gillog/Spring-DIDependency-Injection*

<br/>

---

<br/>

### 스프링 싱글톤 컨테이너

싱글톤으로 빈을 관리하는 이유는 대규모 트래픽을 처리하기 위함이다.

매번 클라이언트에서 요청이 올 때마다 로직을 처리하는 빈을 새로 만들어서 사용하면 메모리가 감당하지 못한다. 그래서 빈을 싱글톤 스코프로 관리하여 여러 Thread가 빈을 공유해 처리한다.

Java로 싱글톤 패턴을 구현할때 단점

* private 생성자를 갖고 있어 상속이 불가능하다.
* 테스트하기 힘들다.
* 서버 환경에서는 싱글톤이 1개만 생성됨을 보장하지 못한다.
* 전역 상태를 만들 수 있기 때문에 객체지향적이지 못하다. (이해안됨)

### 싱글톤 레지스트리

* static 메소드나 private 생성자를 사용하지 않아 객체지향적 개발을 할 수 있다.

* 테스트 하기 편하다.

싱글톤이 멀티쓰레드 환경에서 서비스 형태의 객체로 사용되기 위해서는 내부에 상태정보를 갖지 않는 무상태(Stateless) 방식으로 만들어져야 한다. 여러 쓰레드들이 동시에 상태를 접근하여 수정하면 상당히 위험하기 때문이다.

```java
@Test
@DisplayName("Bean 주소 확인")
public void BeanTest() {
        applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);

        assertThat(appConfig.testService()).isNotSameAs(appConfig.testService());
        System.out.println(appConfig.testService());
        System.out.println(appConfig.testService());

        assertThat(applicationContext.getBean(TestService.class)).isSameAs(applicationContext.getBean(TestService.class));
        System.out.println(applicationContext.getBean(TestService.class));
        System.out.println(applicationContext.getBean(TestService.class));
    }
```

<br/>

```
com.afreecatv.study.test.TestService@28c06b84
com.afreecatv.study.test.TestService@3ba59a4f
com.afreecatv.study.test.TestService@6fc5a1c3
com.afreecatv.study.test.TestService@6fc5a1c3
```

<br/>

##### CGLIB

```java
@Test
@DisplayName("CGLIB 확인")
public void CGLIB() {
    applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
    System.out.println(applicationContext.getBean(AppConfig.class));
}
```



```
com.afreecatv.study.config.AppConfig$$EnhancerBySpringCGLIB$$25bb33df@69d2eefb
```

주의해야 할 점은 반드시 @Configuration 어노테이션이 존재해야 한다는 것이다. 이를 통해 빈 팩토리를 상속한 Application Context로써의 역할을 할 수 있다.(정확히는 Root Application Context)

스프링 프레임워크는 스프링 빈을 등록할 때 @Configuration 어노테이션이 붙은 빈들만 CGLIB를 적용하여 싱글톤을 보장해준다.

@Bean만 사용해도 스프링 빈으로 등록되지만, 의존관계 주입이 필요해서 메서드를 직접 호출하는 경우에는 싱글톤을 보장해주지 않는다.

*출처*

* *https://steady-coding.tistory.com/594*
* *https://mangkyu.tistory.com/151*
* *https://mangkyu.tistory.com/210*

