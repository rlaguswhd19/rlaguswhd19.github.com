---
title : "[JPA] SpringDataJPA"
tags : 
 - Jpa
---

# SpringDataJPA

JpaRepository를 사용하면 손쉽게 Query를 보낼 수 있다.

## JpaRepository<Entity, ID> 인터페이스

* 인터페이스
* @Repository가 없어도 빈으로 등록해준다
* 시작은 **@Import(JpaRepositoriesRegistrar.class)**
* 핵심은 **ImportBeanDefinitinalRegistrar 인터페이스**

<br/>



어떻게 JpaRepository가 Bean으로 등록되고 동작하는지 과정을 한번 살펴보자

## @EnableJpaRepositories

원래는 @Configuration에 붙여야 하는데 @SpringBootApplication에 있으므로 밑에 넣으면 된다. 하지만 Springboot의 경우는 넣지 않아도 된다.

### Why?

먼저 @EnableJpaRepositories에 들어가 보면 @Import(JpaRepositoriesRegistrar.class)가 있다. JpaRepository를 Bean으로 등록해주는것이다.

@Import(JpaRepositoriesRegistrar.class)를 보면 {@link ImportBeanDefinitionRegistrar}이 있는데 이것은 Bean을 정의할 수 있는 인터페이스로 Spring의 일부이다.

프로그래밍을 통해 Bean을 등록할 수 있게 해준다.

간단하게 객체 하나를 만들어서 구현해보겠다.

<br/>

#### Hyeonjong.class

```java
@Getter @Setter
public class Hyeonjong {

	private String name;

}
```

그다음 HyeonjongRegistrar을 만든다.

#### HyeonjongRegistrar.class

```java
public class HyeonjongRegistrar implements ImportBeanDefinitionRegistrar {

	@Override
	public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
		GenericBeanDefinition beanDefinition = new GenericBeanDefinition();
		beanDefinition.setBeanClass(Hyeonjong.class);
		beanDefinition.getPropertyValues().add("name", "hj");
		
		registry.registerBeanDefinition("Hyeonjong", beanDefinition);
	}
}
```

여기서 Bean을 등록하는 과정인데 Import를 해줘야한다.

#### SpringjpaApplication.class

```java
@SpringBootApplication
@Import(HyeonjongRegistrar.class)
public class SpringjpaApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringjpaApplication.class, args);
	}
}
```

그러면 Hyeonjong는 Bean으로 등록되어 @Autowired로 주입받아 사용할 수 있다.

<br/>

명시적으로 Bean으로 등록하지 않았음에도 이런 방식으로 JpaRepository를 받은 interface들을 찾아 Bean으로 등록해준다.