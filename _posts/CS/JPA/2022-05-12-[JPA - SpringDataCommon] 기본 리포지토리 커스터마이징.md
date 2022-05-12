---
title : "[JPA - SpringDataCommon] 기본 리포지토리 커스터마이징"
tags : 
 - Jpa
 - SpringDataCommon

---



# 기본 리포지토리 커스터마이징

모든 리포지토리에 공통적으로 추가하고 싶은 기능이 있거나 덮어쓰고 싶은 기본 기능이 있다.



JpaRepository를 상속 받는 인터페이스 정의

@NoRepositoryBean 추가하여 빈으로 생성하지 않는다고 표시한다.

##### BasicRepository

```java
@NoRepositoryBean
public interface BasicRepository<T, ID extends Serializable> extends JpaRepository<T, ID> {

    boolean contains(T entity);
}

```

EntityManager의 persits 상태에 있는지 확인하는 함수 contains을 만든다.

구현체 class를 하나 만들어 준다.

<br/>

##### SimpleMyRepository

```java
public class SimpleMyRepository<T, ID extends Serializable> extends SimpleJpaRepository<T, ID> implements BasicRepository<T, ID> {

    private EntityManager entityManager;

    public SimpleMyRepository(JpaEntityInformation<T, ?> entityInformation, EntityManager entityManager) {
        super(entityInformation, entityManager);
        this.entityManager = entityManager;
    }

    @Override
    public boolean contains(T entity) {
        System.out.println("SimpleMyRepository contains");
        return entityManager.contains(entity);
    }
}
```

bean으로 주입받지 않고 SimpleJpaRepository의 생성자에서 전달을 받는 EntityManager을 사용해서 내가 추가하는 함수에서 사용한다.

<br/>

```java
@EnableJpaRepositories(repositoryBaseClass = SimpleMyRepository.class)
public class SpringJpaApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringJpaApplication.class, args);
    }
}
```

repositoryBaseClass : 리포지토리 프록시를 만드는 데 사용할 리포지토리 기본 클래스를 구성합니다. 모든 JpaRepository에 BaseClass를 지정한다.
